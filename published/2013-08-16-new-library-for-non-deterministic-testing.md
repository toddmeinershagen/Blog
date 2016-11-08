# New Library for Non-Deterministic Testing

I have currently been working on creating a system-level, feature test for some functionality that our company is developing to allow hospitals and health care providers to request a batch of eligibility requests for patients using a combination of Visual Studio 2012, [SpecFlow 1.8](http://www.specflow.org/specflownew/) and [FluentAssertions](http://fluentassertions.codeplex.com/).  In doing so, I ran into the problem of non-deterministic time.  

Once a user uploads a file of requests, there is not a determined amount of time for that request to get a response from a third-party provider that we use to fulfill the request.  So how do we handle the fact that we need to make some assertions at the end of the upload?  What would be nice would be to either retry assertions that fail or keep trying an assertion in the event of failure for a given timeout period.  In this case, we need to know that we can get a response within 30 seconds.

Enter [James.Testing](https://github.com/toddmeinershagen/James.Testing).  (Check it out when you get a chance.  You can download the dependency to your project from NuGet.org [here](https://www.nuget.org/packages/James.Testing/).

[James.Testing](https://github.com/toddmeinershagen/James.Testing) is a library of test utilities named after the author who wrote the book of James in the Bible.

> "Dear brothers and sisters, when troubles come your way, consider it an opportunity for great joy. For you know that when your faith is tested, your endurance has a chance to grow." **(James 1:2-3)**

It's a fairly apt description of what testing ought to do for our applications as well.  Source for the library can be found [here](http://www.github.com/toddmeinershagen/james.testing).  Below are some of its features. 

## Action Extensions

### Executing an Action with Retries

Many times in integration tests, there is a non-deterministic time period between executing some initial action and asserting your expectations for the outcome.  In this case, it would be nice to have a method for automatically having the test retry your action for a number of times even though the assertion fails.  This method also supports setting a wait time in between retries so that you don't overload your system.

**Example:**

```csharp
var counter = 0;
Action action = () => counter++;
action.ExecuteWithRetries(times, waitTimeInSeconds);
```

### Executing an Action with a Timeout Period

In other cases, you might want to execute a given action for a given time period.  For instance, if you have a requirement to expect a response within 30 seconds, you can set the maximum timeout period to 30 seconds for your assertions.  This method also allows a user to set a given wait time between executions of a given action.

**Example:**

```csharp
var counter = 0;
Action action = () => counter++;
action.ExecuteWithTimeout(timeoutInSeconds, waitTimeInSeconds);
```

### Executing an Action while Gulping Exceptions

Sometimes when you execute an action you expect an exception to occur, but you don't want the exception to cause a failure because there is something else you need to verify.  This method will do just the trick.

**Example:**

```csharp
var counter = 0;
Action retryAction = () =>
	{
		while (counter++  
{
    Thread.CurrentThread.CurrentCulture = new CultureInfo("fr-FR");
    session = new Session(new LocalPhantomEnvironment());

    session.CurrentCulture.Name.Should().Be("fr-FR");
};

action.ExecuteWithCleanup(() => session.End());
```

## Wait Methods

### Waiting For a Time Period

Many times in multi-threaded and integration tests, you need to wait for a number of seconds or milliseconds for other events to process.  James.Testing now provides a more readable syntax for these events.

**Example:**
```csharp
Wait.For(1).Seconds();
Wait.For(250).Milliseconds();
```

### Waiting Until Something is True

Sometimes in multi-threaded and integration tests, you need to wait until something is true or that some state has been updated before moving forward.  The easiest way to deal with this is to let James.Testing wait until some predicate expression has come true.  
  
**Example:**
```csharp
Wait.Until(() => Test.Current.EventLogs.Count == 2);
```

There are cases in which a predicate may never end up being true.  By default, the Wait.Until() method has a timeout of 15 seconds.  You can also configure that when calling the API by passing in a timeout period.

**Timeout Expressed as TimeSpan**
```csharp
Wait.Until(() => Test.Current.EventLogs.Count == 2, TimeSpan.FromSeconds(5));
```
**Timeout Expressed as Integer - Seconds**
```csharp
Wait.Until(() => Test.Current.EventLogs.Count == 2, timeoutInSeconds);
```