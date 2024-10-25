## Calling Components
@cha:components

Seaside applications are based on the definition and composition of components. Each component is responsible for its rendering, its state and its own control flow. Seaside lets us freely compose and _reuse_ such components to create advanced and dynamic applications. You have already built several components. In this chapter, we will show how to reuse these components by ''calling'' them in a modal way. Embedding components in other components will be discussed in the following chapter.

### Displaying a Component Modally


Seaside components have the ability to specify that another component should be rendered (usually temporarily) in their place. This mechanism is triggered by the message `call:`. During callback processing, a component may send the message `call:` with another component as an argument. The component passed as an argument in this way can be referred to as the _delegate_. The `call:` method has two effects:

1. In subsequent rendering cycles, the delegate will be displayed in place of the original component. This continues until the delegate sends the message `WAComponent>>answer` to itself.
1. The current execution state of the calling method is suspended and does not return a value yet. Instead, Seaside renders the web page in the browser (showing the delegate in place of the original component).


The delegate may be a complex component with its own control flow and state. If the delegate component later sends the message `answer`, then execution of the (currently suspended) calling method is resumed at the site of the `call:`. We will explain this mechanism in detail after an example. 

!!note Important From the point of view of a component, it calls another component and that component will \(eventually\) answer.

### Example of Call/Answer


To illustrate that mechanism, let's use the `ContactListView` and `ContactView` components developed in earlier chapters. Our goal is simple: in a `ContactListView` component, we will display a link to edit the contacts (as shown in *@ref:list-contact@*), and when the user selects that link, display the `ContactView` on that `Contact`. We accomplish this using the `call:` message.

![New version of the `ContactListView`. % width=80&anchor=ref:list-contact](figures/listContact.png)



The `editContact:` method is passed a contact as an argument. It creates a `ContactView` component for the contact and calls this new component by sending it the message `call:`.

```
ContactListView >> editContact: aContact
    self call: (ContactView new 
        contact: aContact;
        yourself)
```


Next, we change the method `ContactListView>>renderContact:on:` to invoke the method we just defined when the edit link is selected, as below:

```
ContactListView >> renderContact: aContact on: html
    html text: aContact name , ' ' , aContact emailAddress.
    html text: ' ('.
    html anchor   " <-- added "
        callback: [ self editContact: aContact ];
        with: 'edit'.
    html space.
    html anchor
        callback: [ self removeContact: aContact ];
         with: 'remove'.
    html text: ')'
```


In the previous chapters, the `save` method of the `ContactView` component just displayed the contact values using a dialog. Now, using the message `answer`, we are able to return control from the newly created `ContactView` component to the `ContactListView` which created it and called it. Modify the `ContactView` so that when the user presses _Save_ it returns to the caller \(the `ContactListView`\):

```
ContactView >> save
    self answer
```


Have a look at the way the method `editContact:` creates a new instance of `ContactView` and then passes this instance as an argument to the `call:` message. When you call a component, you're passing control to that component. When that component is done \(in this case the user pressed the _Save_ button\), it will send the message `answer` to return control to the caller.

Interact with this application now and follow the link. Fill out the resulting form and press the _Save_ button. Notice that you're back to the `ContactListView` component. So, you `call:` another component and when it is done it should `answer`, returning control of the display to the caller. 

!!note Important You can think of the call/answer pair as the Seaside component equivalent of raising and closing a modal dialog respectively.


### Call/Answer Explained



![Call and Answer at Work. % width=80&anchor=ref:call](figures/flow_basics)

Figure *@ref:call@* illustrates the call/answer principle. The application is showing our `ContactListView` component. 

When the user presses _edit_ next to a contact name, the `ContactListView` component executes its callbacks until it reaches the line `self call: ...`, where it sends the message `call:` and passes it the `ContactView` component. This causes `ContactView` to take control of the browser region occupied by `ContactListView`. Note that the other component `A`, can continue to be active; this is an example of having multiple, simultaneous control flows in an application. 

When the component `ContactView` reaches the line `self answer`, it sends the message `answer`, which has the effect of closing the `ContactView` component, giving back control to the `ContactListView` component, and possibly returning a value, as you will see in the next section. The returned value can be a complex object such as credit card information or a complete Contact object, or it can be as simple as a primitive object such as an integer or a string. With Seaside you handle objects directly and there is no need to translate or marshall them to pass them around different components. 

After the `answer` message send, the execution of the component `ContactListView` continues just after the call that opened the component `ContactView`. This is marked as `"continue here"` in the diagram.

### Component Sequencing


As we just showed, calling is a _modal_ interaction, that is, the method `call:` doesn't return until the component it called answers. That allows us to sequence component display.

```
ContactListView >> editContact: aContact
    | view |
    view := ContactView new.
    view contact: aContact.
    self call: view.
    self inform: 'Thanks for editing  ' , aContact name
```



