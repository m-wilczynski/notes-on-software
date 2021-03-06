[Distributed systems](/architecture/distributed-systems)
# Outbox pattern

#### Problem

When working with distributed systems async communication is a must and messages brokers are one of (if not the) most popular approach to achieve it in modern software.
As always, use of brokers (such as RabbitMQ) comes with a tradeoffs and there are lot of things we need to deal with.
One of them is ensuring atomicity of publishing a message to the broker and its consistency with business transaction commited to database. As messages brokers in general cannot participate in database transactions (with exceptions like NServiceBus' approach with MSDTC), we need to somehow ensure that:
- if business operation fails, we we'll neither commit it to database, nor will we publish any messages related to it
- if business operation succeeds and message publish succeeds but transaction commit fails, we'll discard everything
- if business operation succeeds but message publish fails, transaction will be rolled back

#### Example

Let's assume situation such as:
```csharp
public class MyOperationCommandHandler : ICommandHandler<MyOperationCommand>
{
    //Injected via ctor
    private readonly IPublishEndpoint _publishEndpoint;

    public async Task<Result> Handle(MyOperationCommand command)
    {
        using (var connection = new SqlConnection(connectionString))
        {
            connection.Open();
            using (var transaction = connect.BeginTransaction())
            {
                try 
                {
                    //Let's assume Dapper as data access framework

                    //1. Retrieve domain object from DB
                    var modelToOperateOn = await connection.QueryAsync<MyModel>(
                        sql: getSql, 
                        param: myParams, 
                        transaction: transaction);

                    //2. Perform requested command
                    modelToOperateOn.PerformMyOperation(command.MapForOperation());

                    //3. Save mutated domain object
                    await connection.ExecuteAsync(
                        sql: updateSql,
                        param: new { /* domain model mapped to UPDATE statement */ },
                        transaction: transaction);

                    //4. Publish event generated by domain object
                    //let's assume there could only be one for simplicity (with many it even gets trickier)
                    if (modelToOperateOn.DomainEvents.Any())
                    {
                        await _publishEndpoint.Publish(modelToOperateOn.DomainEvents.First());
                    }

                    //5. Everything went smoothly, commit!
                    transaction.Commit();

                    return Result.Ok();
                }
                //Something went wrong...
                catch (Exception ex)
                {
                    //6. Log what happened
                    Logger.Error(ex);

                    //7. Rollback transaction
                    transaction?.Rollback();

                    return Result.Failed();
                }
            }
        }
    }
}

```

Now, let's assume something went wrong (and in distributed systems you always should prepare for failure).

**Scenario #1**
- steps 1, 2 and 3 were succesfully executed and transaction has been commited
- step 4 failed (message broker offline, broker rejected message etc.)
- steps 7 gets executed, business transaction is not persisted to database - we're fine

**Scenario #2**
- steps 1, 2 and 3 were succesfully executed and transaction has been commited
- step 4 succeeds too
- step 5 fails (database went offline, connection problems, deffered constraint checks etc.)
- steps 7 gets executed if possible (probably not, so we just check if it's possible and move on)
- we got messages from step 4 published but no indication of them in our system!

**Scenario #3**
- we swap `Publish` and `Commit` in code
- steps 1, 2 and 3 were succesfully executed and transaction has been commited
- `Commit` succeeds
- `Publish` fails
- we're now in situation where business operation has been persisted but publish never occured; processes that should react to it will never react to it since they will never know it happened


#### Solution

To solve situation as above we might attempt to somehow compensate either `Commit` or `Publish` in catch clause (send CancelEvent or delete data from storage immediately after publish failed).
But, the more complicated business process, the harder it gets to compensate failures in such manner (and it's really cumbersome to maintain).
Another solution is to introduce **Outbox** pattern and:
- save generated event in same database as domain object we operated on - this enables us to achieve atomicity via database transaction
- use polling (via scheduler like Hangfire or Quartz.NET) to publish in small intervals every event we stored but have not yet sent

Such approach would look - from high level perspective - like that:


![Diagram](https://raw.githubusercontent.com/m-wilczynski/notes-on-software/master/architecture/distributed-systems/outbox-pattern.png)

And it's storage might be:

| Id | Type | Data | CorellationId |
| --- | --- | --- | --- |
| GUID | `My.App.Events.OperationPerformedEvent` | serialized event, ie. in JSON | GUID of domain object that initiated such event |

Finally, we might decide to remove event when it was published succesfully or keep dates of occurence and publish (as Kamil suggests - see link below).

In the end, outbox pattern comes with same guarantee as RabbitMQ itself - at least one delivery.
We must make sure that when messages get delivered twice, it's processing is idempotent.

#### Inspirations
* Chris Richardson, *https://microservices.io/patterns/data/transactional-outbox.html*
* Kamil Grzybek, *http://www.kamilgrzybek.com/design/the-outbox-pattern/*
