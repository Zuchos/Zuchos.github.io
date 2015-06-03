---
layout: post
title: "More reactive Publisher (aka Publisher vol. 2)"
date: 2015-06-01 15:06:51 +0200
comments: true
published: true
categories: [akka, akka-http, akka-streams]
draft: true

---

In my previous post I created ```Publisher``` for akka-streams that was buffering incoming data and then passing those down the stream. Johannes Rudolph aptly observed that the flow of that solution is the buffer overflow scenario (too many incoming requests may lead to out-of-memory issue).

>Thanks for the post! It's nice to see that people are actually starting to use akka-stream and akka-http. A note: implementing `ActorPublisher` shouldn't be necessary in most cases. In this case you built an unbounded buffer in front of a stream which defeats akka-stream/reactive streams back pressure logic. Now if the consumer cannot keep up with reading the data all the unwritten data will start to pile up in memory. Generally, it isn't possible to switch from a pull-style (akka-stream/reactive-stream) model to a push-style model (actor message tell) somewhere in the processing chain. In cases where you still need to do this (e.g. because you are dealing with a "live" data source) there's a somewhat safer option: use [`Source.actorRef`](https://github.com/akka/akka/blob/release-2.3-dev/akka-stream/src/main/scala/akka/stream/scaladsl/Source.scala#L342) which lets you define a limited buffer and makes you choose a strategy what to do when the buffer is full. <cite>Johannes Rudolph</cite>

First I would like to explain that this case comes from my pet project. In that project I'm expecting that users around the world will send me the data, so I want to make API as simple as possible (what could be simpler than REST API?).
Users are not interested in the result of computation, they are interested in contributing the data. So the system should accept as many data as possible (return status OK to the user - it would mean that we received the data) and the process it with it's own speed. I rather expect to have many request from different user than one user will be sending tons of those.
The buffer overflow is possible situation here so Johannes was right that it should be tackled. First I took a look on proposed solution ```Source.actorRef()```. The problem with ```ActorRefSourceActor``` is that the all available ```OverflowStrategy``` values are not notifying sender that the problem occurred and that leads to lost of data. So I couldn't use that solution.

So I came up with different one, I added bufferSize ```val``` to ```DataPublisher``` and in receive method I extracted ```cacheIfPossible()``` method:

{% codeblock SimpleService lang:scala https://github.com/Zuchos/akka-http-with-streams/tree/blogpost1/src/main/scala/pl/zuchos/example/NaiveGsServer.scala %}

  override def receive: Actor.Receive = {
    case Publish(s) =>
      cacheIfPossible(s)
    case Request(cnt) =>
      publishIfNeeded()
    case Cancel => context.stop(self)
    case _ =>
  }

  private def cacheIfPossible(s: Data) {
    if (queue.length == bufferSize) {
      sender() ! Failure(new BufferOverflow)
    } else {
      queue.enqueue(s)
      sender() ! Success()
      publishIfNeeded()
    }
  }

{% endcodeblock %}

<!--more-->

So the main change here is that we are testing buffer size, if the buffer is full we respond with ```Failure``` to the Sender else we are adding data to the buffer and sending back ```Success```. In ```SimpleService``` we are **asking** instead of **talking** to actor and then map the answer (```Try```) to the correct ```HttpResponse```.


{% codeblock SimpleService lang:scala https://github.com/Zuchos/akka-http-with-streams/tree/blogpost1/src/main/scala/pl/zuchos/example/NaiveGsServer.scala %}

      path("data") {
        (post & entity(as[String]) & parameter('sender)) {
          (dataAsString, sender: String) =>
            complete {
              val publisherResponse: Future[Any] = dataPublisherRef ? Publish(Data(sender, dataAsString))
              publisherResponse.map {
                case Success(_) => HttpResponse(StatusCodes.OK, entity = "Data received")
                case Failure(_: BufferOverflow) => HttpResponse(StatusCodes.ServiceUnavailable, entity = "Try again later...")
                case _ =>
                  HttpResponse(StatusCodes.InternalServerError, entity = "Something gone terribly wrong...")
              }
            }
        }

{% endcodeblock %}

That solution is better than the previous one and for my case it fits better than using `Source.actorRef`. It's also not fully reactive (we don't forward back pressure to the client) and we trust that the declared buffer will fit in memory (I tried to use [`FixedSizeBuffer.scala`](https://github.com/akka/akka/blob/release-2.3-dev/akka-stream/src/main/scala/akka/stream/impl/FixedSizeBuffer.scala) but it's a private class).
On the other hand it still exposes simple REST API and process data asynchronously.