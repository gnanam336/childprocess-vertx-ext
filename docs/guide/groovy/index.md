# Child Process extension for Vert.x

Child Process is a Vert.x component for spawning OS child processes.

* based on https://github.com/brettwooldridge/NuProcess _Low-overhead, non-blocking I/O, external Process implementation for Java_.
* standard streams are fully non blocking
* spawned process can be killed

## Using Child Process

To use Child Process, add the following dependency to the _dependencies_ section of your build descriptor:

* Maven (in your `pom.xml`):

```xml
<dependency>
 <groupId>com.julienviet</groupId>
 <artifactId>childprocess-vertx-ext</artifactId>
 <version>1.3.0</version>
</dependency>
```

* Gradle (in your `build.gradle` file):

```groovy
dependencies {
 compile 'com.julienviet:childprocess-vertx-ext:1.3.0'
}
```

## Spawning child processes

You can spawn child processes with the [`Process.spawn`](../../apidocs/com/julienviet/childprocess/Process.html#spawn-io.vertx.core.Vertx-java.lang.String-) method:

```groovy
Process.spawn(vertx, "ls")

```

you can give arguments to child processes

```groovy
Process.spawn(vertx, "ls", ["-lh", "/usr"])

```

by default child processes use the current process environment options, you can pass key-value pairs
as new environment variables

```groovy
def env = [:]
env["MY_VAR"] = "whatever"
Process.spawn(vertx, "ls", [
  env:env
])

```

[`Process.env`](../../apidocs/com/julienviet/childprocess/Process.html#env--) gives you the current process environment key-value pairs

```groovy
def options = [
  env:Process.env()
]
Process.spawn(vertx, "ls", options)

```

By default, the child processes uses the current process _current working directory_, the
[`setCwd`](../../apidocs/com/julienviet/childprocess/ProcessOptions.html#setCwd-java.lang.String-) option overrides it

```groovy
def options = [
  cwd:"/some-dir"
]
Process.spawn(vertx, "ls", options)

```

## Interacting with child processes

The child process streams are available as

* [`stdin`](../../apidocs/com/julienviet/childprocess/Process.html#stdin--)
* [`stdout`](../../apidocs/com/julienviet/childprocess/Process.html#stdout--)
* [`stderr`](../../apidocs/com/julienviet/childprocess/Process.html#stderr--)

```groovy
def process = Process.spawn(vertx, "cat")

process.stdout().handler({ buff ->
  println(buff.toString())
})

process.stdin().write(Buffer.buffer("Hello World"))

```

Calling [`kill`](../../apidocs/com/julienviet/childprocess/Process.html#kill--) kills the child process, on POSIX it sends the
`SIGTERM` signal.

```groovy
def process = Process.spawn(vertx, "cat")

process.stdout().handler({ buff ->
  println(buff.toString())
})

process.stdin().write(Buffer.buffer("Hello World"))

// Kill the process
process.kill()

```

Child processes can also be forcibly killed

```groovy
def process = Process.spawn(vertx, "cat")

process.stdout().handler({ buff ->
  println(buff.toString())
})

process.stdin().write(Buffer.buffer("Hello World"))

// Kill the process forcibly
process.kill(true)

```

## Child process lifecycle

You can be aware of the child process termination

```groovy
def process = Process.spawn(vertx, "sleep", ["2"])

process.exitHandler({ code ->
  println("Child process exited with code: ${code}")
})

```

## Delayed start

Calling [`Process.spawn`](../../apidocs/com/julienviet/childprocess/Process.html#spawn-io.vertx.core.Vertx-java.lang.String-) starts the process after the current event loop task
execution, so you can set handlers on the process without a race condition.

Sometimes you want to delay the start of the child process you've created, for instance you are creating a process
from a non Vert.x thread:

```groovy
def process = Process.create(vertx, "echo \"Hello World\"")

process.stdout().handler({ buff ->
  println(buff.toString())
})

// Start the process
process.start()

```