# CQRS – Reads & Writes

## Introduction and Concepts

- As stated in the previous section, _CQRS_ stands for _Command Query Responsibility Segregation_.
- As the name states, it's a pattern which revolves around the segregation between _Commands_ and _Queries_, in other words, _Writes_ and _Reads_.
- Most traditional application architectures are similar to CRUD datastores. There's one mental model which is used to create, read, update, and delete records.
- In most applications I've built this model starts to break down in the long run. The model you've defined isn't optimal for every type of query you want to perform or manipulating seemingly simple data is overly complex because of choices made in the past.
- Sometimes, this is because you'd like to retrieve data from your model which isn't really possible because the information you need is scattered over lots of objects.
- Sometimes, this is because of the different ways you'd like to manipulate your data. This often leads to a slew of relationships between the objects in your model. Some are relevant in one context, while others are only relevant in a completely different context.
- What CQRS brings to the mix is seperating these concerns into two distinct conceptual models, one for writing and one for reading. These are respectively called the _Command_ model and the _Query_ model.
- The reasoning behind this is that a single conceptual model is usually only good at one of these two (either reading or writing).
- Splitting it up allows you to define a model defined specifically for its purpose. This allows you to mitigate a lot of complexity which is created by mixing these responsibilities in a single model.
- {>>Should we even mention this? This might just confuse the audience more than it adds anything to the talk.<<}Naturally, there's a relationship between these models. Sometimes they translate to a single model on the database level and your application just projects this model in two different ways on the application level. There are other, more common ways to translate between the two in CQRS land though, but we'll get to that later.
- Now, how does this translate to practical software?

## In Practice

- As mentioned in the previous section, Bertrand Meyer stated that a method should either change the state of an object, or return a result, but not both.

### Commands

-  The _Command_ model is fully aimed to provide ways to change the state of an object.
- In CQRS, the _Command_ model is accessed using _Commands_.
- _Commands_ are simple objects that contain all the data required to perform an action. The name of the _Command_ states its intent (i.e. what the action actually is). In Java, this means that _Commands_ are _POJOs_ where the class name describes the action while the fields of the class provide the information needed to perform the action.

### Queries

- Where the interaction with the _Command_ model is new and shiny, the interaction with the _Query_ model should look familiar to everyone.
- Developers interact with the _Query_ model in the exact same way they did before, with the exception that the state of the objects forming the model cannot be changed directly. They can only be changed by sending _Commands_ to the _Command_ model.

## Demo

- We've mentioned a lot of terms so far. Let's see how this translates into an actual application. We've prepared an example project revolving around the familiar shopping cart. {>>What will we show in the demo? From my perspective this is still the introductory _chapter_ and we should focus mostly on the creation and sending of _Command_ objects and seeing the results of the _Commands_ in the _Query_ model.<<}