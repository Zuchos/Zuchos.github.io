---
layout: post
title: "How to write a publisher for akka-streams?"
date: 2015-05-23 00:05:10 +0200
comments: true
categories: [scala, akka, akka-http, akka-streams]
---
Recently I started using *akka-http* and what I was trying to achieve was to receive data from request, send response that the data were received successfully and then process it asynchronously. The other requirement was that the processing flow could be complicated in the future and some parts of it could be faster than other, so I decided to use *akka-streams* for that. I started with empty *akka-http* service:

{% codeblock SimpleService lang:scala https://github.com/Zuchos/akka-http-with-streams/tree/blogpost1/src/main/scala/pl/zuchos/example/NaiveGsServer.scala %}

  trait SimpleService {

    implicit val system: ActorSystem
    implicit def executor: ExecutionContextExecutor
    implicit val materializer: FlowMaterializer

    val routes = {
      path("hello") {
        get {
          complete("Hello World!")
        }
      }
    }
  }

  object NaiveGsServer extends App with SimpleService {

    override implicit val system = ActorSystem()
    override implicit val executor = system.dispatcher
    override implicit val materializer = ActorFlowMaterializer()

    val config = ConfigFactory.load()

    Http().bindAndHandle(routes, config.getString("http.host"), config.getInt("http.port"))

  }
{% endcodeblock %}

<!--more-->
Now we want to add new route that will accept data from sender. For this purpose we are going to add it to the routing definition.

{% codeblock lang:scala routes https://github.com/Zuchos/akka-http-with-streams/tree/blogpost1/src/main/scala/pl/zuchos/example/NaiveGsServer.scala %}
  val routes = {
    path("hello") {
      get {
        complete("Hello World!")
      }
    } ~
    path("data") {
      (post & entity(as[String]) & parameter('sender.as[String])) {
        (dataAsString, sender: String) =>
          complete {
            HttpResponse(StatusCodes.OK, entity = "Data received")
          }
      }
    }
  }
{% endcodeblock %}

What is now missing is the Publisher that will publish data that came from http request into the akka-stream. To do that we need to define ```DataPublisher```. It will be an implementation of ```ActorPublisher``` trait. It will be receiving data and then it will be publishing those to the next element in the flow.

{% codeblock lang:scala DataPublisher https://github.com/Zuchos/akka-http-with-streams/tree/blogpost1/src/main/scala/pl/zuchos/example/actors/FramePublisher.scala %}
  case class Data(sender: Option[String], body: String)

  class DataPublisher extends ActorPublisher[Data] {
    var queue: mutable.Queue[Data] = mutable.Queue()

    override def receive: Actor.Receive = {
     	case Publish(s) => queue.enqueue(s)
        publishIfNeeded()
      case Request(cnt) =>
        publishIfNeeded()
      case Cancel => context.stop(self)
      case _ =>
    }

    def publishIfNeeded() = {
      while (queue.nonEmpty && isActive && totalDemand > 0) {
        onNext(queue.dequeue())
      }
    }
  }

  object DataPublisher {
    case class Publish(data: Data)
  }
{% endcodeblock %}	

As you may see, the main method is ```receive()``` which is responsible for accepting the incoming data and responding on demand on data that is coming from subscribers.
The last thing is to define the processing flow.

{% codeblock lang:scala flow definition https://github.com/Zuchos/akka-http-with-streams/tree/blogpost1/src/main/scala/pl/zuchos/example/NaiveGsServer.scala %}
  val dataPublisherRef = system.actorOf(Props[DataPublisher])
  val dataPublisher = ActorPublisher[Data](dataPublisherRef)

  Source(dataPublisher)
    .runForeach(
      (x: Data) =>
        println(s"Data from ${x.sender} are being processed: ${x.body}")
    )
    .onComplete(_ => system.shutdown())
{% endcodeblock %}	  
and then publish the incoming data:

{% codeblock lang:scala publishing https://github.com/Zuchos/akka-http-with-streams/tree/blogpost1/src/main/scala/pl/zuchos/example/NaiveGsServer.scala %}
  path("data") {
    (post & entity(as[String]) & parameter('sender.as[String])) {
  	(dataAsString, sender: String) =>
        complete {
          dataPublisherRef ! Publish(Data(sender, dataAsString))
          HttpResponse(StatusCodes.OK, entity = "Data received")
        }
    }
{% endcodeblock %}
  
Now your application is ready to process incoming data with akka-streams. You may find complete example on [github](https://github.com/Zuchos/akka-http-with-steams)

Update: I developed this example a bit in my [next post](http://zuchos.com/blog/2015/06/10/more-reactive-publisher-aka-publisher-vol-2/)