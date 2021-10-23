[Csharp](/languages/csharp)
# Enum breaking changes

### Problem

When deciding to go for strongly typed (either for sync communication via typed clients or async communication with tools like MassTransit) you may run into compatibility issues when breaking changes are introduced. In general new fields or new, optional values of current fields are non-breaking, but that's not the case for new, possible `enum` values and it's due to how they're serialized.

There are two common approaches to enum serialization:
1. **serialize using enum member name** - for `DateTimeKind.Utc` it would be `"Utc"`
2. **serialize using enum member integer value** - for `DateTimeKind.Utc` it would be `1`

And now - the example!<br>
If we assume test enum like this...
```csharp
enum TestEnum
{
    One = 1,
    Two = 2
}
```
...results would be rather unsatisfactory:
```csharp
//1. This will throw
TestEnum parsedByName = (TestEnum)Enum.Parse(typeof(TestEnum), "Three");
//2.1 This will allow unknown value
TestEnum parsedByIntValue = (TestEnum)Enum.Parse(typeof(TestEnum), "3");
//2.2 This (and approach above too) will also allow garbage
TestEnum parsedByIntValueByImplicitCast = (TestEnum)1234;
```

### My approach

I've had this issue with various scenarios (sync response contracts, async event contracts) and with various serializers (JSON with various serializers, msgpack via MessagePack.CSharp).

I've decided to split transport-related data with how it's exposed to contract consumers.<br>
Let's take `TestEnum` mentioned above and use it as part of response or event contract:
```csharp
public class GiveMeDataResponse
{
    public TestEnum Payload { get; set; }
}
```
With no changes it will suffer from same issues that were mentioned above.

Let's add another value to our `TestEnum` to indicate undefined/unspecified value:
```csharp
enum TestEnum
{
    Undefined = 0, //if zero is a valid value, we could go for -100 etc.
    One = 1,
    Two = 2
}
```
and modify our contract:
```csharp
public class GiveMeDataResponse
{
    public int PayloadAsInt { get; set; }

    /*
        This should be marked as ignored for serializers;
        for ex. MessagePack.CSharp allows [IgnoreDataMember] 
        from System.Runtime.Serialization
    */
    public TestEnum Payload
    {
        get
        {
            //You can go with Enum.TryParse approach as an alternative
            if (Enum.IsDefined(typeof(TestEnum), PayloadAsInt))
            {
                return (TestEnum)PayloadAsInt;
            }

            return TestEnum.Undefined;
        }
    }
}
```

Now whenever we receive unknown enum value, we would simply fallback to always known, `TestEnum.Undefined` value.<br>
This in fact should actually be handled on business level with some sort of fallback or ignoring (question to ask: *"How do we communicate users, that we don't know what to do with response from other system?"*).

What's most important is that we would not suffer from neither serialization exceptions nor garbage or unknown integers flying around anymore.

### Bonus - publisher-driven contracts

As a bonus, if you're using publisher-driven contracts (which is most of the cases for services having more than one consumer), you can actually add approach mentioned above to your response contracts.

This way your consumers can simply depend on `GiveMeDataResponse.Payload` property and have their end covered even when your service produces enum values yet unknown to them (when they did not bump their our contracts with Nuget or you forgot to communicate changes etc.).

Of course it would not work for polyglot tech stack companies, but if you work for one, you're not publishing your contracts via Nuget anyway.