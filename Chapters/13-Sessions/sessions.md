## Managing Sessions
@cha:session

When a user interacts with a Seaside application for the first time, a new _session_ object is automatically instantiated. This instance lasts as long as the user interacts with the application. Eventually, after the user has not interacted with the session for a while, it will time-out -- we say that the session _expires_. The session is internally used by Seaside to remember page-views and action callbacks. Most of the time developers don't need to worry about sessions.

In some cases the session can be a good place to keep information that should be available globally. The session is typically used to keep information about the current user or open database connections. For simple applications, you might consider keeping that information within your components. However, if big parts of your code need access to such objects it might be easier to use a custom session class instead.

Having your own session class can be also useful when you need to clean-up external resources upon session expiry, or when you need extra behavior that is performed for every request.

In this chapter you will learn how to access the current session, debug a session, define your own session to implement a simple login, recover from session expiration, and how to define bookmarkable urls.

### Accessing the Current Session


From within your components the current session is always available by sending `self session`. This can happen during the rendering phase or while processing the callbacks: you get the same object in either case. To demonstrate a way to access the current session, quickly add the following code to a rendering method in your application:

```
html anchor
    callback: [ self show: (WAInspector current on: self session) ];
    with: 'Inspect Session'
```



This displays a link that opens a Seaside inspector on the session. Click the link and explore the contents of the active session. To get an inspector within your image you can use the code `self session inspect`. In both cases you should be able to navigate through the object.

In rare cases, it might be necessary to access the current session from outside your component tree. Think twice before doing that though: it is considered to be extremely bad coding style to depend on the session from outside your component tree. Anyway, in some cases it might come in handy. In such a case, you can use the following expressions: 

```
WARequestContext value session
```


But again you should avoid accessing the session from outside of the component tree. 


#### Accessing the Session from the Debugger


In older versions of Seaside, session objects could not be inspected from the debugger as normal objects. If you tried to evaluate `self session` the debugger would answer `nil` instead of the expected session object. This is because sessions are only accessible from within your web application process, and the Smalltalk debugger lives somewhere else. In Seaside 3.0 this problem is fixed on most platforms.

If this doesn't work for you, then you need to use a little workaround to access the session from within the debugger. Put the following expression into your code to open an inspector from within the web application and halt the application by opening a debugger:

```
self session inspect; halt
```



### Customizing the Session for Login


We will now implement an extremely simple login facility to show how to use a custom session. We will enhance the `miniInn` application we developed in Chapter  and add a login facility.

When a user interacts with a Seaside application for the first time, an instance of the application's session class is created. The class `WASession` is the default session class, but this can be changed for each application, allowing you to store key information on this class.  Different parts of the system will then be able to take advantage of the information to offer different services to the user.

We will define our own session class and use it to store user login information. We will add login functionality to our existing component.
 The login functionality could also be supported by using a task and/or a specific login component. The principle is the same: you use the session to store some data that is accessible from everywhere within the current session. 

In our application we want to store whether the user is logged in. Therefore we create a subclass called `InnSession` of the class `WASession` and we will associate such a new session class to our hotel application. We add the instance variable `user` to the session to hold the identity of the user who is currently logged in.

```
WASession << #InnSession
   slots: { #user}; 
   package: 'SeasideBook'
```


We define some utility methods to query the user login information.

```
InnSession >> login: aString
    user := aString
```


```
InnSession >> logout
    user := nil
```


```
InnSession >> isLoggedIn
    ^ user isNil not
```


```
InnSession >> user
    ^ user
```


Now you need to associate the session we just created with your existing application; you can either use the configuration panel or register the new application setup programmatically.

