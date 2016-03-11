# Simplifying BPM applications architecture

BPM industry clients have been wishing for something known as "zero code" myth. 

People desire systems that do things magically without having to write any code. They tried it, but they found that in the end they have to tell the computer what to do in each situation, and that is exactly all coding is about...

BPM tried to solve this problem by doing some things graphically, at least to enable the understanding of business people of what an application is designed to act like.

They removed the writing of a lot of unfamiliar syntax for non developer people and replaced it with dragging a dropping some symbols into the screen to tell the computer what to do. This is actually another way of coding but it's disguised.

Let's be pragmatic, it's imposible to replace the entire possible combinations of code that could be written with a bunch of symbols, that's simply unrealistic. You can rely on BPM to define state transitions but, at the end of the day any other activity will require to write code to perform custom tasks.

Anyway, zero code paradigm it's a problem itself and something not desirable because it comes without real flexibility.

The good thing, is that BPM solves a problem fairly easy in a visual declarative way, that in traditional imperative style coding (java) is more obscure and hard to solve, that is state transition and management. 

That sounds beautiful, a system in which we can declaratively define what things are, and how they change their state, again, **declaratively** with a few symbols.

Then it's easy to imagine some interception points that with code are able to do fill whatever need we want to fulfill.

You could even declare visually at some point "Ok coffee maker, now make me some coffee!", of course, it's a joke… But you get the idea.



I now want to enumerate the pros and cons of a BPM product.

### The bad

Todays BPM solutions are very monolithic and they have some really bad disadvantages:

- They get outdated fast because the release cycle is slow.

- They are always *AllInOne* solutions instead of *TakeWhatYouNeed*, which is normally the desired case.

- They are very heavy weighted.

- They are usually coupled to some IDE's.

- They are coupled to the technology used by it's own frameworks.

- They are highly opinionated.

- Because they are normally based on enterprise Java stacks, they require specific application servers to run, what is not always the case.

- They come with a default workspace as the endpoint for the user with a fixed UX and so poorly customizable

  ​

### The good

In the other hand, BPM solutions have come with some builtin interesting features:

- They are based on standards.


- They usually provide a Security Layer, that can be replaced or extended.

- They provide abstractions for users, organizations and memberships.

- They implement auditing systems.

- They implement the concept of a **workflow engine** and a **workflow editor**.

  ​

# Our proposal



### The idea

What If we were able to abstract what a workflow engine API should look like in a generic way and create some authentication/authorization layer that could work across BPM/whatever system with a small provider based setup?

Briefly said, can we use facades for the core things that BPM is good at and designed for, just to be agnostic of what technology we can use?

The answer is yes.

In that scenario, we have break the boundaries imposed by the tech monolith and now we can start writing applications by loosely interfacing with thoose facade API's and become the real owners of all the data that flows inside the system.

This opens the door to rapidly prototyping and mocking every part of our application and to do the implementation in isolation from other layers. That finally leads our teams to be faster and to understand what is happening inside in an easier way.



### To be backend or to be frontend

This section has a more philosophical intention and tries to convince about the power of the idea above.

> Abstracting this away, we now could consider the BPM the backend of the whole domain, and our workspace plus api's plus applications it's frontend, i'm not talking about UI. I'm disecting it in the same sense that compilers have frontends (Java or Objective-C) and backends (JVM or LLVM). So backends are compile targets for frontends, but they theoretically could run over any the available backends with the right compiler.

This means that we could use other backend BPM while mantaining the same Frontend (UI, API contracts, facades..). This makes the BPM a plug-in and would let it care about the state of the applications and it's transitions, that generally is the hardest part of any application and BPM solves it just with "drag & drop".

Enterprise world does not have this POV yet, but decoupling and simplifiying architecture allows in a future to change the backend technology or upgrade it while keeping all the end to end systems working in the same way.

So basically, since BPM products implement standards, and they are implementations of the same concepts, let's abstract them and consider it an interchangable brain/core for our applications that solves easily a very hard problem, **state**.

Finally, from a development point of view, it's important to highlight that this comes with a very interesting implicit benefit: 

Our developers just have to **learn once** how to develop applications, and those will **run everywhere**, on top of any BPM.

* Simplified learning curve


* Gain expertise quickly


* Become profitable


* No coupling to an specific product



*That reduces the amount of work amazingly, and lets us build high quality applications in a relatively small amount of time.*



