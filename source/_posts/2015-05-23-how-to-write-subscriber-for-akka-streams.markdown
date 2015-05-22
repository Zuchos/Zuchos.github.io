---
layout: post
title: "How to write subscriber for akka-streams?"
date: 2015-05-23 00:05:10 +0200
comments: true
categories: 
---
Recently I started using *akka-http* and what I was trying to achive was to receive data from request, send response that the data were recieved succefully and then, process it asynchronsly. I started with empty *akka-http* service:

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
	
Now we want to add new route that will accept data from sender. For this purpose we are going to add new route to the defined routes.

	path("hello") {
	  get {
	    complete("Hello World!")
  	}
	}
	path("data") {
	  (post & entity(as[String]) & parameter('sender.as[String])) {
	    (dataAsString, sender: String) =>
	      complete {
	        HttpResponse(StatusCodes.OK, entity = "Data received")
	      }
	  }
	}
What is now missing is the Publiser that will publish data that came from http request into the akka-stream. To do that we need to define ```DataPublisher```. ```DataPublisher``` will be an implementation of ```ActorPublisher``` trait. It will be receiving data and then it will be publishing those to the next element in the flow.

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

As you may see, the main method is the receive method which is responsible for accepting the incoming data and responding on demand on data that is comming from subscribers. 
The last thing to implement is to define the processing flow:

	val dataPublisherRef = system.actorOf(Props[DataPublisher])
	val dataPublisher = ActorPublisher[Data](dataPublisher)
 
	Source(dataPublisher)
	  .runForeach(
	    (x: Data) =>
	      println(s"Data from ${x.sender} are being processed: ${x.body}")
	  )
	  .onComplete(_ => system.shutdown())
and then publish the incoming data:

	path("data") {
	  (post & entity(as[String]) & parameter('sender.as[String])) {
    	(dataAsString, sender: String) =>
	      complete {
	        dataPublisherRef ! Data(sender, dataAsString)
	        HttpResponse(StatusCodes.OK, entity = "Data received")
	      }
	  }
  
Now you application is ready to process data incoming data with akka-streams. You may find complete example on [github](https://github.com/Zuchos/akka-http-with-steams)