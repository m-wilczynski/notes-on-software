[Distributed systems](/architecture/distributed-systems)
# Message broker per system

Message brokers are current go-to solution for communication between components, both inside system and as integration bus between systems.
In most common scenario you'd probably have one big instance of your broker (and that's fine!).
But sometimes you'd like to have seperate instance of your broker per system or - let's say - per business domain.

**Pros on why go for such model:**
1. **responsibility** both in dev and ops can be tied to particular business domain (example: finance, logistics, HR etc.)
2. **specialization** of particular node (and its mirrors) for ex. with better HW (higher IOPS etc) is easier
3. **maintenance** and configuration should be easier when managing only your area of business (relates to pt. 1)

**Here are some rules I've wrote down for working with such approach for event-driven architecture:**
- publisher should be dummy when it comes to routing - publishing only to broker inside its system boundry
- dev and ops in charge of particular publisher should be interested on how fast (and whether succesfully) events they are publishing are processed
- subsribers should generally have idea on which broker interesting message will appear; that's where they should subsribe


And some diagram for summarizing my approach (my broker of choice here is RabbitMQ):
![Diagram](https://raw.githubusercontent.com/m-wilczynski/notes-on-software/master/architecture/distributed-systems/message-broker-per-system.png)


