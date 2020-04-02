# Trace Agent

A java agent for tracing which configurable by an embedded simple text file. 
As the config file is resource within the agent jar no rebuild is needed for tracing methods.


# The example

It is much easier to understand how this can be used if I show you it through an example.

## The project we would like to trace

Let's say we have a project what we would like analyze. In this example its code very simple:

```java 

package net.test;

public class App {
    public static void main( String[] args )
    {
        new TestClass().test();
        TestClass2nd testClass2nd = new TestClass2nd();
        testClass2nd.anotherMethod();
        testClass2nd.methodWithArgs("secret", 42);
    }
}


class TestClass {

  public void test() {
      System.out.println("Hello World!");
      try {
        Thread.sleep(100);
      } catch(Exception e) {
      }
  }
}

class TestClass2nd {

  public void anotherMethod() {
      System.out.println("2nd Hello World!");
      try {
        Thread.sleep(100);
      } catch(Exception e) {
      }
  }

  public void methodWithArgs(String str, int i) {
      System.out.println("methodWithArgs");
  }
}

```

Which can be executed as:

```
$ java -jar testartifact-1.0-SNAPSHOT.jar
Hello World!
2nd Hello World!
methodWithArgs
```

## Let's trace it

If we would like to:
- measure the elapsed time of the `test` method in nanosecond
- see the call stack at the beginning of `anotherMethod`
- and measure the elapsed time in milliseconds also within the `anotherMethod` 
- the trace the actual argument values used for calling the method `methodWithArgs`

without touching the testartifact then we could set up the `actions.txt` (the config of the trace agent) like this:

```
elapsed_time_in_nano net.test.TestClass test
elapsed_time_in_ms net.test.TestClass2nd anotherMethod
stack_trace net.test.TestClass2nd anotherMethod
trace_args net.test.TestClass2nd methodWithArgs
```

This `actions.txt` is part of the trace agent jar as a resource (no recompile/rebuild is needed just edit the file within the jar).

And to start the trace one could use:

```
$ java -javaagent:target/trace-agent-1.0-SNAPSHOT.jar -jar testartifact-1.0-SNAPSHOT.jar
Hello World!
TraceAgent (timing): `public void net.test.TestClass.test()` took 101639483 nano
TraceAgent (stack trace):
        at net.test.TestClass2nd.anotherMethod(App.java)
        at net.test.App.main(App.java:8)
2nd Hello World!
TraceAgent (timing): `public void net.test.TestClass2nd.anotherMethod()` took 100 ms
TraceAgent (trace_args): `public void net.test.TestClass2nd.methodWithArgs(java.lang.String,int) called with [secret, 42]
methodWithArgs
```


## The config format

The config format is simple lines with the following structure:

```
<action-name> <class-name> <method-name>
```
