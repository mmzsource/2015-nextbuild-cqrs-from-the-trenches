# CQRS – Reads & Writes

## Introduction and Concepts

- As stated in the previous section, _CQRS_ stands for _Command Query Responsibility Segregation_.
- As the name states, it's a pattern which revolves around the segregation between _Commands_ and _Queries_, in other words, _Writes_ and _Reads_.
- Most traditional application architectures are similar to CRUD datastores. There's one conceptual model which is used to create, read, update, and delete records.
- In most applications I've built this model starts to break down in the long run. The model you've defined isn't optimal for every type of query you want to perform or manipulating seemingly simple data is overly complex because of choices made in the past.
- Sometimes, this is because you'd like to retrieve data from your model which isn't really possible because the information you need is scattered over lots of objects.
- Sometimes, this is because of the different ways you'd like to manipulate your data. This often leads to a slew of relationships between the objects in your model. Some are relevant in one context, while others are only relevant in a completely different context.
- What CQRS brings to the mix is seperating these concerns into two distinct conceptual models, one for writing and one for reading. These are respectively called the _Command_ model and the _Query_ model.
- The reasoning behind this is that a single conceptual model is usually only good at one of these two (either reading or writing).
- Splitting it up allows you to define a model defined specifically for its purpose. This allows you to mitigate a lot of complexity which is created by mixing these responsibilities in a single model.
- Besides splitting up the _Command_ and _Query_ model this also opens the possibility for multiple _Query_ models, each optimized for a specific purpose. One of these could resemble a regular relational model, while another could resemble an audit log.
- Separating models also allows us to use different technology stacks for each of them. Although using a different programming language for each model might not be beneficial for your application, using multiple storage technologies might be (polyglot persistence).
- Now, how does this translate to practical software?

## In Practice

- As mentioned in the previous section, Bertrand Meyer stated that a method should either change the state of an object, or return a result, but not both.

### Commands

-  The _Command_ model is fully aimed to provide ways to change the state of an object.
- In CQRS, the _Command_ model is accessed using _Commands_.
- _Commands_ are simple objects that contain all the data required to perform an action. The name of the _Command_ states its intent (i.e. what the action actually is). In Java, this means that _Commands_ are _POJOs_ where the class name describes the action while the fields of the class provide the information needed to perform the action.

### Queries

- Where the interaction with the _Command_ model is new and shiny, the interaction with the _Query_ model should look familiar to everyone.
- Developers interact with the _Query_ model in the exact same way they did before, with the exception that the state of the objects forming the model cannot be changed (directly).
- As you can see these two models are completely separated but they come together at the user's end. The _Query_ model is used to provide the data for the user to interpret. Based on the data that is shown to the user he or she decides to perform a certain action. This action is translated into a _Command_ and this is sent to the _Command_ model. Based on the effects of this _Command_ the _Query_ model will be updated to represent the new state of the system.

## Demo

- We've mentioned a lot of terms so far. Let's see how this translates into an actual application. We've prepared an example project revolving around the familiar shopping cart.
- What do we want to illustrate in the demo?
	- We want to show what a _Command_ is. The fact that the action which should be performed is dependent upon the class name might appear strange to some people.
	- We want to show how a _Command_ is sent. Contrary to a regular method call, there is a disconnect between the code performing the action and the code requesting the action (this might be dependent on the CQRS implementation, but this is the case with Axon).
	- We want to show that a sent _Command_ leads to a changed _Query_ model.
	- We want to show how different the _Query_ models can be (e.g. an audit log, a more relational model).