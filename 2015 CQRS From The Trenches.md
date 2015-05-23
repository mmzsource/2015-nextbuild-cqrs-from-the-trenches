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

### The future of software engineering?

Probably not.

### Just an old pattern?

The CQRS pattern was first introduced by Greg Young and Udi Dahan. They took inspiration from a pattern called Command Query Separation which was defined by Bertrand Meyer in his book “Object Oriented Software Construction” (1988). The main idea behind Command Query Separation is: A method should either change the state of an object, or return a result, but not both. In other words, asking the question should not change the answer.

Basically CQRS is an architectural pattern that says that instead of having one monolithic API to an application, you split that into two: one for commands and one for queries - hence the name.

## Terms & Relations

If you follow the Command Query Separation pattern, there are 2 types of methods in your code: Commands and Queries.

A **Command** is something the system has to do. It's a combination of expressed intent (which describes what you want to do) as well as the information required to take action based on that intent. Examples: UserLogin or AddToCart or StartMachine.

A **Query** is a question or a request for information about something. Examples: getAllUsers, listAllItemsInCart, getCurrentMachineState.

Another important CQRS term is **Event**. An event describes something that has occurred in the application. Examples: UserLoggedIn, ItemAddedToCart, or MachineStarted. Events loosely couple all components in your application together.

Another term related to CQRS is **EventSourcing**. They are independent techniques, but they augment each other. So it's possible to build a CQRS system with and without eventsourcing. We've been using eventsouring in all of our CQRS projects up until now and we'll get into the pros and cons soon. In short, eventsourcing means storing current state as a series of events. It means letting your events be the origin of all changes. In other words, changes will occur as a result of an event. The event stream that results is the overview of all changes that ever occurred in the system. If a repository needs to rebuild a domain object from storage, it will replay all past events.

Now it's time to glue these concepts together: ![Architecture overview](http://www.axonframework.org/docs/2.3/images/detailed-architecture-overview.png)

1. The User (or actually any client) generates a **command** by requesting some form of change.
2. The CommandHandler is responsible for routing the command to the correct "Aggregate". An aggregate is an entity or group of entities that is always kept in a consistent state. It's a domain object responsible for guarding its own invariants. The state changes of aggregates result in the generation of Domain Events.
3. The generated Events are stored in an event store and are published on an event bus.
4. Event Handlers handle incoming events. They feed databases which can be sql, nosql, in memory, etc., all tuned to their specific clients.
5. Specific clients **query** specific data from storage and are served by highly tuned 'microservices' which are totally tailored for these specific queries.

I would like to underline some aspects of this picture. 

- First of all, it's very clear that commands and queries are very separated. The incoming commands and outgoing query results are only loosely coupled by events.
- Second: it's really important to understand the power of events; if I replay the events which are stored in the eventstore, I can build a totally new service based on all events which ever occurred in the system since the beginning and thereby add functionality without touching any existing code. 
- Third: Since it's so easy to tailor my read-model (I just make a new one), I can deliver very specific data in no time. For instance, I could build a view which does nothing else than build up complicated monthly or yearly reports. No need to bother a central database with that task. Or I could build a view which combines several domain objects into information completely tailored to my needs. Again: no need to do complex joins on a central database.

**TODO:** better architecture picture (mainly bigger. SVG?)

## Why should u use it (or not)

Now the big question is: why should you use this pattern, or shouldn't you? 

In the end, CQRS is just another tool in the toolbox, well suited for some tasks, but completely unusable for others. Let's discuss some general advantages and disadvantages before zooming into our experiences in practice.

### Pros

- Often reading data is much more frequent than writing. CQRS totally separates the two, making it a lot easier to tailor write - and read performance separately. Furthermore: reads from a user perspective has to be more performant than writes. User tends to find it easier to accept a slower response when data is changed. This also fits really nicely with the CQRS way of working.
- By splitting the application into two components, it allows you to build and think separately about two different models: commands and queries. This really helps making better API's which are easier to maintain. It's totally clear where changes are needed (command side or query side).
- If something goes wrong on the query side of your code: no problem. Just fix the bug, delete the data and replay the events with the bugfree component.
- Eventsourcing makes testing a breeze
- CQRS aggregates really put state in the forefront of your design, implementation and test efforts which is a really good thing.

### Cons

- Learning curve
- You cannot changed the event history. You can create compensating events and you can even upcast events, but this is often a lot more complex than fixing the bug and performing a database cleanup with a script. 
- Fairly new kid on the block and therefore sometimes hard to find help / information.

- Strong / Weak
- Tarpit criteria
- Framework selection criteria (James Shore)

**TODO:** Add pros and cons and sort them in order of priority

## WHEN should you use it?

From the axon documentation:

Will each application benefit from Axon? Unfortunately not. Simple CRUD (Create, Read, Update, Delete) applications which are not expected to scale will probably not benefit from CQRS or Axon. Fortunately, there is a wide variety of applications that does benefit from Axon.

Applications that will most likely benefit from CQRS and Axon are those that show one or more of the following characteristics:

The application is likely to be extended with new functionality during a long period of time. For example, an online store might start off with a system that tracks progress of Orders. At a later stage, this could be extended with Inventory information, to make sure stocks are updated when items are sold. Even later, accounting can require financial statistics of sales to be recorded, etc. Although it is hard to predict how software projects will evolve in the future, the majority of this type of application is clearly presented as such.

The application has a high read-to-write ratio. That means data is only written a few times, and read many times more. Since data sources for queries are different to those that are used for command validation, it is possible to optimize these data sources for fast querying. Duplicate data is no longer an issue, since events are published when data changes.

The application presents data in many different formats. Many applications nowadays don't stop when showing information on a web page. Some applications, for example, send monthly emails to notify users of changes that occurred that might be relevant to them. Search engines are another example. They use the same data your application does, but in a way that is optimized for quick searching. Reporting tools aggregate information into reports that show data evolution over time. This, again, is a different format of the same data. Using Axon, each data source can be updated independently of each other on a real-time or scheduled basis.

When an application has clearly separated components with different audiences, it can benefit from Axon, too. An example of such application is the online store. Employees will update product information and availability on the website, while customers place orders and query for their order status. With Axon, these components can be deployed on separate machines and scaled using different policies. They are kept up-to-date using the events, which Axon will dispatch to all subscribed components, regardless of the machine they are deployed on.

Integration with other applications can be cumbersome work. The strict definition of an application's API using commands and events makes it easier to integrate with external applications. Any application can send commands or listen to events generated by the application.

## Real world experience

- Interaction with CRUD systems
  - Person information
  - Data transfer

## Alternatives

- Datomic
- PGQ

## Brainfarts

Fire and forget: reply from the command handler generally does not contain everything needed to update in the client. An extra query is needed. But when? When will the event be propagated completely? A websockets API is needed in addition.

CQRS will never be applicable to a whole application; part of it may be for single user doing CRUD, with other parts more collaborative and suitable for CQRS. Fowler: In particular CQRS should only be used on specific portions of a system (a Bounded Context in DDD lingo) and not the the system as a whole. In this way of thinking, each Bounded Context needs its own decisions on how it should be modeled.

I did a littlebench mark on my laptop and I can store around 180,000 events per second on my spinning disc and on my laptop which is more than most applications will ever need anyway. [Alard on infoq on 2014-05-13](http://www.infoq.com/interviews/allard-buijze-cqrs-event-sourcing)