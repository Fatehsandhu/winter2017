#### Important things to know before you begin coding

There are some very important things to know before you begin coding network tasks:

**Basic knowledge of HTTP is required.** You must know what a ‘request’ is, and what a ‘response’ is. 
These are the fundamental building blocks in an app that uses the network.

**Basic knowledge of web services is required.** You must know what a ‘web service’ is, and how to use one. 
In this course, you do not have to code/create a web service – you simply have to use one. 
You must also be able to understand and use [JSON](http://json.org).

**Network operations are asynchronous.** That means that we do not know _if_ or _when_ our request will be responded to.

**You must learn something about closures.** A closure in Swift is an executable code object. 
Network operations use closures.

**Simple tasks are easy to code, but complex tasks require more study.** Simple tasks include ‘get one’ and ‘get all’ from a resource. 
More complex tasks include HTTP POST, data modifications, authentication, and so on.

### Swift Closures

Think of a _closure_ as an _inline function_. (You have likely worked with similar constructs in the past, including JavaScript functions (and closures), and C# lambda expressions.)

Read the Apple doc on this topic:
https://developer.apple.com/library/prerelease/content/documentation/Swift/Conceptual/Swift_Programming_Language/Closures.html

Watch the WWDC video on Swift closures, from 25 min->33 min in this video:
[WWDC Intro to Swift Video](https://developer.apple.com/videos/play/wwdc2016/404/)<br>
<sub><sup><b>This requires Safari. Without Safari, you can open a 'network stream' in VLC player with this URL: http://devstreaming.apple.com/videos/wwdc/2016/404hskg1ijeev16mdej/404/hls_vod_mvp.m3u8</b></sup></sub>

A closure, like a regular function, has parameters and a return type:

```
// Instead of this..
func myFunc(parameters) -> returnValue {
  statements
}

// ...a closure looks like this
{ (parameters) -> returnValue in
    statements
}
```

But you can't call that closure.
Let's look at how you might call a closure:

```
// A closure that takes and Int and returns nothing (Void)
let closure = { (value: Int) -> Void in
    print("The value is \(value)")
}
closure(10)
```

Here we assigned the closure to a variable, then called it by using the variable with function syntax.<br>
The key point is that a _closure can be assigned to a variable_.

If follows from being able to assign closures to a variable, that _closures can be used as function parameters_.<br>
'Closures as function parameters' is an important concept in this week's lecture.

#### Void and ()

To indicate a closure takes no parameters, use `()`,
or to indicate it returns nothing, `Void` or `()` is used:
```swift
// These are the same
() -> ()
() -> Void

// TIP: A func in swift that doesn't specify a return value is implicity returning Void:
func noReturnValue() -> Void {
}
```

##### Syntax Tip
If the closure has _no parameters and returns nothing_, that is `() -> Void`, you can leave out the line<br>
`() -> Void in`.

If the closure _takes parameters and returns nothing_, like `(Int) -> Void`,<br>
you can leave out the `-> Void`.

This will look like:
```swift
let example = { 
    (x: Int) in // no '-> Void' needed
    print("\(x)")
}
```

#### Closures as function arguments

Closures are super-handy (and commonly used) to pass in to functions as arguments.<br>
The WWDC video linked above shows an interesting use of closures for iterating collections, such that your closure gets called for every item of the collection.<br>
Although this is a popular use of closures, in this class, we will instead focus on using closures as _completion callbacks_, so that the caller can be notified a function is complete.

```
func doLotsOfWork(completion: () -> Void) {
    // do lots of work, and then
    completion() // call the completion closure
}

// Calling the function and having it tell us when it is done
doLotsOfWork(completion: {
    print("Yay you are done!")
})
```

#### Asynchronous code and Closures go together like peas and carrots

When you call an asynchronous function, your code does not stop to wait for it complete.<br>
We will see our first asynchronous function this week when performing a networking call to get data.

Networking code is asynchronous because it can take long periods of time to complete and your program should not stop executing to wait.<br>
The _URLSessionDataTask.resume()_ function behaves like this.

```swift
// setup a networking task
let task = URLSession.sharedSession.dataTask(with: request, completionHandler: onComplete)

// tell the task to start
task.resume()
```

This last line, `task.resume()`, does not block the program execution. So how do we know when the data is downloaded and ready?<br>
Notice the `completionHandler` parameter. That is a closure of the form `(Data?, URLResponse?, Error?) -> Void`.

The `onComplete` variable used above would look like:
```
let onComplete = { (data: Data?, response: URLResponse?, error: Error?) -> Void
    if let error = error {
        print(error)
    } else {
        // do something with data
    }
}
```

Because of Swift's type inference, if a closure is defined in-place as a function parameter, you can leave out the types.
This looks like:
```swift
let task = URLSession.sharedSession.dataTask(with: request, completionHandler: {
    (data, response, error) in // for instance, Swift knows that 'data' is of type 'Data?'
    // check for an error or use the data
})
```


### Getting started, hands-on

This week, you will use the Project_Templates/WebServiceModel template to create a simple app that uses a web service.

Download the template from the [GitHub code example repository](../Project_Templates/WebServiceModel/). Then, perform the project-rename task.

We will use a public web service. It is here:

https://ict.senecacollege.ca/api

By itself, a web browser is not a good tool to use to inspect a web service. Instead, use this web app:

[http://jsonformatter.curiousconcept.com](http://jsonformatter.curiousconcept.com)

Enter a resource URI in its “JSON Data Url” field, and click its “Process” button.
The response to each request will be displayed in its own grey-bordered box. Try it with these URIs:

https://ict.senecacollege.ca/api/programs

https://ict.senecacollege.ca/api/courses

https://itunes.apple.com/search?term=big+bang+theory&entity=tvEpisode&limit=10&sort=recent

ALternatively, on the command-line you can type:<br>
`curl https://ict.senecacollege.ca/api/programs`
<br>And you can see the JSON response.

#### Composing a network request, and handling the response

In iOS (and OS X), we use a ‘task’ object to compose a network request, and handle the response.

The ‘task’ object is an instance of 
[NSURLSessionDataTask](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSessionDataTask_class/Reference/Reference.html).
It uses a ‘session’ object, and a ‘request’ object.

The ‘session’ object is an instance of 
[NSURLSession](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSession_class/Introduction/Introduction.html),
which represents a logical session between your app and a web service. 
A ‘session’ object must be configured, using a ‘session configuration’ object (which is an instance of 
[NSURLSessionConfiguration](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSessionConfiguration_class/Reference/Reference.html)).

The ‘request’ object is an instance of [NSMutableURLRequest](https://developer.apple.com/library/ios/documentation/cocoa/reference/foundation/Classes/NSMutableURLRequest_Class/Reference/Reference.html),
which needs a ‘URL’ object, and can be configured with request headers if necessary.

In summary, a ‘task’ object relies on the presence of a number of other initialized and configured objects.

#### Configuring and executing the ‘task’ object

As discussed in the first part of the lecture, when a ‘task’ object is created, it does not immediately execute (it is in a suspended state). You must execute it with `resume()`.
The ‘task’ object executes in the background, so it does not impair the responsiveness of your app’s user interface.

The ‘task’ requires a _block of code_ that will run when the task completes execution, which is a _closure_ .
The _closure_ is defined on the ‘task’ object’s _completionHandler_ parameter. It exposes these values:

*   The data returned in the response body
*   Metadata about the response
*   If necessary, error information

They are all _optional types_, so unwrapping is necessary to use them safely.

The data type of the data returned in the response body is `Data`. Therefore, we must transform it into the format we expect. 
In this course, we plan to work with web services that use JSON, 
so we will [transform](https://developer.apple.com/library/mac/documentation/Foundation/Reference/NSJSONSerialization_Class/Reference/Reference.html) 
the `Data` object to JSON and then to an array or dictionary, as appropriate for the request.

> The types `NSData` and `Data` are equivalent in Swift 3 (the version we use).  
`NSData` is the term most often used in docs and examples to describe the type,  
and is still the only one documented, see https://developer.apple.com/reference/foundation/nsdata  
>  
> On the other hand, the doc for Swift `Data` is empty: https://developer.apple.com/reference/foundation/data

#### Code example

The [../Project_Templates/WebServiceModel](../Project_Templates/WebServiceModel) template has a functional code example. It loads a list of programs in a table view, and each row can be tapped to output the details for a program in the console.

#### Learning resources

[URL Loading System Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html) (a summary will be added below)

#### A deeper look at the networking classes

The following is a summary of the important parts of the URL Loading System Programming Guide, as well as the class reference documents for NSURLSession and others.

> Content copied directly/verbatim from the Apple documentation is shown in quoted block style

> The URL loading system is a set of classes and protocols that allow your app to access content referenced by a URL.

> The most commonly used classes in the URL loading system allow your app to retrieve the content of a URL from the source. In iOS 7 and later or OS X v10.9 and later, NSURLSession is the preferred API for new code that performs URL requests. Old code examples and search engine results will use other classes (NSURLConnection, AFNetworking). While useful and instructive, you should not rely on old code examples.

> The URL loading classes use two helper classes that provide additional metadata—one for the request itself (NSURLRequest) and one for the server’s response (NSURLResponse).

> The NSURLSession class and related classes provide an API for downloading content via HTTP. Like most networking APIs, the NSURLSession API is highly asynchronous. If you use the default, system-provided delegate, you must provide a completion handler block that returns data to your app when a transfer finishes successfully or with an error. 

> The NSURLSession API supports three types of sessions. We will use the “default session” type.

> Within a session, the NSURLSession class supports three types of tasks: data tasks, download tasks, and upload tasks. We will mostly use “data tasks”.

> The NSURLSession API provides a wide range of configuration options. Most settings are contained in a separate configuration object (which is an instance of NSURLSessionConfiguration).

For many getting-started examples, when you instantiate an NSURLSession object, you specify the following: 

1.  A configuration object that governs the behavior of that session and the tasks within it
2.  Optionally, a delegate object; however, we will use _nil_. If you do not provide a delegate, the NSURLSession object uses a system-provided delegate.
3.  The name of an _operation queue_ that performs the task specified in the _completion handler block_. We will use _NSOperationQueue.mainQueue_.

> Your app can provide the request body content for an HTTP POST request in three ways. 
We will use an NSData object.

> To upload body content with an NSData object, your app calls … the 
uploadTask(with: URLRequest, from: Data, completionHandler:) method
to create an upload task, and provides request body data through the from:Data parameter.  

> The session object computes the Content-Length header based on the size of the data object.

> Your app must provide any additional header information that the server might require—content type, for example—as part of the URL request object.


**Life Cycle of a URL Session with System-Provided Delegates**

Here is the basic sequence of method calls that your app must make and completion handler calls that your app receives when using NSURLSession with the system-provided delegate:

1. Create a session configuration.

2. Create a session, specifying a configuration object, a nil delegate, and an operation queue.

3. Create task objects within a session that each represent a resource request. Write a completion handler block.

> Each task starts out in a suspended state. After your app calls resume on the task, it begins downloading the specified resource.

> When a task completes, the NSURLSession object calls the task’s completion handler.

> When your app no longer needs a session, invalidate it by calling _finishTasksAndInvalidate_ (to allow outstanding tasks to finish before invalidating the object).

> Note: NSURLSession does not report server errors through the error parameter. The only errors your app receives through the error parameter are client-side errors, such as being unable to resolve the hostname or connect to the host. 
 
> Server-side errors are reported through the HTTP status code in the NSHTTPURLResponse object. 

