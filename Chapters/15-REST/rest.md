## REST
@cha:rest

Seaside is not built around REST services by default, to increase programmer productivity and make to development much more fun. In some cases, it might be necessary to provide a REST API to increase the usability and interoperability of a web application though. Luckily Seaside provides a _Seaside REST_ package to fill the gap and to allow one to mix both approaches.

In this chapter, we show how to integrate web applications with Seaside REST services. We start with a short presentation of REST. Then we define a simple REST service for the todo application we implemented in Chapter . We finish this chapter by inspecting how HTTP requests and responses work. We want to thank Olivier Auverlot for providing us with an initial draft of this chapter in French.


### REST in a Nutshell

REST (Representational State Transfer) refers to an architectural model for the design of web services. It was defined by Roy Fielding in his dissertation on [Architectural Styles and the Design of Network-based Software Architectures](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm). The REST architecture is based on the following simple ideas:

- REST uses URIs to refer to and to access resources.
- REST is built on top of the stateless HTTP 1.1 protocol. 
- REST uses HTTP commands to define operations.


This last point is essential in REST architecture. HTTP commands have precise semantics:

- GET lists or retrieves a resource at a given URI. 
- PUT replaces or updates a resource at a given URI. 
- POST creates a resources at a given URI. 
- DELETE removes the resources at a given URI. 


Seaside takes a different approach by default: Seaside generates URIs automatically, Seaside keeps state on the server, and Seaside does not interact well with HTTP commands. While the approach of Seaside simplifies a lot of things in web development, sometimes it is necessary to play with the rules. REST is used by a large number of web products and adhering to the REST standard might increase the usability of an application.

REST applications with Seaside can take two shapes: The first approach creates or extends the interoperability of an existing application by adding a REST API. Web browsers and other client applications can (programmatically) access the functionality and data of an application server, see Figure *@fig:first-architecture@*.


![First architecture: adding REST to an existing application. % width=70&anchor=fig:first-architecture](figures/1st-architecture.png)

A second approach consists of using REST as the back-end of an application and make it a fundamental element of its architecture. All objects are exposed via REST services to potential clients as well as to the other parts of the application such as its Seaside user-interface, see Figure *@fig:second-architecture@*.


![Second architecture: REST centric core. % width=70&anchor=fig:second-architecture](figures/2nd-architecture.png)

This second approach offers a low coupling and eases deployment. Load-balacing and fail-over mechanisms can easily be put in place and the application can be distributed over multiple machines.

With Seaside and its Rest package you can implement both architectures. In this chapter we are going to look at the first example only, that is we will extend an existing application with a REST API.


### Getting Started with REST


To get started load the package `Seaside-Rest-Core`, and if you are on Pharo `Seaside-Pharo-Rest-Core`. All packages are available from the `Seaside30Addons` repository and you can load then easily with the following Gofer script:

!!todo check that and update to github and 

```
Gofer new
  squeaksource: 'Seaside30Addons';
  package: 'Seaside-REST-Core';
  package: 'Seaside-Pharo-REST-Core';
  package: 'Seaside-Tests-REST-Core';
  load.
```


Recent Seaside images already contain the REST packages preloaded. 

#### Defining a Handler


We are going to extend the todo application from Chapter *@cha:todoApp@* with a REST API. We will first build a service that returns a textual list of todo items.

Our REST handler, named `ToDoHandler`, should be declared by defining a Seaside class which inherits from `WARestfulHandler`. This way we indicate to Seaside that `ToDoHandler` is a REST handler. The todo items will be accessed through the same model as the existing todo application: `ToDoList default`. This means we do not need to specify additional state in our handler class.

```
WARestfulHandler << #ToDoHandler
   package: 'ToDo-REST'
```


!!note With Seaside-REST, we do not subclass from `WAComponent` that is reserved to the generation of stateful graphical components, but you should subclass from WARestfulHandler.