Let's suppose that you have redefined the method `editContact:` as shown above. The method calls the view component and then, after the view answers, it displays a message. Here's something to wrap your brain around. What if the user fills in the form, presses the _Save_ button, then presses _Back_ and changes the values in the form and saves again? After the first save, the above method calls `WAComponent>>inform:` but when the user presses _Back_ your method backs up into the `call:` of `ContactView`. 

 What Seaside does is the following: It snapshots the state of execution of your method so that it can jump back in response to the _Back_ button. We'll go into much more detail about this later in . For now just try it and confirm that things work as you'd expect.



### Answer to the Caller
There is a version of the `answer` message which takes an argument. This version, named `answer:` returns a value to the caller. One common use of this is to return a boolean to indicate if the user canceled or completed the operation. Since we don't have a cancel button in our `ContactView`, let's add one and answer appropriately, see *@ref:cancel@*.

But before doing that, we will refactor the `renderContentOn:` method. It's too long and overdue for refactoring. Using the refactoring capabilities of your favorite browser, extract methods so that it looks like this.

```
ContactView >> renderContentOn: html
    html form: [
        self renderNameOn: html.
        self renderEmailOn: html.
        self renderGenderOn: html.
        self renderSendUpdatesOn: html.
        self renderDateOn: html.
        self renderSaveOn: html ]
```


Now edit your new `renderSaveOn:` method to add a cancel button:

```
ContactView >> renderSaveOn: html
        html submitButton on: #cancel of: self. " <-- added "
        html submitButton on: #save of: self
```



![Contact edition with a cancel button. % width=80&anchor=ref:cancel](figures/withCancel.png)

Redefine the following methods to cancel and save the editing.

```
ContactView >> save
    self answer: true
```


```
ContactView >> cancel
    self answer: false
```


Now we can change the method `ContactViewList>>editContact:` to use the returned value to avoid showing the final `inform:` dialog. 

```
ContactListView >> editContact: aContact
    | view |
    view := ContactView new.
    view contact: aContact.
    (self call: view)
        ifFalse: [ ^ nil ].
    self inform: 'Thanks for editing ' , aContact name
```


If you try using the application as it currently stands, you may get a nasty surprise: if the user changes the name in the form and then presses _Cancel_, the underlying object will still be updated! So, rather than passing it the object we want to edit, we should instead pass it a copy of the contact to be edited and then, depending on the result passed by the `answer:`, decide whether to substitute the corresponding contact in the contact list.

```
ContactListView>>editContact: aContact
    | view copy |
    view := ContactView new.
    copy := aContact copy.
    view contact:  copy.
    (self call: view) 
        ifTrue: [ Contact removeContact: aContact; addContact: copy ]
```

	
!!note When you edit a user now, you'll notice that the user ends up moved to the end of the list of the users; this is the expected behaviour.


### Don’t call while rendering


One of the most common mistakes for first-time Seaside developers is to send the message `call:` a component from another component's rendering method, `renderContentOn:`. The rendering method's purpose is _rendering_. Its only job is to display the current state of the component. Callbacks are responsible for changing state, calling other components, etc. If you want to render one component inside another one read .

!!note Important Don't `call:` a component from `renderContentOn:`, only call components from callbacks or from `WATask` subclasses.


### A Look at Built-In Dialogs


Now it's time to look at the source code for the  `WAComponent>>inform:` method from the `WAComponent` class. Do not type this code, it is already part of Seaside. The definition of the method `inform:` of the class `WAComponent` is the following one.

```
WAComponent >> inform: aString
    "Display a dialog with aString to the user until he clicks the ok button."
    
    ^ self wait: [ :cc | self inform: aString onAnswer: cc ]
```


The method `inform:` sends the message `wait:` to raise a newly created `WAFormDialog` component, exactly the same way as `call:` does.

How do you find related methods? Looking through  `WAComponent` reveals:

- `WAComponent>>inform:` displays a dialog with a message to the user until he clicks the button.
- `WAComponent>>confirm:` displays a message and _Yes_ and _No_ buttons. Returns true if user selected _Yes_, false otherwise.
- `WAComponent>>request:`, `WAComponent>>request:default:`, `WAComponent>>request:label:`, and   `WAComponent>>request:label:default:` display a message, an optional label and an input box. The string entered into the input box is returned. If the `default:` argument is specified it is used for the initial contents of the input box.
- `WAComponent>>chooseFrom:`, `WAComponent>>chooseFrom:caption:`, `WAComponent>>chooseFrom:default:`, and `WAComponent>>chooseFrom:default:caption:` display a drop-down list with different choices to let the user choose from. A default selection and a title can be given. The methods answer the selected item.


### Handling The Back Button


