## Introduction

We've explained the basic concepts of CQRS and I hope you agree, CQRS sounds great! But of course there are downsides to each approach. During this section we'd like to share some experiences with you.

## FEEDBACK

One of the things I ran into when developing CQRS applications was the problem of feedback. 
Most of the applications I build are web applications. They use HTTP Requests and Responses as their main communication model.

But the CQRS model isn’t such a great match for the http request response model; sending commands is meant to be ‘fire and forget’. There is no feedback (well, maybe some feedback that the command was received). For instance: after sending a request for updating or creating some data, the client will only receive a response which essentially states ‘ok, I’ve heard you’. The options are:

- Sending out a new query to receive the updated content 
	- But you’re not sure if the model is already updated… how many times should you retry before you can conclude the command wasn’t executed
	- performance wise not a great option, especially with high latency networks
- Make a transaction span all the way from commands to events. [see axon group](https://groups.google.com/forum/m/#!topic/axonframework/9xxPin4_Kdc)
	- unfortunately, you’ll lose all possibilities of scaling when using this option (which is why you probably started with axon in the first place)
- Building a separate web socket API with some sort of data binding (e.g. meteors DDP protocol)
	- I feel we’re dealing with accidental complexity here… In case you’re having doubts, ask your users or project manager if this is essential

## CQRS UI Consequences

[This request response model is a nice match with user interaction requirements for user based systems. One thing I learned is that users come to an application with certain goals. They search the user interface for controls which best match their goals. When they use these user interface controls they want to know if they reached their goal, if this control triggered the system to actually meet their goal. In other words: they want feedback.]

You _must_ think differently about your UI when using CQRS. That's not to state that the above is not valid, it's just adding on top of it. You __can not__ (properly) build a new CQRS backend using your old UI. CQRS is kind of all-or-nothing in that regard.

## Match with REST

REST - in it's purest sense - is actually pretty CRUD based POST, GET, PUT, DELETE, while CQRS commands are intent based. There are pragmatic solutions of course, but we noticed this.

## SET VALIDATION

Now this is a nice discussion popping up again and again when talking about CQRS. The problem is essentially this: my aggregate doesn’t know enough to make the transition to another state. It must consult something outside the aggregate to make an informed decision. 

For instance: a user aggregate wants to know if it can transit to a ‘user created’ state and publish a user created event. But it can only do that if it’s totally sure there’s no other user with the same username and therefore it has to consult some sort of user repository. 

There are options to handle this, but they are not intuitive (from a CRUD point of view) and add complexity.

<!--

There are [a couple of basic ways to handle this:](https://groups.google.com/forum/#!msg/axonframework/RZ4D6kzbPjU/1azyCD0gcE0J
)

- Have the command handler execute a query to validate uniqueness. This means accepting the very small chance of a duplicate when two users register the same name at approximately the same time. 
	- this could break down easily with other use cases (high frequency of commands involving some sort of ordering for instance)
	- this kind of feels like you actually wanted one domain model (commands and queries through the same domain) which is a valid choice in lots of cases!
- Let the command handler keep a "used usernames" table, in which it updates and reads the usernames being used
	- This could easily break down when you’re having a *lot* of users, killing performance
- Detect the duplicates in the Event Handler that updates the query model. When a duplicate is detected, the second account should be blocked.
	- but this means sending commands from event handlers which isn’t recommended, because … TODO
- Use a 2-step process where a Saga confirms an account.
	- in my opinion this option feels best from a CQRS point of view (some sort of requested new user account state and after a check in the saga a account created state). But if you take a step back, you might wonder if this is really ‘better’ when compared to a normal CRUD application. It involves some extra complexity.
- Build an interceptor that checks the incoming create-commands against this query-model and rejects the command if it contains a duplicate username.
	- this solution makes it harder to scale out (TODO: I think...)

-->

If you’re particularly interested in this topic, do a search on set validation and cqrs. That’ll ensure a nice spending of your weekend.

## DUPLICATION

Another valid point popping up again and again: code duplication. Commands, Events, Entities and DTOs all sharing the same sort of properties. Just check out the cqrs and / or axon sample projects and you’ll see what I mean. I’ve noticed it in my own projects as well. Maybe it's because I use CQRS wrong, or applied it to the wrong project, but I'm not alone here, just check the axon mailing list for example.  

<!--

There are mainly 2 options here:

- Use value objects throughout the whole application, which carry around the shared properties. This is a nice option although it smells a little bit like you’re actually wanting one model. But it’s just a smell.
- Use a DSL to generate all the duplicate stuff. This kinda solves the initial problem, but doesn’t solve the maintenance problem.

TODO: what’s more to say? This just sucks and in my opinion and it’s worse than in most applications I’ve worked with so far. :(

{>>I think this section deserves a bit of nuance. I agree there's overhead here. Especially in smaller systems but the ripple effect you're describing is only valid if commands, events, persistence, and API change at the same time. I'm sure this happens a lot during the development phase and I'm sure it will happen once a system goes to production but I would be worried if this happens all the time (maybe CQRS wouldn't be a good choice in this case?). Furthermore, I'm always wary of code reuse just for the sake of code reuse. Just because your internal representations change doesn't mean your API should change.<<}

-->

## ITERATIVE DEVELOPMENT AND MAINTENANCE

On the axon mailing list, there is some good advice on designing CQRS based systems. It involves - i kid you not - thinking before doing.  List aggregates, commands, events, saga’s, and their relations, etc. Unfortunately this doesn’t map very well with most projects I’ve been working on which generally adopt some sort of twisted agile approach where the team is expected to immediately start building stuff and must show working features a.s.a.p. and evolve from there.

CQRS actually supports extension really well; you can add views without touching any other code. However, changing the basic structure is another story. Now I’m not going to pretend updating normal CRUD applications is a breeze, because it isn’t. BUT, there actually is a lot of knowledge on how to do it. Not to mention tooling (ecosystems). This means I don’t necessarily have to be a CQRS expert to change something in the structure of the application.

For instance: my event needs to change. [Now what](https://groups.google.com/forum/#!topic/axonframework/oszj_jAzoHg)?

- have code that can handle both versions of the events
- use event upcasting
- Make the new fields Optional with sane default values
- notice when an event has changed too much and should become a new type of event

You tell this to your maintenance department. Don’t be surprised to see confused faces. They are used to database upgrade scripts which run together with the deployment of the enhanced application. They have tools for that. Now this doesn’t mean it’s the only way or the best way, but it’s what you’ll face in practice.

I guess this is a general issue. As developers, maintenance is not always the first thing on our mind when developing applications. But the maintenance phase actually costs 40 - 80% of the total lifecycle cost of the application. Just let that sink in… Your teams development efforts are easily doubled or even tripled during maintenance...
Robert Glass came up with the 60/60 rule after a meta study of software costs. On average 60% of the software cost is spent during maintenance of which 60% is consumed by enhancements (error correction consumes ~ 20%).

Now, before you all get over enthusiastic about CQRS, please consult some senior maintenance guys in your organisation. Ask them what they think about your bright and shiny new tool. I’m not kidding. Involve them. Make sure you have a shared vision. Do not force your solution onto a team which doesn’t have the know-how and the tooling to maintain it during its lifetime. 

## YAGNI

In my opinion CQRS just doesn’t fit most of the applications most of the time. Yes, it scales out really well, but there’s a price to pay. A lot of systems actually just fail in the real world. A lot of startups would be helped more with a team which delivers functionality fast to test their business assumptions. You know, ‘minimum viable product’ to use another buzzword. Then, when the application really needs heavy scaling, there will be enough money and pressure to actually rewrite the whole thing. This money just isn’t there when you’re building the first version. There are no users yet. There probably isn’t enough money to deal with your learning curve and the extra price you pay to eventually, maybe, when it might be successful, scale out. In other words: you probably aint gonna need it. 

## EVENT SOURCING AND DELETION

In my mind, entities in the same application are usually related in one big logical data model. Yes, we need different views for different clients and yes, a business analyst has another view of the application than a user. In general, an information architect merges all these views in one big logical data model. 

Now, of course CQRS doesn’t tell you you shouldn’t do that. But most of the times, there are multiple systems involved in physically implementing this logical model. This means there are relations with other systems (which brings us to the set validation problem again). And I’ve experienced some trouble deleting stuff. 

For instance: in Holland you can request to have all your data removed from a system. Now how does that work in an event sourcing system? The past can’t be changed. You can mark data as ’removed’ or something like it, but I guess this doesn’t hold up in court. You should actually be able to DELETE stuff. 

This requirement made us separate user data and profiles from application data. The first being stored in CRUD systems and the second in event sourced systems. This enables me to delete a user from the CRUD database. Unfortunately there are no constraints (since it is not one physical database) which tell me I also need to delete related data. This means I can have events pointing to users which do not exist anymore. Null pointers for the win. 

Of course, this is a general concern for distributed systems, but it's more of a problem when dealing with a stack of immutable events.

## LEARNING CURVE

Just scanning the internet on questions about cqrs… you’ll notice there are a lot of conceptual questions. The same is true on the axon mailing list. And it’s not just newbies asking stuff. There is a big conceptual leap to be made which involves several levels of understanding. It sometimes feels like an onion; when you think you hit the core, there’s yet another layer of understanding. And at some point it’s just natural to start crying while peeling this onion to its core. ;-)

All these little issues, this not quite grasping how it should be designed, how it really should be implemented, how it should be maintained… This really costs a lot of time. Time you can’t bill your client for in my opinion, so be prepared to work and learn a lot in your own time before considering a CQRS based system. 

<!--
## TRANSACTIONS

Some 'already solved' problems pop up again when using cqrs. For instance: [transactions](https://groups.google.com/forum/m/#!topic/axonframework/ckHMUF-Th4g). Things you haven't been thinking about for years now rear up their ugly head again. Take for instance 2PC (Two Phase Commit) between eventstore and event bus... implemented yourself. Or... taking the conceptual leap from ACID to BASE. Really cool to learn, but is there enough time and business value to make the transition?

Alard Buijze [on infoq](http://www.infoq.com/articles/cqrs_with_axon_framework): 

> The traditional layered architecture is not one that supports scalability very well. Transaction management is a very heavy process in a scaled environment. XA transactions (... a transaction that affects more than one resource) ask for a big price to pay on each transaction, while only one-in-many transactions actually go wrong. CQRS uses asynchronous updates through events. **If any conflicting event is found (e.g. an item was bought, but the item is not in stock), we need to fix that in a compensating transaction.** It is effectively the ACID vs. BASE discussion. It really is something that we (as developers) have to get used to. We have to educate ourselves, and our customers that things just go wrong from time to time. Instead of presenting the error to the user (with the default "please try again later" screen), we try to solve it in the background. </br>
How information is queried, such as in the sharded databases you described, is really up to the developers to decide. **With CQRS, you can make a different choice for different parts of your application. All these different parts are updated by the same source: the events.** 

ACID: Atomicity, Consistency, Isolation, Durability - the set of properties that guarantee that database transactions are processed reliably.

BASE: Basically Available, Soft state, Eventual consistency (BASE), a consistency model.

Now, if an expert like Alard says something like that, I think you should really consider it. Can your team and your maintenance department handle the conceptual transition? Can you foresee the nasty problems that might pop up? Are you experienced enough on the subject? 

-->

## Axon

Great framework, Great documentation, Great service, constrains newbies in a good way.

