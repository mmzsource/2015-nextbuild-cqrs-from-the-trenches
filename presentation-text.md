CQRS from the trenches
======================

## Sheet 1 - CQRS from the trenches

<!-- 
  Speaker: BG
  Goal: Make sure everyone is in the correct room
--> 

Everybody in here expecting to be in the CQRS presentation? 

- Room 1: Eventsourcing angular
- Room 3: Continuous delivery
- Room 4 & 5 already running workshops (invariants & scala)

## Sheet 2 - Welcome

<!-- 
  Speaker: BG
  Goals: 
    - Introduce the speakers
    - State our main goal
    - Axon disclaimer
-->

Welcome to our CQRS presentation. We are Dennis Laumen, Ben van Gameren and Maarten Metz and we're colleagues working at IHomer. Our bio's and link to our company are on the nextbuild website in case you're interested.

But we're not here for a marketing talk, we're here to talk about CQRS. From the trenches that is, since we've already got our hands dirty working with CQRS. Together we've implemented and released 3 real world applications, running in production as we speak. 

In this talk we'd like to give you a basic understanding of CQRS, and its place in the IT landscape. We'll focus on the main concepts, their relation and their behaviour.  

Although we've been using the Axon framework for our CQRS projects, this talk is not about Axon, it's about general CQRS concepts. CQRS critique in this presentation is in no way related to Axon. Axon is a well written framework, with good documentation and support.

## Sheet 3 - Outline

<!--
  Speaker: BG
  Goals: Make sure people understand the structure of our talk
-->

Now it's time to move on to the structure of this presentation. 

First of all we'd like to introduce the main terms and architecture and zoom in from there. You'll notice quickly how central the concept of reads and writes is in CQRS. The concept of writes is heavily related to state which is the next subject. After that, we'll introduce another important concept which is eventsourcing. At that point, you'll know enough about CQRS to be able to understand our general evaluation of it. Finally, we'll zoom into our general evaluation and zoom in to some subjects we like and dislike about CQRS.

Since we'll cover a lot of ground, you're invited at any time to ask questions.

Clear? Let's go!

## Sheet 4 - Terms (CQRS & CQS)

<!--
  Speaker: MM
  Goals: Define 'CQRS' and explain its roots
-->

The CQRS pattern was first introduced by Greg Young and Udi Dahan. They took inspiration from a pattern called Command Query Separation which was defined by Bertrand Meyer in his book “Object Oriented Software Construction” (1988). The main idea behind Command Query Separation is: A method should either change the state of an object, or return a result, but not both. In other words, asking the question should not change the answer.

Basically CQRS is an architectural pattern that says that instead of having one monolithic API to an application, you split that into two: one for commands and one for queries - hence the name.

## Sheet 5 - WHY?!?!

<!--
  Speaker: MM
  Goals: Give the audience a first glimps of WHY you would want CQRS
-->

We'll dive into the WHY and HOW a lot deeper, but let me give you a glimps of one of the main goals of CQRS: CQRS aims to help developers build scalable and extensible applications. Bear with me, we'll explain in detail what separation of writes and reads has to do with that. But let's first discuss two other important terms: commands (or writes) and queries (or reads)

## Sheet 6 - Command

<!--
  Speaker: MM
  Goals: Define 'Command'
-->

(It's all on the sheet)

## Sheet 7 - Query

<!--
  Speaker: MM
  Goals: Define 'Query'
-->

(It's all on the sheet)

Now that we're familiar with the most important terms, let's move on to a very basic architecture.

## Sheet 8 - CRUD architecture

<!--
  Speaker: MM
  Goal: Start from the reference point of the audience. A standard CRUD architecture.
-->

Here we have a well known layered architecture. Some client talks with some application. One domain Model lives inside the application. It's state is written to a database. Reads and Writes are served by the same domain model.

## Sheet 9 - CQRS Architecture - 1

<!--
  Speaker: MM
  Goal: Introduce the simplest CQRS architecture possible
-->

Now let's make one small change: separate the commands and the queries, as suggested by the CQRS way of designing. So everything's the same, only the domain model is split up into two models: the Query Model and the Command Model. Clear?

But wait a minute... If we separate the commands from the queries, we're no longer constrained to just a single database, right? Right! 

## Sheet 10 - CQRS Architecture - 2

<!--
  Speaker: MM
  Goal: Explain the positive consequences of separating reads and writes
-->

Since we're having completely separated writes and reads, we're able to use different databases for both. Again: bear with me, we're only touching the surface here, we'll be explaining a lot more in this talk.

For instance, we'll soon explain why some arrows have two heads while others only have one. First let's take one step more:

## Sheet 11 - CQRS Architecture - 3

<!--
  Speaker: MM
  Goal: Take the positive consequences one step further.
-->

We're also not constrained to only one query model. We can have multiple query models if needed. They can all serve their specific client request. Completely tailor made. So now it's beginning to get clear why CQRS aims at helping developers build scalable and extendable applications, right? 

CQRS applications are easily extendable because I can add Query models when needed, without touching any existing code. Think of the domain model a couple of sheets back; "one model to rule them all" turns out to be a pain in the ass when requirements change and things get really complicated. Also think of the runtime behaviour; writes wanting to lock the database and 'stop the world' for a second while reads are stacking up. All because reads and writes are served by the same model.

CQRS applications are scalable because I can tailor my infrastructure separately to the actual usage of my query and my command models. 

## Sheet 12 - Architecture and Terms

<!--
  Speaker: MM
  Goal: Couple terms and architecture in one picture and make sure everyone understands.
-->

So, where does that bring us? We have clients, firing commands to our application. And we have clients firing queries to our application. And both are handled by completely separated models.

Is that clear so far? Any questions? 

Now we're going to dive into Reading and Writing and Dennis will be explaining:

## Dennis on stage

In building everyday CRUD application I've noticed ... <your read - write story here >

... which brings us to the concept of state. 

## Sheet Aggregate State



## Sheet Shopping Cart State Machine






