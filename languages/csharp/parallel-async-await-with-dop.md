[Csharp](/languages/csharp)
# Parallel async-await with degree of parallelism (throttling)

### Problem

Sometimes, when doing a lot of CPU-instensive data, we want to split the work in multiple threads. That's when `Parallel.ForEach` comes in handy, since it will execute given "job" with optimal number of threads.

Apparently, we cannot use such approach if we want to make a lot of HTTP calls or database calls with `async-await lambdas`.<br>
Consider we have external HTTP API, that accepts single entity as an incoming payload. To make things easier for our users, we've let them import many such entities to be processed at once, but we cannot change the external API.

### Naive, one-by-one solution

A naive implementation would be to with blocking, one-by-one processing:
```csharp
foreach (var entity in entities)
{
    var payload = await AnyJsonSerializer.Serialize(entity);
    await _httpClient.PostAsync(url, payload);
}
```
This "poor man's solution" is obviously very ineffective, since it waits for completion of one entity, to process another one (so it's actually blocking, besides the fact that it returns `Thread` to `ThreadPool`). Assuming we have 1000 entities and each takes 200ms to complete, we'll have to wait 3min 30sec.

### Parallel solution

We could improve naive solution to process all entities in parallel with `async-await` lambdas and `Task.WhenAll`:
```csharp
var entitiesTasks = entities.Select(async entity => 
{
    var payload = await AnyJsonSerializer.Serialize(entity);
    await _httpClient.PostAsync(url, payload);
});

await Task.WhenAll(entitiesTasks);
```

This however produces another problem: we now have 1000 `Task` that will be enqueued to `ThreadPool`. This might result in few scenarios:
- (more likely) **endpoint** we're trying to call **will either get overwhelmed** by all of those requests at once or will reject some of them due to some internal policies, anti-DDoS or any backpressure mechanics; for other I/O operations we could run into similiar issues, saturating connection pool to SQL database or disk I/O when calling filesystem
- (less likely) we might experience **thread pool starvation**, where it gets flooded by those 1000 (or more) `Task`s and features in application that are also async are **waiting for their time to resume** with continuation (ie. any job to do after Task completes)

A good article about thread pool starvation form Visual Studio team:
https://github.com/microsoft/vs-threading/blob/main/doc/threadpool_starvation.md

### Parallel solution with degree of parallelism

And that's how we end up with final solution with DOP/throttling.<br>
At first I decided to implement my own algorithm for this, but I've ended up finding simple, yet effective solution by Stephen Toub (https://devblogs.microsoft.com/pfxteam/implementing-a-simple-foreachasync-part-2/) using `Partitioner`, that will effectively partition input to N partitions and then process elements in each partition one-by-one.

Below example shows how to do it for mentioned above POST case, but can be further generalized with `Func` as a parameter, on what to do with each entity from `entities`:
```csharp
public IEnumerable<Task> PostInParallel(IEnumerable<Entity> entities, int degreeOfParallelism)
{
    return Paritioner.Create(entities).GetParitions(degreeOfParallelism)
        .Select(async partition => 
        {
            using (partition)
            {
                //while body can be generalized with Func<T,Task> passed as an argument
                while (partition.MoveNext())
                {
                    var payload = await AnyJsonSerializer.Serialize(entity);
                    await _httpClient.PostAsync(url, payload);
                }
            }
        });
}
```