Last we need to initialize our hander by defining a class-side initialization method. We register the handler at the entry point `todo-api` so that it is reachable at [http://localhost:8080/todo-api](http://localhost:8080/todo-api). Don't forget to call the method to make sure the handler is properly registered.

```
ToDoHandler class >> initialize
   WAAdmin register: self at: 'todo-api'
```


#### Defining a System


The idea behind Seaside-REST is that each HTTP request triggers a method of the appropriate service implementation. All service methods are annotated with specific method annotations or pragmas. 

It is possible to define a method that should be executed when the handler receives a GET request by adding the annotation `<get>` to the method. As we will see in , a wide range of other annotations are supported to match other request types, content types, and the elements of the path and query arguments.

To implement our todo service, we merely need to add the following method to `ToDoHandler` that returns the current todo items as a string:

```
ToDoHandler >> list
   <get>
  	
   ^ String streamContents: [ :stream |
      ToDoList default items do: [ :each |
         stream nextPutAll: each title; crlf ] ]
```


The important thing here is the method annotation `<get>`, the name of the method itself does not matter. The annotation declares that the method is associated with any GET request the service receives. Later on we will see how to define handlers for other types of requests.

In a web browser enter the URL [http://localhost:8080/todo-api](http://localhost:8080/todo-api). You should get a file containing the list of existing todo items of your applications. If the file is empty verify that you have some todos on your application by trying it at [http://localhost:8080/todo](http://localhost:8080/todo). In case of problems, verify that the server is working using the Seaside Control Panel. If everything works well you should obtain a page with the list of todo items. To verify that our service works as expected we can also use _cURL_ or any other HTTP client to inspect the response:

```
$ curl -i curl -i http://localhost:8080/todo-api
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 71
Date: Sun, 20 Nov 2011 17:04:52 GMT
Server: Zinc HTTP Components 1.0

Finish todo app chapter
Annotate first chapter
Discuss cover design
```



By default Seaside tries to convert whatever the method returns into a response. In our initial example this was enough, but in many cases we want more control over how the response is built. We gain full control by asking the _request context_ to respond with a custom response. The following re-implementation of the `list` method has the same behavior as the previous one, but creates the response manually.

```
ToDoHandler >> list
   <get>

   self requestContext respond: [ :response |
      ToDoList default items do: [ :each |
         response contentType: 'text/plain'.
         response
            nextPutAll: each title;
            nextPutAll: String crlf ] ]
```


### Matching Requests to Responses


In the initial example we have seen how to define a service that catches all GET requests to the handler. In the following sections we will look at defining more complicated services using more elaborate patterns. In  we are going to look at matching other request types, such as POST and PUT. In  we are going to see how to serve different content types depending on the requested data. In  we will see how to match path elements and in  how to extract query parameters.


#### HTTP Method

Every service method must have a pragma that indicates the HTTP method on which it should be invoked.

If we would like to add a service to create a todo item with a POST request, we could add the following method:

```
ToDoHandler >> create
   <post>
	
   ToDoList default items
      add: (ToDoItem new
         title: self requestContext request rawBody;
         yourself).
   ^ 'OK'
```


We use the message `rawBody` to access the body of the request. The code creates a new todo item and sets its title. It then replies with a simple `OK` message.

To give our new service a try we could use cURL. With the `-d` option we define the data to be posted to the service:

```
$ curl -d "Give REST a try" http://localhost:8080/todo-api
OK
```


If we list the todo items as implemented in the previous section we should see the newly created entry:

```
$ curl http://localhost:8080/todo-api
Finish todo app chapter
Annotate first chapter
Discuss cover design
Give REST a try
```


Similarly Seaside supports the following request methods:


| Request Method | Method Annotation | Description |
| --- | --- | --- |
| GET | `<get>` | lists or retrieves a resource |
| PUT | `<put>` | replaces or updates a resource |
| POST | `<post>` | creates a resource |
| DELETE | `<delete>` | removes a resource |
| MOVE | `<move>` | moves a resource |
| COPY | `<copy>` | copies a resource |

#### Content Type


Using HTTP and Seaside-REST, we can also specify the format of the data that is requested or sent. To do that we use the `Accept` header of the HTTP request. Depending on it, the REST web service will adapt itself and provide the corresponding data type. 
 
We will take the previous example and we will modify it so that it serves the list of todo items not only as text, but also as JSON or XML. To do so define two new methods named `listJson` and `listXml`. Both methods will be a GET request, but additionally we annotate them with the mime type they produce using `<produces: 'mime-type'>`. This annotation specifies the type of the data returned by the method. A structured format like XML and JSON is friendly to other applications that would like to read the output.

```
ToDoHandler >> listJson
   <get>
   <produces: 'text/json'>
   
   ^ (Array streamContents: [ :stream |
      ToDoList default items do: [ :each |
         stream nextPut: (Dictionary new
            at: 'title' put: each title;
            at: 'done' put: each done;
            yourself) ] ])
      asJavascript
```

  
```
ToDoHandler >> listXml
   <get>
   <produces: 'text/xml'>

   ^ WAXmlCanvas builder 
      documentClass: WAXmlDocument;
      render: [ :xml |
         xml tag: 'items' with: [ 
            ToDoList default items do: [ :each |
               xml tag: 'item' with: [
                  xml tag: 'title' with: each title.
                  xml tag: 'due' with: each due ] ] ] ]
```


While in the examples above we (mis)use the JSON and XML builders that come with our Seaside image. You might want to use any other framework or technique to build your output strings.

By specifying the accept-header we can verify that our implementation serves the expected implementations:

```$ curl -H "Accept: text/json" http://localhost:8080/todo-api=true
[{"title": "Finish todo app chapter", "done": false},
 {"title": "Annotate first chapter", "done": true},
 {"title": "Discuss cover design", "done": false},
 {"title": "Give REST a try", "done": true}]
```



```
$ curl -H "Accept: text/xml" http://localhost:8080/todo-api
<items>
  <item>
    <title>Finish todo app chapter</title>
    <done>false</done>
  </item>
  <item>
    ... 
```


If the accept-header is missing or unknown, our old textual implementation is called. This illustrates that several methods can get a get annotation and that one is selected and executed depending on the information available in the request. We explain this point later. 

Similarly the client can specify the MIME type of data passed to the server using the content-type header. Such behavior only makes sense with PUT and POST requests and is specified using the `<consumes: 'mime-type'>` annotation. The following example states that the data posted to the server is encoded as JSON.

```
ToDoHandler >> createJson
   <post>
   <consumes: '*/json'>

   | json |
   json := JSJsonParser parse: self requestContext request rawBody.
   ToDoList default items
      add: (ToDoItem new
         title: (json at: 'title');
         done: (json at: 'done' ifAbsent: [ false ]);
         yourself).
   ^ 'OK'
```


We can test the implementation with the following cURL query:

```
$ curl -H "Content-Type: text/json" \
   -d '{"title": "Check out latest Seaside"}' \
   http://localhost:8080/todo-api
OK
```


#### Request Path


URIs are a powerful mechanism to specify hierarchical information. They allow one to specify and access to specific resources. Seaside-Rest offers a number of methods to support the manipulation of URIs. Some predefined methods are invoked by Seaside when you define them in your service. 

The method `list` we implemented in  is executed when the URI does not contain any access path beside the one of the application.

```
ToDoHandler >> list
   <get>
   	
   ^ String streamContents: [ :stream |
      ToDoList default items do: [ :each |
         stream nextPutAll: each title; crlf ] ]
```


If we define services with methods that expect multiple arguments, the arguments get mapped to the unconsumed path elements. In the example below we use the first path element to identify a todo item by title, and then perform an action on it using the second path element:

```
ToDoHandler >> command: aTitleString action: anActionString
   <get>

   | item |
   item := ToDoList default items
      detect: [ :each | each title = aTitleString ]
      ifNone: [ ^ 'unknown todo item' ].
   anActionString = 'isDone' ifTrue: [
      ^ item done
         ifTrue: [ 'done' ]
         ifFalse: [ 'todo' ] ].
   ...
   ^ 'invalid command'
```


Now we can query the model like in the following examples:

```
$ curl http://localhost:8080/todo-api/Invalid/isDone
unknown todo item
$ curl http://localhost:8080/todo-api/Discuss+cover+design/isDone
done
$ curl http://localhost:8080/todo-api/Annotate+first+chapter/isDone
todo
```


#### Query Parameters

So far we used the request type (), the content type () and the request path () to dispatch requests to methods. The last method which is also the most powerful one, is to dispatch on specific path elements and query parameters.

Using the annotation `<path:>` we can define flexible masks to extract elements of an URI. The method containing method is triggered when the path matches verbatim. Variable parts in the path definition are enclosed in curly braces `{aString}` and will be assigned to method arguments. Variable repeated parts in the path definition are enclosed in stars `*anArray*` and will be assigned as an array to method arguments.

The following example implements a search listing for our todo application. Note that the code is almost exactly the same as the one we had in our initial example, except that it filters for the query string:

```
ToDoHandler >> searchFor: aString
   <get>
   <path: '/search?query={aString}'>

   ^ String streamContents: [ :stream |
      ToDoList default items do: [ :each |
         (each title includesSubString: aString)
            ifTrue: [ stream nextPutAll: each title; crlf ] ] ]
```


The method is executed when the client sends a GET request which starts with the path `/search` and contains the query parameter `query`. The expression `{aString}` makes sure that the method argument `aString` is bound to that request argument.

Give it a try on the console. With the right query string only the todo items with the respective substring are printed:

```
$ curl http://localhost:8080/todo-api/search?query=REST
Give REST a try
```


#### Conflict Resolution


Sometimes there are several methods which Seaside-REST could choose for a request, here's how it finds the "best" one:

1. Exact path matches like `/index.html` take precedence over partial `/index.{var}` or `{var}.html` or wildcard ones `{var}`.
1. Partial path matches like `/index.{var}` or `{var}.html` take precedence over wildcard ones `{var}`.
1. Partial single element matches `{var}` take precedence over multi element matches `*var*`.
1. Exact mime type matches like `text/xml` take precedence over partial `*/xml` or `xml/*`, wildcard `*/*` and missing ones.
1. Partial mime type matches like `*/xml` or `xml/*` take precedence over wildcard ones `*/*` or missing ones.
1. If the user agent supplies quality values for the Accept header, then that is taken into account as well.



### Handler and Filter


So far, our REST service did not interact much with the existing Seaside todo application (other than through the shared model). Often it is however desired to have both -- the application and the REST services -- served from the same URL.

To achieve this we have to subclass `WARestfulFilter` instead of `WARestfulHandler`. The `WARestfulFilter` simply wraps a Seaside application. That is, it handles REST requests exactly as the `WARestfulHandler`, but it can also delegate to the wrapped Seaside application.

To update our existing service we rename `ToDoHandler` to `ToDoFilter` and change its superclass to `WARestfulFilter`. Now the class definition should look like:

```
WARestfulFilter << #ToDoFilter
   package: 'ToDo-REST'
```


A filter cannot be registered as a an independent entry point anymore, thus we should remove it from the dispatcher to avoid errors:

```
WAAdmin unregister: 'todo-api'
```


Instead we attach the filter the todo application itself. On the class-side of `ToDoListView` we adapt the `initialize` method to:

```
ToDoListView class >> initialize
   (WAAdmin register: self asApplicationAt: 'todo')
      addFilter: ToDoFilter new
```


After evaluating the initialization code, the `ToDoFilter` is now executed whenever somebody accesses our application. The process is visualized in *@fig:request-handling@*. Whenever a request hits the filter (1), it processes the annotations (2). Eventually, if none of the annotated methods matched, it delegates to the wrapped application by invoking the method noRouteFound: (3). 

% +request-handling|width=50%+
![Request handling of `WARestfulFilter` and `WAApplication` % width=70&anchor=fig:request-handling](figures/REST3.png)

Unfortunately -- if you followed the creation of the REST API in the previous sections -- our `#list` service hides the application by consuming all requests to [http://localhost:8080/todo](http://localhost:8080/todo). We have two possibilities to fix the problem:

1. We remove the method `#list` so that `ToDoFilter` automatically calls `noRouteFound:` that eventually calls the application.
1. We add a new service that captures requests directed at our web application and explicitly dispatches them to the Seaside application. For example, the following code triggers the wrapped application whenever HTML is requested:



```
ToDoFilter >> app
    <get>
    <produces: 'text/html'>

    ^ self noRouteFound: self requestContext
```


This change leaves the existing API intact and lets users access our web application with their favorite web browser. This works, because browser request documents with the mime-type `text/html` by default. Of course, we can combine this technique with any other of the matching techniques we discussed in the previous chapters.

### Request and Response


Accessing and exploring HTTP requests emitted by a client is an important task during the development of a REST web service. The request gives access to information about the client (IP address, HTTP agent, ...).

To access the request we can add the expression `self requestContext request inspect` anywhere into Seaside code. This works inside a `WAComponent` as well as inside a `WARestfulHandler`.

When the method is executed, you get an inspector on the current request as shown in *@fig:headerhttp_linux.png@*.


![Inspecting a request. % width=70&anchor=fig:headerhttp_linux.png](figures/headerhttp_linux.png)

The following example uses the expression `headers at: 'content-type'` that returns the `Content-Type` present in the inspected HTTP client request. 

```
ToDoHandler >> list
    <get>

    self requestContext request inspect.
    ^ String streamContents: [ :stream |
        ...
```


In the case of the transmission of a form (corresponding to the application/x-www-form-urlencoded MIME type), we can access the submitted fields using the message `postFields`.

It is also possible to customize the HTTP response. A common task is to set the HTTP response code to indicate a result to the client.

For example we could implement a delete operation for our todo items as follows:

```
ToDoHandler >> delete: aString
    <delete>
   	
    | item |
    item := ToDoList default items
       detect: [ :each | each title = aString ]
       ifNone: [ nil ].
    self requestContext respond: [ :response |
       item isNil
         ifTrue: [ response status: WARequest statusNotFound ]
         ifFalse: [ 
            ToDoList default remove: item.
            response status: WARequest statusOk ] ]
```

		
The different status codes are implemented on the class side of WARequest. They are grouped into five main families with numbers between 100 and 500. The most common one is status code `WARequest statusOk` (200) and `WARequest statusNotFound` (404).

### Advices and Conclusion


This chapter shows that while Seaside provides a powerful way to build dynamic application using a stateful approach, it can also seamlessly integrate with existing stateless protocols. This chapter illustrated that an object-oriented model of an application in combination with Seaside is very powerful: You can develop flexible web interfaces as composable Seaside components, and you can easily enrich them with an API for interoperability with REST clients. Seaside provides you with the best of all worlds: the power of object design, the flexibility and elegance of Seaside components, and the integration of traditional HTTP architectures.

A piece of advice: 
- Do not use cookies with a REST service. Such service should respect the stateless philosophie of HTTP. Each request should be independent of others.
- During the development, organize your tagged methods following the HTTP commands: (GET, POST, PUT, DELETE, HEAD). You can use protocols to access them faster.
- A good service web should be able to produce different types of content depending on the capabilities of the clients. Being able to produce different formats such as plain text (text/plain), XML (text/xml), or JSON (text/json) increases the interoperability of your web services.


You should now have a better understanding of the possibilities offered by Seaside-REST and be ready to produce nice web services.