# Our solution



It's pretty obvious that we came with an implementation of that idea, and this is how we did it.



#### Wrapping the Workflow Engine's API

Before we got this idea, we started proxying Oracle's BPM Workflow engine in 11g which exposes SOAP services. We mapped them to equivalent REST Services in a RPC style. 

That unlocked the capability for easily writing RIA applications (Rich Internet Applications) and still allowing the use and the benefits that a BPM can offer.

 And even better, we now were able to write them on the clientside.



#### Creating the security abstraction layer

Our next iteration was to wrap it instead doing proxy. At this point we where able to design our own security system based on existing standards, and to be specific: 

- JOSE (Javascript Object Signing and Encription)
- JWS (JSON Web Signatures)
- JWT (JSON Web Token)

> No more sessions, no more cookies, we just use tokens, and that's the most minimalistic yet powerful approach.

Over this abstraction we can integrate any authentication protocol, and the applications just use.. Tokens!


As an example, we implemented SSO (Single-Sign-On) feature using this approach.

Those can contain any information about the user and his memberships. They can have expiracy time, be revokable.. whatever, but they are stateless and that makes them easy to be used by any server or client, even native ones.



#### Building a custom workspace

Our idea here was to build a small layer on top of the different applications we could embed inside the same domain. 

The idea is tu build an light and configurable UI application orchestrator that just links different applications and service providers together and manages their lifecycle.

Let me empathize that even if they are completely separated, this layer act's as a UI Virtual Machine layer and glues them together, it's like an application host.

This are the benefits we consider:

- Single entry point, a login form.

- BPM application dashboard. 

  - This makes segmentation between groups of processes.
  - Each BPM application is represented in a tile on the dashboard.
  - They are configurable to the point that we can express what kind of permissions or organizations are required for a recently entered user, and make auto assignation between them.

- Each tile of the dashboard drives navigation to a customized task/case worklist from which users can open forms (configured as external applications).

- Each tile has a subset of extenal sub applications that users can use and navigate to. Usually we use this to implement custom **instance creation**, **search**, **administration**.. kind of applications.

- Single and simple source of configuration.

- It's a client side application:

  - Backend will only have the weight of instance processing and data exchange with the database

  - There's no UI rendering nor state management on the server-side.

    > Since Serverside UI Rendering, string processing at the end, is one of the most heavy operations for a CPU, having a lot of users that use an application exponentially increases the CPU usage and decreases performance of the server.
  - Todays computers have multiple cores and high amounts of RAM. So doing clientside applications is a way of clustering by using your users computers.


# The result



After implementing all these ideas, we have finally an end to end solution, that it's generic, flexible and simple to use for users and developers. 

>  The screenshots below show a styling made for a specific brand or client.



## Login

The login is our entry point to the workspace.
* Uses the identity API to log in.

* Generates a JWT token that is shared across every application deployed in the same domain.

* It's highly customizable in terms of style

  ​

This is how our entry point looks like for one of our clients.

![login](https://raw.githubusercontent.com/aitoroses/simplifying-bpm-architecture/master/images/login.png)





## Dashboard



The dashboard is the landing interface for the user, from here he can access any application, even if it's not a BPM based.

Same as login, the style and tiles layout is completely customizable.



![dashboard](https://raw.githubusercontent.com/aitoroses/simplifying-bpm-architecture/master/images/tile-view.png)







## Main application and Case/Task Worklist

Just the extension of the dashboard:

* It contains a default BPM application that shows process instances filtered by custom filters declared on the configuration.

* Clicking on a row will load with some fancy animation the form inside the main view.

* Other applications are accessible from tabs, and they load inside the main view.

  ​

![default-app](https://raw.githubusercontent.com/aitoroses/simplifying-bpm-architecture/master/images/default-app.png)





# Conclusion



BPM solves a real world application problem that's is very common, state management and transitions.

Zero code myth is not something realistic, and if it was, we would need an infinite symbol list to represent everything that could happen. It's better to limit that to events, transitions, decisions, and activities, but activities must be coded.

Anything else is more likely to be delegated to other systems so they can be built in an optimal way and with the correct patterns and abstractions.

Now we have unlocked a way to build applications with any technology stack of our choice but we are still able to use the best of BPM together.

And the good news, this solution it's not coupled to any BPM product at all.



Until next time,

Aitor