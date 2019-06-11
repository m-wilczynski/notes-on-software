[MassTransit](/frameworks-and-libraries/masstransit)
# Distributed transactions with Courier

MassTransit offers really elegant way on dealing with distributed transactions with **Courier**.

In a nutshell it's based on a `RoutingSlip` - a list of steps (commands) of your transactions. For each step there is defined **queue** address on which it is expected to be handler (called `Activity`) listening and waiting to process it.

Step will execute sequentionally and on step success propagate to next step's queue.  Messages are sent one by one, not all at once.

There are three main concepts in Courier:
- **activities** - `Activity` is a handler for a command which is a step in transaction
- **execution** (via `Execute` on activity) - actual execution of a step in transaction
- **compensation** (via `Compensate` on activity) - sort of rollback of a step, executed in an opposite order in transaction

#### Execution flow



#### Compensation flow


