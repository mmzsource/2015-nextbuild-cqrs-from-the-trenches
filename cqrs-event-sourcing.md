# CQRS – Event Sourcing

- Maarten has just shown you how aggregates work in his talk about CQRS and state. What we haven't discussed yet however is how we are going to store these aggregates.
- [Show relational model of a shopping cart]
- We could just store the aggregate how we usually store our entities in a CRUD-like system. Just get the current state from the database, execute a command, and store the new state of the system.
- This is a perfectly viable approach, however... There are other possibilities which provide certain benefits, especially in a CQRS system.
- Maarten's demo already pointed us in the right direction, events!
- Instead of persisting the current state, we are going to persist all the events that _lead_ to the current state. (TODO: append only, no deletes!)
- [Show sequence of events which lead to the current state of the shopping cart] (TODO: of sequel pro)
- Now, why would we want to do that? What kind of system would even want to store a list of events instead of just the thing you want, the current state of the system?!?
- [Git/SVN logo? Git graph?]
- Well, we developers work with event sourced systems everyday! The most explicit example might just be your version control system. Git or Subversion is nothing more than an list of changes (events!).
- [RDBMS logo or visualization of translation log?]
- Another example is your favourite RDBMS. Although, as developers we always work with our RDBMS as a store for our current state, the RDBMS itself determines this current state based on an internal transaction log. Databases use this log to get back into a consistent state after crashes or other problems.
- So, now we know Event Sourced systems are all around us. But why would we use Event Sourcing in combination with CQRS? Well, CQRS is an awesome fit for Event Sourcing and Event Sourcing is an awesome fit for CQRS.
- Why is CQRS an awesome fit for Event Sourcing? Mutating state in CQRS works by sending _Commands_ to _Aggregates_. If we have the concept of _Commands_ (request for action) as the input for our state machines it's a natural step to add _Events_ (actions that have occurred) as the output for our state machines. This makes it relatively straight-forward to turn our _Command_ based system into an _Event Sourced_ system.
- [Show architecture overview with multiple Query models]
- Why is Event Sourcing an awesome fit for CQRS? One of the missing links in the pictures we've shown you before is the relationship between the Command and the Query model. We can build one of these Query models by receiving Events from an Aggregate. That's all fine, but what if we'd like to add a new Query model. (e.g. removed items from shopping cart?).
- Because our events are being stored, we can access them after the fact to build up our Query models. We call this _Replaying_ _Events_. Adding a new _Query_ model after the fact allows us to create new projections on existing data.
- What if we would've chosen a traditional _current state_ approach to our persistence and our users would've asked us for a simple new feature: a report which shows items which were removed from our Shopping Cart. In this situation we could've written some more code which tracks every occurrence of this action and stores this in a new database table. This already sounds like _Commands_ and _Events_ doesn't it? But let's keep it old school and continue on this track. This code change would've added this functionality, we would've deployed it to production and our users would log in to see their fancy new report with... no data. Because the data doesn't exist before this deployment.
- If we would've built this new report on top of an event sourced system the results would have been very different. We would've deployed the new feature, the new software would've built up its database from past events we would've stored and our users would log in to see their fancy new report with... all of the available historical data! TODO: add that we DON'T touch any existing code
- There is tremendous business value in storing the events instead of just the current state. It allows us to look from a different perspective at our data because we used a lossless persistence strategy.
- [demo with replaying? show event store first?]
- Event Sourcing offers more advantages. For example, what if a user sends us a bug report. We try to reproduce it and fail... I guess that never happened to anyone, right? In an Event Sourced system we ask our users for information regarding the bug and can check the database for the exact series of _Events_ that lead to the bug. We can export those events to our local development environment and use them for debugging, testing, and fixing the bug.
- Another advantage would be when deploying a new version to production. I think most of us write scripts which upgrade our database schemas to the new version our software needs. But how many people write a downgrade script for when the production deployment fails and we need to roll back? With Event Sourcing, you get all this for free as you can just install the old version of the software again, replay all the events from the past and bring the system back into a consistent state.
- Additionally, you could run the new version of your software next to the old version and switch the "active" version if the deployment succeeds.
- Now on to the demo! What do we want to show in a demo?
	- Show the different Query Models in the database (maybe that's already covered before?)
	- Show the event store and explain what is being stored. Explain that the event store is an audit log in itself and it can be proven it is correct (regulatory domains?).
	- Add new functionality by Replaying! (show database before and after to "prove" we're not cheating here)
- Complete architectuurplaat. uitleggen wat we nu weten. vragen of het geland is. CQRS betreft het scheiden van schrijf en leesmodellen. Dit levert ons o.a. het voordeel op van verschillende query modellen. Het command model . uitleggen plaatje.
- als dat duidelijk, gaan we nu naar onze praktijkervaringen.
----


- {>>We hebben hier een afsluiter nodig.<<} Concluding, Event Sourcing is a really powerful concept. CQRS 


- It's really the cherry on top of the CQRS pie as it truly _enables_ CQRS and adds real business value on top of that.



term: projections?
should we even mention snapshotting or ask for the questions?



CLARIFY this is an append only store. Events can't be changed (usually) and can't be deleted.