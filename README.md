# Chapter 6: Isolate Process Machinery in a Boundary

> The boundary cleanly executes core code in a process and wraps it in a generic API.
The lifecycle layer provides tools to start and stop the boundary layer, even in the context of a larger project.
Workers divide work for performance, isolation, or reliability. You’ll see all of these layers in the following figure.”
- pg 99

## Maintain Composition Through Uncertainty

Back pressure - a technique to slow requests to services under duress.
Pipes that file midstream are difficult to handle. Treating errors as data helps them work with the `|>` operator
Avoiding raising also stops other work from being blocked.

**Question:** For the “Halt on error with context:” part. Why were there two errors before it error'ed on 5?

With block helps compose and put error handling off until the end. It makes it easy for us to write the happy path, but still catch data in an unexpected shape.

## Build Your Optional Server

There are times to consider using processes:
- Sharing state across processes
- Presenting a uniform API for external services such as databases or communications
- Managing side effects such as logging or file IO
- Monitoring system-wide resources or performance metrics
- Isolating critical services from failure

The boundary is an optional layer to separate impurities from the core.

>“A boundary needs a service layer around each individual process type and an external API for clean, direct access across services.”
- pg105

![](https://i.imgur.com/dqythN6.png)

Using GenServer to wrap our functional code. Our main tools:
- Init - set initial state
- Handle_call - sync
- Handle_cast - async

We can write a bunch of call functions to handle different tasks, and even create easy to use functions on the module so we don’t have to manually make calls.

Note: We only have one instance of our QuizManager running at a time (that’s why we name it).

Quiz session’s state is {quiz, email}, so each user needs their own QuizSession.
Session here is a pid

We now have QuizManager and Quiz session, but we need an API to bring them together.

## Wrap the Server in an API

The API insulates the GenServers from the external inconsistent data
We will validate at the API level because that is the closest common access point to the user.
Changesets are great for this, but we don’t want to bring in Ecto just yet

Question: How would changesets work here? They don’t seem like they’re written in a place that’s ‘close to the user’

The validator seems pretty confusing the way it was built up.
The validators make great use of composition

The API layer is a great place to stitch together disparate concepts, but you want to keep it as thin as possible. We are validating here because we don’t want the core to have to worry about bad data.

The API creates the quiz, makes templates,  and starts quiz sessions. This makes sense as these are things that users would want to do.

Interacting with a quiz is now really easy through the API and there is no need to know about GenServers, casts, or calls...

## Prefer Call Over Cast to Provide Back Pressure

Is OTP dead then!?
Serializability and back pressure

E.g. Logger. If it’s getting overloaded with cast messages, it switches to using call, thus slowing the entire app down. This allows Logger to catch up. If it gets real bad, logger just starts dropping messages. These three modes are :async, :sync, :discard
Note: Dave Thomas was talking about this, but now I understand it better.

It feels like we’re missing talking about this bottleneck and what we care most about here. As in, what’s more important, the app working quickly? Or logs?

## Extend Your APIs Safely

We need to be careful when updating an API to ensure that it’s not a breaking change. Here are some good guidelines:
- Don’t add new requirements to existing APIs, only options.
  - Adding new requirements forces you to upgrade all implementation sites at the same time and now we’ve lost the decoupling that the API was supposed to give us.
- Ignore anything you don’t understand
  - Question: I’m not sure I understand this one?
- Don’t break compatibility; Provide new endpoints
  - If new functionality is needed that takes in different inputs, just create a new endpoint rather than trying to extend an existing one.

## Wrap Your Core in a Boundary API

`with` helps us right compossible code and easily catch unexpected values
Server layer - the two GenServers `QuizManager` and `QuizSession` to encapsulate state and handle communications between processes.
Built a compossible validator
Built and API to bring it all together.
