#akka #mat #materializedValue #toMat #keep

Copied from [blog by Daniel Ciorcirlan](https://blog.rockthejvm.com/the-brilliance-of-materialized-values/#:~:text=The%20Jedi%20value%20returned%20by,elements%20flowing%20through%20the%20graph.)

# Materialized Values in Akka Streams

 6 minute read

This article is for the Scala programmer who has the very very basic familiarity with Akka Streams, in the sense that you can use some already made components, glue them together and start the stream, but otherwise no knowledge of behind-the-scenes functionality is required. If you’ve never used Akka Streams, I’ll give a brief overview.

## The setup [📌](https://blog.rockthejvm.com/the-brilliance-of-materialized-values/#the-setup "Permalink")

If you are curious enough to try this code for yourself, you’ll need to start an SBT project. I recomment IntelliJ because it’s easiest. Add the following to your build.sbt file:

```
val akkaVersion = "2.6.4"

libraryDependencies ++= Seq(
  // akka streams
  "com.typesafe.akka" %% "akka-stream" % akkaVersion
)
```

## A 30-Second Background[📌](https://blog.rockthejvm.com/the-brilliance-of-materialized-values/#a-30-second-background "Permalink")

Akka Streams is an implementation of the [Reactive Streams](https://www.reactivemanifesto.org/) specification for the JVM. It allows high-throughput and fault-tolerant streams of data by simply plugging in streaming components. They are sources (or publishers), flows (or transformers) and sinks (or subscribers). The names are self-explanatory: one produces elements, one transforms them along the way, and the last one consumes the data.

You can create streams of data in the following way:

```
import akka.actor.ActorSystem
import akka.stream.scaladsl.{Sink, Source}

// Akka Streams runs on top of actors for high-throughput
implicit val system = ActorSystem()

// streaming components
val source = Source(1 to 1000) // can wrap a normal collection and "emit" the elements one at a time
val flow = Flow[Int].map(x => x * 2) // transforms every incoming x into 2 * x
val sink = Sink.foreach[Int](println) // applies this function to every element that goes into it
val graph = source.via(flow).to(sink) // plug them together

// magic
graph.run() // prints 2, 4, 6, 8, ... on different lines
```

Obviously, there is a whole variety of sources, sinks and flows, but you can plug them all together in a myriad of ways and construct arbitrary pipes of data through which you can send elements.

## The Confusion[📌](https://blog.rockthejvm.com/the-brilliance-of-materialized-values/#the-confusion "Permalink")

Now, if you take a look at the types of the streaming elements in question, you will see the following (with the help of the compiler):

```
val source: Source[Int, NotUsed] = ...
val flow = Flow[Int, Int, NotUsed] = ...
val sink = Sink[Int, Future[Done]] = ...
```

The types there are confusing, especially for a newcomer. It’s quite understandable if you anticipated a source to be of type Source[Int] and a Flow to be of type Flow[Int, Int] much like a function, but what’s the third type doing? What’s the NotUsed and the Future[Done]?

At this point many Akka Streams programmers either ignore the third type and focus on what they understand, or try to read the docs and become entangled in a web of abstractions which have no correspondent in other code. Some of them do succeed, though, sometimes through reading the docs many many many (x10) times. It doesn’t have to be hard, and that third type is actually pretty damn great. Let me explain.

## The Jedi Values[📌](https://blog.rockthejvm.com/the-brilliance-of-materialized-values/#the-jedi-values "Permalink")

It’s one thing to run a stream: plug the stream components together and you obtain what is called a graph (pretty understandable), then you call the run method and the graph now has its own life.

But what kind of value do you get when you run a graph? Running a graph is an expression, much like anything else in Scala, so it must have a type and a value. What’s the value of running a graph?

I’m going to call that a Jedi value, for reasons that (I hope) will become apparent shortly. A Jedi value is what you obtain after running a graph. When we run

```
source.via(flow).to(sink).run()
```

the Jedi value of this graph is of type NotUsed, which is a bit like Unit and not useful for processing. Now here’s the thing with Jedi values: ALL streaming components in Akka Streams have a Jedi value when plugged into a living breathing graph, so in the above code we have 3 components and thus 3 Jedi values somewhere. However, the graph itself can only return ONE. So then the question becomes, which Jedi value does the graph return?

When you run something like source.via(flow).to(sink), the _left-most_ Jedi value will be picked. In our case, the Jedi value of the source. You can change that.

```
source.via(flow).toMat(sink)((leftJediValue, rightJediValue) => rightJediValue).run()
```

By calling the toMat method, you can pass a function that takes the left Jedi value before the last dot, and the sink’s Jedi value, and return something from them. In this case I’m returning just the right Jedi value. By calling toMat, I’ve changed the Jedi value of the graph, which will now take the Jedi value of the sink. You can use toMat in between any streaming operator to choose which Jedi value you want.

In this case above, the Jedi value of the graph is now the Jedi value of the sink, which is Future[Done]. So now you can use it:

```
import system.dispatcher
val future = source.via(flow).toMat(sink)((leftJediValue, rightJediValue) => rightJediValue).run()
future.onComplete(_ => println("Stream is done!"))
```

Notice that the Jedi value has a _meaning_, which is that the future can be monitored for completion and you can tell when the stream has finished running. Also notice that the Jedi value has _no connection whatsoever_ with the elements being shoved into the stream. That function that picks the right argument from the two arguments supplied to it can also be written as:

```
source.via(flow).toMat(sink)(Keep.right).run() // Keep.right is exactly (a, b) => b
```

or even shorter

```
source.via(flow).runWith(sink) // Keep.right is assumed
```

There are various components in Akka Streams that return various Jedi values. For example, if you want to extract the sum of all elements in a stream:

```
val summingSink = Sink.fold[Int, Int](0)((currentSum, newNumber) => currentSum + newNumber)
// remember from the previous snippet: runWith takes the rightmost Jedi value
val sumFuture = source.runWith(summingSink)
sumFuture.foreach(println)
```

Not only will you know when the future is done, but you will also know what the sum of the elements was. Notice that in this case we can use the elements in the stream to process them in a single value that the stream will return at the end (the Jedi value).

## Why are Jedi values great?[📌](https://blog.rockthejvm.com/the-brilliance-of-materialized-values/#why-are-jedi-values-great "Permalink")

Jedi values are awesome, because without them, once you start the graph, your pipes are opaque, sealed and irreversible. There’s no way of controlling streams - I’ll talk another time about how to shove data into a stream manually, or as a reaction to something. There’s no getting any information out of the stream. There’s no processing the values aside from flows. The world would be a very dark place.

Various streaming components in Akka streams can offer various Jedi values. Sinks usually offer Futures, which often allow you to combine the values inside the streams into one, and tell if/when the stream has finished. Some flows offer control mechanisms - like KillSwitches which allow you to stop the stream at any point. Some sources offer entry points through which you can send the data inside the stream.

I call them Jedi values because they are powerful. The Akka Streams library calls them _materialized values_. That’s because, when you plug components together, you have an inert graph, but when you call the run method, the graph comes alive, or is _materialized_. The Jedi value returned by materializing a graph is called a materialized value. The value may or may not be connected to the actual elements that flow through the stream, and the value can be of any type - which again, may or may not be different from the types of elements flowing through the graph.

I hope this article saved you the many hours it took me to internalize this abstract and often confusing concept.