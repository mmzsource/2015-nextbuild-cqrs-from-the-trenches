CQRS FROM THE TRENCHES
======================

Welcome to CQRS from the trenches. In this talk we'd like to give you a basic understanding of CQRS, and its place in the IT landscape. We'll focus on the main concepts, their relation and their behaviour. Together we've implemented and released 3 real world applications, running in production as we speak. 

Although we've been using Axon for our CQRS projects, this talk is not about Axon, it's about general CQRS concepts. CQRS critique in this presentation is in no way related to Axon. Axon is a well written framework, with good documentation and support. 

**TODO:** list main measurements from 3 projects. # lines of code, # concurrent users, # transactions per second (peak, average), etc.

### Outline

- What is CQRS and HOW does it work?
- Place in the IT landscape.
- Real world experience
  - Learning curve
  - Relation with CRUD databases (a.o. personal information, constraints)
  - Importing legacy data
  - Changing events
  - Changing aggregates
- Sample project?

## WHAT is CQRS?

### Hype? 

[Google Trend](http://www.google.com/trends/explore#q=cqrs)

(The numbers on the graph reflect how many searches have been done for a particular term, relative to the total number of searches done on Google over time. They don't represent absolute search volume numbers, because the data is normalized and presented on a scale from 0-100.)

### Just an old pattern?

The CQRS pattern was first introduced by Greg Young and Udi Dahan. They took inspiration from a pattern called Command Query Separation which was defined by Bertrand Meyer in his book “Object Oriented Software Construction” (1988). The main idea behind Command Query Separation is: A method should either change the state of an object, or return a result, but not both. In other words, asking the question should not change the answer.

CQRS goes some steps further than Meyer's idea.

### The future of software engineering?

Probably not.

## Terms & Relations

If you follow the Command Query Separation pattern, there are 2 types of methods in your code: Commands and Queries.

A **Command** is something the system has to do. It's a combination of expressed intent (which describes what you want to do) as well as the information required to take action based on that intent. Examples: UserLogin or AddToCart or StartMachine.

A **Query** is a question or a request for information about something. Examples: getAllUsers, listAllItemsInCart, getCurrentMachineState.

Another important CQRS term is **Event**. An event describes something that has occurred in the application. Examples: UserLoggedIn, ItemAddedToCart, or MachineStarted. Events loosely couple all components in your application together.

Another term related to CQRS is **EventSourcing** although it's possible to build a CQRS system with and without it. We've been using eventsouring in all of our CQRS projects up until now and we'll get into the pros and cons soon. In short, eventsourcing means storing current state as a series of events. It means letting your events be the origin of all changes. In other words, changes will occur as a result of an event. The event stream that results is the overview of all changes that ever occurred in the system. If a repository needs to rebuild a domain object from storage, it will replay all past events.

Now it's time to glue these concepts together: ![Architecture overview](http://www.axonframework.org/docs/2.3/images/detailed-architecture-overview.png)

1. The User (or actually any client) generates a **command** by requesting some form of change.
2. The CommandHandler is responsible for routing the command to the correct "Aggregate". An aggregate is an entity or group of entities that is always kept in a consistent state. It's a domain object responsible for guarding its own invariants. The state changes of aggregates result in the generation of Domain Events.
3. The generated Events are stored in an event store and are published on an event bus.
4. Event Handlers handle incoming events. They feed databases which can be sql, nosql, in memory, etc., all tuned to their specific clients.
5. Specific clients **query** specific data from storage and are served by highly tuned 'microservices' which are totally tailored for these specific requests.

I would like to underline some aspects of this picture. 

- First of all, it's very clear that commands and queries are totally separated. The incoming commands and outgoing query results are loosely coupled by events.
- Second: it's really important to understand the power of events; if I replay the events which are stored in the eventstore, I can build a totally new service based on all events which ever occurred in the system since the beginning and thereby add functionality without touching any existing code. 
- Third: Since it's so easy to tailor my read-model (I just make a new one), I can deliver very specific data in no time. For instance, I could build a view which does nothing else than build up complicated monthly or yearly reports. No need to bother a central database with that task. Or I could build a view which combines several domain objects into information completely tailored to my needs. Again: no need to do complex joins on a central database.


**TODO:** better architecture picture (mainly bigger. SVG?)

## Why should u use it (or not)

Now the big question is: why should you use this pattern, or shouldn't you? 

In the end, CQRS is just another tool in the toolbox, well suited for some tasks, but completely unusable for others. Let's discuss some general advantages and disadvantages before zooming into our experiences in practice.

- Pro / cons
- Strong / Weak
- Tarpit criteria
- Framework selection criteria (James Shore)

## Real world experience

- Interaction with CRUD systems
  - Person information
  - Data transfer

## Alternatives

- Datomic
- PGQ

Problems:

Fire and forget: reply from the command handler generally does not contain everything needed to update in the client. An extra query is needed. But when? When will the event be propagated completely? A websockets API is needed in addition.