Web browsers allow you to navigate your browsing history using the back button. The problem is that when you press the back button, the application interface and the underlying model can be out of sync. When you press the back button, only the browser is involved and not the server and the server has no way to know that you changed. Therefore your UI can be out of sync from its domain. Seaside offers you a way to control the back button effect.

There are two kinds of synchronization problems: UI state and model state. Seaside offers a good solution for UI state synchronization.

**Experiment with the problem.** In this section, we show the back button problem and show how Seaside makes it easy to handle. Perform the following experiment.

1. Browse the `WebCounter` application that we developed in the first chapter of this book.
1. Click on the _++_ link to increase the value of the counter until the counter shows a value of 5.
1. Press the back button two times: you should see 3 now.
1. Click on the _++_ link.


Your web browser does not show you 4 as you would expect, but instead displays 6. This is because the `WebCounter` component was not aware that you had pressed the back button. This situation can also arise if you open two windows that interact with the same application.

**Solving the Problem.** Seaside offers an elegant way to fix this problem. Define the method  `WAComponent>>states` so that it returns an array that contains the objects that you want to keep in sync. In our `WebCounter` example we want to keep the count instance variable synchronized so we write the method as follows.

```
WebCounter>>states
    ^ Array with: count
```


This is not really what we want because the Seaside backtracking support is mostly intended for UI state and not model state. We want to backtrack the counter component, not the integer instance variable.

```
WebCounter>>states
    ^ Array with: self
```

Redo our back button experiment and you will see that after pressing the back button two times you can correctly increment the counter from 3 to 4.

### Show/Answer Explained


This section explains the method `show:` in `WAComponent`. `show:` is a variation of `call:`. You may want to skip this section if you are new to Seaside. You will find it helpful later on if you need to have more control on how components replace each other.

The method `show:` passes the control from the receiving component to the component given as the first argument. As with `call:` the receiver will be temporarily replaced by `aComponent`. However, as opposed to `call:`, `show:` does not block the flow of control and immediately returns.

If we replace the `call:` in the method `editContact:` with `show:` the application does not behave the same way as before anymore:

```
ContactListView >> editContact: aContact
    | view |
    view := ContactView new.
    view contact: aContact.
    self show: view.
    self inform: 'Thanks for editing  ' , aContact name
```


The reason is that `show:` does not block and the confirmation is displayed immediately, effectively replacing the `ContactView`. Clicking on the _Ok_ then reveals the `ContactView`. Of course, this is not the intended behavior. We can fix this issue by assigning an answer handler to the view that displays the confirmation:

```
ContactListView >> editContact: aContact
    | view |
    view := ContactView new.
    view contact: aContact.
    view onAnswer: [ :answer |
        self inform: 'Thanks for editing  ' , aContact name ].
    self show: view
```


This solves our problem, but is arguably not very readable. Luckily there is `show:onAnswer:` that combines the two method calls:

```
ContactListView >> editContact: aContact
    | view |
    view := ContactView new.
    view contact: aContact.
    self show: view onAnswer: [ :answer |
        self inform: 'Thanks for editing  ' , aContact name ]
```


In fact, what we did above is continuation-passing style. Like this we can emulate the blocking behavior of `call:` by using `show:` and a block that defines what happens afterwards. Any code that uses `call:` can be transformed like this, however in case of loops that can become quite complicated \(see \).

### Transforming a Call to a Show


Why is `show:` useful at all? 

- First of all `show:` allows one to replace multiple components in one request. This is not possible with `call:` as it blocks the flow of execution and the developer has no possibility to display another component at the same time. 
- Another reason to use `show:` is that it is more lightweight and that it uses fewer resources than `call:`. This means that if the blocking behavior is not needed, then `show:` is more memory friendly. 
- Finally some Smalltalk dialects cannot implement `call:` due to limitations in their VM.


If you want or must get rid of the `call:` statements in a sequence of calls things are relatively simple. Transform code using `call:`

```
Task >> go
  | a b c |
  a := self call: A.
  b := self call: B.
  c := self call: C.
  ...
```


to the following using `show:onAnswer:`

```
Task >> go
  self show: A onAnswer: [ :a |
    self show: B onAnswer: [ :b |
      self show: C onAnswer: [ :c |
        ... ] ] ]
```


If you have a loop like the following one, things are slightly more complicated:

```
Task >> go
  [ self call: A ]
     whileTrue: [ self call: B ]
```


The example below shows an equivalent piece of code that uses recursion to implement the loop:

```
Task >> go
  self show: A onAnswer: [ :a |
    a ifTrue: [
      self show: B onAnswer: [ :b |
        self go ] ] ]
```


The transformation technique applied here is called _continuation-passing style_ or short _CPS_. The `onAnswer:` block implements the continuation of the flow after the shown component answered. Unfortunately for more complicated flows CPS leads to messy code pretty quickly.

### Summary


In this chapter we showed how to display component using the `call:` method. In the next chapter we will demonstrate how to embed components within each other.