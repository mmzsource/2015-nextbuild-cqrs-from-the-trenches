# CQRS – Events

- Maarten has just shown you how aggregates work in his talk about CQRS and state. What we haven't discussed yet however is how we are going to store these aggregates.
- [Show relational model of a shopping cart]
- We could just store the aggregate how we usually store our entities in a CRUD-like system. Just get the current state from the database, execute a command, and store the new state of the system.
- This is a perfectly viable approach, however... There are other possibilities which provide certain benefits, especially in a CQRS system.
- [Show sequence of events which lead to the current state of the shopping cart]
- Maarten's demo already pointed us in the right direction, events!
- Instead of persisting the current state, we are going to persist all the events that _lead_ to the current state.
- Now, why would we want to do that? What kind of system would even want to store a list of events instead of just the thing you want, the current state of the system?!?
- [Git/SVN logo? Git graph?]
- Well, we developers work with event sourced systems everyday! The most explicit example might just be your version control system. Git or Subversion is nothing more than an list of changes (events!).
- [RDBMS logo or visualization of translation log?]
- Another example is your favourite RDBMS. Although, as developers we always work with our RDBMS as a store for our current state, the RDBMS itself determines this current state based on an internal transaction log. Databases use this log to get back into a consistent state after crashes or other problems.
- Luckily, CQRS is an awesome fit for Event Sourcing and Event Sourcing is an awesome fit for CQRS.
- Why is CQRS an awesome fit for Event Sourcing? Mutating state in CQRS works by sending _Commands_ to _Aggregates_. If we have the concept of _Commands_ (request for action) as the input for our state machines it's a natural step to add _Events_ (actions that have occurred) as the output for our state machines. This makes it relatively straight-forward to turn our _Command_ based system into an _Event Sourced_ system.
- [Show architecture overview with multiple Query models]
- Why is Event Sourcing an awesome fit for CQRS? One of the missing links in the pictures we've shown you before is the relationship between the Command and the Query model. We can build one of these Query models by receiving Events from an Aggregate. That's all fine, but what if we'd like to add a new Query model. (e.g. removed items from shopping cart?).
- Event Sourcing opens up the possibility of a new Query Model by...
- General advantages of Event Sourcing. replaying, bugs, deploying and rolling back.
- Optimization with snapshots?













- Comparison with Git, VCS.
- We use Event Sourced systems everyday

- aggregates generate events (check with overlap state chapter)
- events are published to listeners
- events are stored in event store

- events can be replayed to regenerate state
- reproduce bugs

- this is how the query models get built
- this allows you to add query models after the fact

- codewise, events are comparable to commands
- verleden tijd qua namen
- een event is iets dat plaats heeft gevonden
- show event store in slides
- bij commands noemen dat t opdrachten zijn qua grammar
- snapshots noemen?
- impedence mismatch noemen bij read wrote verhaal
- business value of event log

- there are no deletes
- a delete is just an event stating certain info is no longer valid
- personal information?
- 