# EventEmitter.NET
A type-safe generic EventEmitter inspired by NodeJS's event emitter.

## Install

Install via NuGet

```shell
dotnet add package EventEmitter.NET
```

## Usage

Before you can listen or emit events, you must first create an event emitter. An event emitter
can be made by creating a new `EventDelegator` instance

```c#
var events = new EventDelegator();
```

The `EventDelegator` is responsible for routing triggered events to their respective
event listeners. The `EventDelegator` is context safe by default, it uses a context string
to define which context its running in. This means two instances of `EventDelegator` cannot
trigger each-other's event listeners. If required, a context string can be provided to the
`EventDelegator` constructor

```c#
var events = new EventDelegator("my-unique-context-string");
```

### Triggering events

To trigger events, simply provide an `eventId` string and the event data object to pass
to the event listeners. Only event listeners who are listening to the `eventId` with the same
event data type will be triggered.

```c#
var events = new EventDelegator();

events.Trigger("my-event-id", "some-data");
```

This will invoke the registered callbacks of the event listeners listening to the eventId `my-event-id` and looking for the `string` event data type. 
Any event data type can be passed.

```c#
interface ITest
{
    public int test1 { get; }
}

public class TestEventData : ITest
{
    public int test1 { get; set; }
    public string test2;
}

var events = new EventDelegator();

var testData1 = new TestEventData()
{
    test1 = 11,
    test2 = "abccc"
};

events.Trigger("test-event", testData1);
```

This will also trigger event listeners looking for any sub-type of `T`, such as subclasses or interfaces. In the example above,
listeners listening to the following event data types will trigger

* `object`
* `ITest`
* `TestEventData`

### Listening to events

Any of the following functions of `EventDelegator` can be used to
listen for events:

* `ListenFor(string eventId, Action callback)`
  * Listen for all event triggers of `eventId`, regardless of event data type
* `ListenFor<T>(string eventId, EventHandler<GenericEvent<T>> callback)`
  * Listen for event triggers of `eventId` with the event data type `T`
* `ListenForOnce<T>(string eventId, EventHandler<GenericEvent<T>> callback)`
  * Listen for event triggers of `eventId` with the event data type `T` only once.

Expanding from the previous example, here is how you would listen to events

```c#
interface ITest
{
    public int test1 { get; }
}

public class TestEventData : ITest
{
    public int test1 { get; set; }
    public string test2;
}

var events = new EventDelegator();

// Listen to events
events.ListenFor<TestEventData>("test-event", delegate(object sender, GenericEvent<TestEventData> @event)
{
    Console.WriteLine("I get invoked with TestEventData");
});

events.ListenFor<ITest>("test-event", delegate(object sender, GenericEvent<ITest> @event)
{
    Console.WriteLine("I get invoked with ITest");
});

events.ListenFor<string>("test-event", delegate(object sender, GenericEvent<string> @event)
{
    Console.WriteLine("I never get invoked!");
});


var testData1 = new TestEventData()
{
    test1 = 11,
    test2 = "abccc"
};

events.Trigger("test-event", testData1);
```

### Helper Functions

If you are building an API that will expose dynamic events, you can have your class implement the 
`IEvents` interface. This interface only exposes an `EventDelegator Events { get; }` property. Once you
inherit this interface, your class can then be used with the `EventsExtensions` extension functions
to add the following functionality to your class

* `On(string eventId, Action callback)`
* `On<T>(string eventId, EventHandler<GenericEvent<T>> callback)`
* `Once<T>(string eventId, EventHandler<GenericEvent<T>> callback)`
* `Off<T>(string eventId, EventHandler<GenericEvent<T>> callback)`

These extension functions wrap the `EventDelegator` API to provide easy integration