---
layout: docs
title: Hello World
---

*The program described here is also available as `HelloWorld.scala` in the `examples` project.*

## Hello World

For our first **Tuco** program we will write a telnet server that greets the user, asks for her name, says hello, and then ends the session.

The first thing we need to do is bring **Tuco** types and constructors into scope. Fine-grained imports are also possible but in most cases this is the expected way to do things.

```tut:silent
import tuco._, Tuco._
```

We define the behavior of our server as a value of type `SessionIO[Unit]`. The server will take care of accepting the connection, negotiating a terminal type, etc., and will then interact with the user as we specify here.

```tut:silent
val hello: SessionIO[Unit] =
  for {
    _ <- writeLn("Hello World!")             // write a CRLF-terminated line and flush
    n <- readLn("What is your name? ")       // prompt and await input
    _ <- writeLn(s"Hello $n, and goodbye!")  // write again, and we're done
  } yield ()
```

To complete the specification of our telnet server we construct a `Config` with our behavior and various server options. The defaults are reasonable so we will just provide a port, which is the only required option.

```tut:silent
val conf = Config(hello, 6666)
```

We can now start our server by running the `.start` action, which starts up the server and returns an `IO` action that we can use to stop the server.

```tut:invisible
// define this before starting the server to ensure it compiles
val test = {
  Expect(conf)
    .expect("What is your name? ")
    .sendLn("Bob")
    .expect("Hello Bob, and goodbye!")
    .test
}
```

```tut
val stop = conf.start.unsafePerformIO
```

We can now connect to the server from another terminal window via `telnet`.

```tut:evaluated
// run our test and ensure the server stops; the call to stop below is a no-op
println(test.ensuring(stop).unsafePerformIO)
```

We can do this as many times as we like, with a the default maximum of 25 simultaneous connections. To terminate the server go back to the REPL and run the `stop` program.

```tut
stop.unsafePerformIO
```

In the next chapter we will try a more complex interaction by implementing a [Guessing Game](guessing-game.html).