**Configuration Panel.** To access the configuration panel of your application go to [http://localhost:8080/config/](http://localhost:8080/config/). In the list select your application \(probably called \`miniinn'\) and click on its associated _configure_ link. You should get to the configuration panel which lists several things such as: the library your application uses \(see *@ref:/book/web-20@*\); and its general configuration such as its root component \(see *@cha:deployment@*\).  

Click on the drop-down list by _Session Class_ -- if there is only text here, press the _override_ link first . Among the choices you should find the class `InnSession`. Select it and you should get the result shown in Figure *@fig:innSessionConfig@*. Now _Save_ your changes.


![The session of miniInn is now InnSession. %width=80&anchor=fig:innSessionConfig](figures/innSessionConfig.png )

**Configuring the application programmatically.** To change the associated session of an application, we can set the preference `#sessionClass` using the message `WASession>>preferencesAt:put:`. We can do that by redefining the _class_ `initialize` method of the application as follows. Since this method is invoked automatically only when the application is loaded, make sure that you evaluate it manually after changing it.

```
MiniInn class >> initialize
    | application |
    application := WAAdmin register: self asApplicationAt: 'miniInn'.
    application preferenceAt: #sessionClass put: InnSession
```


To access the current session use the message `WAComponent>>session`.  We define the methods `login` and `logout` in our component.

```
MiniInn >> login
    self session login: (self request: 'Enter your name:')
```


```
MiniInn >> logout
    self session logout
```


Then we define the method `renderLogin:` which, depending on the session state, offers the possibility to either login or logout.

```
MiniInn >> renderLogin: html
    self session isLoggedIn
        ifTrue: [
            html text: 'Logged in as: ' , self session user , ' '.
            html anchor
                callback: [ self logout ];
                with: 'Logout' ]
        ifFalse: [
            html anchor
                callback: [ self login ];
                with: 'Login']
```


We define a dummy method `renderSpecialPrice:` to demonstrate behavior only available for users that are logged in.

```
MiniInn >> renderSpecialPrice: html
    html text: 'Dear ' , self session user, ', you can benefit from our special prices!'
```

Then we redefine the method `renderContentOn:` to present the new functionality.

```
MiniInn>>renderContentOn: html
    self renderLogin: html.
    html heading: 'Starting date'.
    html render: calendar1.
    startDate isNil
        ifFalse: [ html text: 'Selected start: ' , startDate asString ].
    html heading: 'Ending date'.
    html render: calendar2.
    (startDate isNil not and: [ endDate isNil not ]) ifTrue: [
         html text: (endDate - startDate) days asString ,
             ' days from ', startDate asString, ' to ', endDate asString, ' ' ].
    self session isLoggedIn "<-- Added"
        ifTrue: [ self renderSpecialPrice: html ]
```


Figures *@fig:Session1@*, *@fig:Session2@* and *@fig:Session3@* illustrate the behavior we just implemented. The user may log in using the top-level link. Once logged in, extra information is available to the user.

![With Session.% width=60&anchor=fig:Session1](figures/Session1.png)

![ With Session: Enter your name. % width=60&anchor=fig:Session2](figures/Session2.png)

![With Session: Starting Date and Ending Date. % width=60&anchor=fig:Session3](figures/Session3.png)


### LifeCycle of a Session


It is important to understand the lifecycle of a session to know which hooks to customize. 
*@fig:lifetime@* depicts the lifetime of a session:
1. When the user accesses a Seaside application for the first time a new session instance is created and the root component is instantiated. Seaside sends the message `WAComponent>>initialRequest:`  to the active component tree, just before triggering the rendering of the components. Specializing the method `initialRequest:` enables developers to inspect the head fields of the first request to an application, and to parse and restore state if necessary.
1. All subsequent requests are processed the same way. First, Seaside gives the components the ability to process the callbacks that have been defined during the last rendering pass. These callbacks are usually triggered by clicking a link or submitting a form. Then Seaside calls `WAComponent>>updateUrl:`  of all visible components. This gives the developer the ability to modify the default URL automatically generated by Seaside. Then Seaside redirects the user to the new URL. This redirect is important, because it avoids processing the callbacks unnecessarily when the user hits the _Back_ button. Finally Seaside renders the component tree.
1. If the session is not used for an extended period of time, Seaside automatically expires it and calls the method  `WASession>>unregistered`. If the user bookmarked the application, or comes back to the expired session for another reason, a new session is spawned and the lifecycle of the session starts from the beginning.


![Life cycle of a session. % width=60&anchor=fig:lifetime](figures/lifetimeofsession.png)


### Catching the Session Expiry Notification


Sessions last a certain period of time if there are no requests coming in, after which they expire. The default is 600 seconds or 10 minutes. You can change this value to any other number using the configuration interface, or programmatically using the following expression:

```
"Set the session timeout to 1200 seconds (20 minutes)"
anApplication cache expiryPolicy configuration 
    at: #cacheTimeout put: 1200
```


Depending on the type of your application you might want to increase this number. In industrial settings 10 minutes \(600 seconds\) has shown to be quite practical: it is a good compromise between user convenience and memory usage.

When a session expires Seaside sends the message `WASession>>unregistered` to `WASession`. You can override this method to clean up your session, for example if you have open files or database connections. In our small example this is not really necessary, but to illustrate the functionally we will now logout the user automatically when the session expires:

```
InnSession >> unregistered
    super unregistered.
    user := nil
```


Note that at the time the message `unregistered` is sent, there is no way to inform the user in the web browser about the session expiry. The message `unregistered` is called asynchronously by the Seaside server thread and there is no open connection that you could use to send something to the client -- in fact the user may have already closed the browser window. We will see in the next section how to recover if the user does try to return to the session.

### Manually Expiring Sessions

In some cases developers might want to expire a session manually. This is useful for example after a user has logged out, as it frees all the memory that was allocated during the session. More important it makes
it impossible to use the _Back_ button to get into the previously authenticated user-account and do something malicious.

A session can be marked for expiry by sending the message `WASession>>expire`  to a `WASession`.  Note that calling `expire` will not cause the session to disappear immediately, it is just marked as expired  and not accessible from the web anymore. At a later point in time Seaside will call `unregistered` and the garbage collector eventually frees the occupied memory.

Let us apply it to our hotel application: we change our MiniInn application to automatically expire the session when the user logs out.

```
InnSession >> logout
    user := nil.
    self expire
```


Note that expiring a session without redirecting the user to a different location will automatically start a new session within the same application. Here we change that behavior to make it point to the Seaside web site as follows.

```
InnSession >> logout
    user := nil.
    self expire.
    self redirectTo: 'http://www.seaside.st'
```


If the user tries to get back to the application, he is automatically redirected to a new session.

### Summary


Sessions are Seaside's central mechanism for remembering user specific interaction state. Sessions are identified using the `_s` parameter in the URL. As an application developer there is normally no need to access or change the session, because it is used internally by Seaside to manage the callbacks and to store the component tree. In certain cases it might be useful to change the behavior of the default implementation or to make information accessible from anywhere in the application.

Pay attention that if components depend on the presence of a specific session class, you introduce strong coupling between the component and the session. Such sessions act as global variables and should not be overused.