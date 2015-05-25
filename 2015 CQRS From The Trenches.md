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


- [MM] Welcome
- [MM] WHAT is CQRS
- [MM] T&R
- [??] 'Preview' of demo to illustrate terms & behavior
- [MM] State verhaal 
- [MM] Demo: show state changes based on commands 
- [] Write model volledig los van read model
- [] Show how easy it is to add a read model
- [] Eventsourcing: yes or no?
- [BG] Replay toelichting
- [BG] Demo: show power of replay
- [DL] Pros en Cons
- [] Toelichten 'trenches'
  - [] Learning curve
  - [BG] koppeling met CRUD
  - [] Changing events
  - [] Changing aggregates


## WHAT is CQRS?

### Hype? 

[Google Trend](http://www.google.com/trends/explore#q=cqrs)

(The numbers on the graph reflect how many searches have been done for a particular term, relative to the total number of searches done on Google over time. They don't represent absolute search volume numbers, because the data is normalized and presented on a scale from 0-100.)

### The future of software engineering?

Probably not.

### Just an old pattern?

The CQRS pattern was first introduced by Greg Young and Udi Dahan. They took inspiration from a pattern called Command Query Separation which was defined by Bertrand Meyer in his book “Object Oriented Software Construction” (1988). The main idea behind Command Query Separation is: A method should either change the state of an object, or return a result, but not both. In other words, asking the question should not change the answer.

Basically CQRS is an architectural pattern that says that instead of having one monolithic API to an application, you split that into two: one for commands and one for queries - hence the name.

It aims to help developers build scalable and extensible applications.

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

- First of all, it's very clear that commands and queries are separated. They have different API's. The incoming commands and outgoing query results are loosely coupled by events.
- Second: it's really important to understand the power of events; if I replay the events which are stored in the eventstore, I can build a totally new service based on all events which ever occurred in the system since the beginning and thereby add functionality without touching any existing code. 
- Third: Since it's so easy to tailor my read-model (I just make a new one), I can deliver very specific data in no time. For instance, I could build a view which does nothing else than build up complicated monthly or yearly reports. No need to bother a central database with that task. Or I could build a view which combines several domain objects into information completely tailored to my needs. Again: no need to do complex joins on a central database.

**TODO:** better architecture picture (mainly bigger. SVG?)

{>>Should we mention eventual consistency here? In my experience a lot of people have trouble grasping this concept as it appears to them as a loss of control.<<}
  {>> MMz - Yes please! <<}

## Why should u use it (or not)

Now the big question is: why should you use this pattern, or shouldn't you? 

In the end, CQRS is just another tool in the toolbox, well suited for some tasks, but completely unusable for others. Let's discuss some general advantages and disadvantages before talking about its sweet spots and zooming into our experiences in practice.

### Pros

- Separation of reads and writes. 
  - Often reading data is much more frequent than writing. CQRS totally separates the two, making it a lot easier to tailor write - and read performance separately. Furthermore: reads from a user perspective has to be more performant than writes. User tends to find it easier to accept a slower response when data is changed. This also fits really nicely with the CQRS way of working.
  - By splitting the application into two components, it allows you to build and think separately about two different models: commands and queries. This really helps making better API's which are easier to maintain. It's totally clear where changes are needed (command side or query side).
- CQRS really puts state in the forefront of your design, implementation and test efforts which is a really good thing.
- Eventsourcing makes testing a breeze
- If something goes wrong on the query side of your code it is pretty easy to put things straight again: Just fix the bug, delete the data and replay the events with the bugfree component. Besides the possibility of replaying events, having an event log available significantly simplifies reproduction of bugs (just replay the events which caused the issue).

### Cons

- Learning curve
- Immutable event history. You cannot change the event history. You can create compensating events and you can even upcast events, but this is often a lot more complex than fixing the bug and performing a database cleanup with a script in a 'normal' CRUD application.
- Fairly new kid on the block and therefore sometimes hard to find help / information.

Previous pros and cons are somewhat arbitrary chosen. Let's try to view it from a more 'inter-subjective' point of view. For this, I'd like to use the very often cited "out of the tar pit" paper (2006) by Moseley and Marks.

### Tarpit criteria

In "out of the tarpit" Moseley and Marks claim that complexity is the single major difficulty in the successful development of large-scale software systems. Since *understanding a system* is a prerequisite for avoiding all sorts of problems when working with it, and complexity destroys the ability to understand a system, they state that complexity is the root cause of the vast majority of problems with software.

From "Tarpit":
- “...we have to keep it crisp, disentangled, and simple if we refuse to be crushed by the complexities of our own making...” – Dijkstra
- Approaches to understanding:
	- Testing
		- Commands, events, and aggregates allow for awesome testing (state verification, yay!). Axon even allows this without direct interaction with the test subject.
	- Informal Reasoning
		- See Complexity caused by Control. If the system can be split into clean and simple components this allows for great informal reasoning. If that's harder to do, I think CQRS makes it difficult to understand a system.
- Causes of complexity:
	- Complexity caused by State
		- CQRS imho scores great on this front as it forces you to think about state on a lot of fronts (what to "store" in aggregates, what to store in read models) and all mutations happen using immutable (pure?) commands.
	- Complexity caused by Control
		- If your application can be split up into cleanly separated components CQRS *can* score great on this front (e.g. Motown add-ons are really easy to grasp as they deliver a specific piece of functionality and interact with the domain using commands and events). However, when defining more straight-forward functionality (e.g. CRUD with a bit of business logic, which is a **major** part of **a lot** of applications) it is often a cause for complexity as you have to follow the flow of commands and events over multiple components.
		- "In cases (such as existing large systems) where this separation cannot be directly applied we believe the focus should be on avoiding state, avoiding **explicit** control where possible, and striving at all costs to get rid of code."
	- Complexity caused by Code Volume
		- Not sure what the impact of CQRS is here. In my experience it's pretty verbose (i.e. commands, events) but it does allow for cleanly separated components (which tend to be easier to grasp by themselves). What are your experiences here? {>> MMz - In my opinion it's verbose and there's a lot of duplication in commands and events. I've seen it @ lendex, seen it @ motown and heard the same problem arise @ windcentrale. Only 'solution' I can think of: climb up the ladder of abstraction and *generate* events and commands from one source, for instance some sort of specification. This, however, raises a lot of other issues (for instance will changes in the code be reflected in the spec, etc) <<} 
- Accidents and Essence:
	- Essential Complexity: is inherent in, and the essence of, the problem (as seen by the users).
		- We hence see essential complexity as “the complexity with which the team will have to be concerned, even in the ideal world”.
	- Accidental Complexity: is all the rest — complexity with which the development team would not have to deal in the ideal world (e.g. complexity arising from performance issues and from suboptimal language and infrastructure). https://www.dropbox.com/s/42xfqu0ivts3jp4/Screenshot%202015-05-16%2016.24.59.png?dl=0
	I think CQRS is in the 'accidental complexity' corner; the user probably couldn't care less. 

### Framework selection criteria (James Shore)

- Lock in: Boo! Because there is no generally accepted API / standard for CQRS frameworks, it's almost impossible to switch between different frameworks at the moment. 
- Opinionated Architecture. Boo! From the axon documentation: "Axon does not, in any way, try to hide the CQRS architecture or any of its components from developers. Therefore, depending on team size, it is still advisable to have one or more developers with a thorough understanding of CQRS on each team." It's easy to see this is not Axon specific, but a general CQRS property since CQRS is so different from 'normal' CRUD applications.
- Accidental Complexity. It's a toss up. There's a lot of learning involved in switching to CQRS. This takes time; time a customer probably isn't prepared to invest. Then again: extending and scaling the application is really easy. {>>None of the complexity is essential unless the customer has some really specific remarks. That doesn't make it the wrong choice though, I just think it isn't the *right* choice in a lot of circumstances. I would *always* consider bits and pieces of CQRS when making architectural choices.<<}
- Testability. Yay! Especially when using eventsourcing, testing is fantastic. Also: repeating bugs in production is really easy.
- Performance. {>> **TODO:** I don't know! Do we have some real world measurements on this one? <<} {>>I personally don't but I think the performance advantages are evident on a theoretical level. I just don't think most *regular* systems ever need this kind of performance :-). – Dennis <<}
- Monitoring. Yay! Especially when eventsourcing is used. Automatic audit trail is really helpfull here. {>>Yay indeed!<<}

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

"So, when should you avoid CQRS?

The answer is most of the time." – [Udi Dahan :-D](http://www.udidahan.com/2011/04/22/when-to-avoid-cqrs/)

{>>I think we should expand on this because I think this is the crux of the talk. A "from the trenches" talk should give the listener *some* idea where this tool fits in his toolbox.<<}

## Real world experience

- Interaction with CRUD systems
  - Person information
  - Data transfer

## Alternatives

- Datomic {>>@MMz_, I think you know the most about Datomic. I think I know why it's an alternative to event sourcing with regards to the acknowledgement of time as a concept but maybe you can elaborate on this.<<}

Event Sourcing:
- PGQ -> alternative to event sourcing? -> yep, I think it is! in terms of read scalability 
- Reporting database

Domain modeling:
- Architectural concepts within DDD, CQRS seems like an event-driven evolution of these. If going all-in CQRS doesn't seem like a fit, DDD could be the sweet spot. (plain ol' CRUD > DDD > CQRS)

Complexity/Scalability:
- Message-driven architecture/Fine-grained SOA/Microservices. It seems the current trend is mitigating complexity by splitting up monoliths. These microservices by themselves are usually to small and simple to do full-blown CQRS but usually do have some aspects of ES. If a microservice is responsible for it's own bounded context, how does one add a new microservice dependent on the data from this context. Well, event streams of course. :-)

## Brainfarts

Fire and forget: reply from the command handler generally does not contain everything needed to update in the client. An extra query is needed. But when? When will the event be propagated completely? A websockets API is needed in addition.

CQRS will never be applicable to a whole application; part of it may be for single user doing CRUD, with other parts more collaborative and suitable for CQRS. Fowler: In particular CQRS should only be used on specific portions of a system (a Bounded Context in DDD lingo) and not the the system as a whole. In this way of thinking, each Bounded Context needs its own decisions on how it should be modeled.

I did a littlebench mark on my laptop and I can store around 180,000 events per second on my spinning disc and on my laptop which is more than most applications will ever need anyway. [Alard on infoq on 2014-05-13](http://www.infoq.com/interviews/allard-buijze-cqrs-event-sourcing)

## Personal takeaways

Room for some personal takeaways not yet represented in the texts above, these 

### Dennis

- Even if I never use CQRS again it was a valuable learning experience as it opens your eyes to alternative architectures. Where before I wouldn't have thought twice about "just" using CRUD nowadays I'm way more creative in the architecture phase of a project. In this way, I compare it too the lessons I took from dabbling in FP, yes there is another way, yes there is a potentially better way, and yes state is evil. "No more one-size-fits-all architecture"
- It learns you a lot about your code's relationship to data. Not everything has to be stored in a database, sometimes volatile data is fine, and there's no such thing as real consistency. Embrace uncertainty.
- Besides the above, finally NoSQL (or polyglot persistence) makes sense to me. The flexibility that an event-driven architecture gives you in relation to data makes it possible to pick and choose alternative storage mechanisms without going all-in. This is way harder in traditional CRUD systems.
- Storing intent is powerful. The fact that data can be modeled after the fact is insanely flexible.
- In most CRUD applications separating domain models from data models is considered "too much work" or "overhead" in the early phases of a project. In the later phases of a project, this often comes back to bite us in the ass. In CQRS with ES, there is no choice (but is this a good thing? :-)).
- ES is the real MVP! All the rest is just fluff, conventions, and other stuff to make some related things easier. All of the actual power comes from ES.
- On that note, it's no surprise Greg Young's "CQRS" book is called ["Event Centric"](http://www.amazon.com/Event-Centric-Simplicity-Addison-Wesley-Signature/dp/0321768221) :-)