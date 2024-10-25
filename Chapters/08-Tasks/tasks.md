## Tasks

@cha:tasks

% +Embedding a ContactView into another component.>file://figures/iAddress-ListAndEditor.png|width=80|anchor=ref:listAndItem+

In Seaside, it is possible to define components whose responsibility is to represent the flow of control between existing components. These components are called tasks. In this chapter, we explain how you can define a task. We also show how Seaside supports application control flow by isolating certain paths from others. We will start by presenting a little game, the number guessing game. Then, we will implement two small hotel registration applications using the calendar component to illustrate tasks.

### Sequencing Components


Tasks are used to encapsulate a process or control flow. They do not directly render XHTML, but may do so via the components that they call. Tasks are defined as subclasses of  `WATask`, which implements the key method  `WATask>>go`, which is invoked as soon as a task is displayed and can call other components.

Let's start by building our first example: a number guessing game \(which was one of the first Seaside tutorials\). In this game, the computer selects a random number between 1 and 100 and then proceeds to repeatedly prompt the user for a guess. The computer reports whether the guess is too high or too low. The game ends when the user guesses the number.

!!note Those of you who remember learning to program in BASIC will recognise this as one of the common exercises to demonstrate simple user interaction. As you will see below, in Seaside it remains a simple exercise, despite the addition of the web layer. This comes as a stark contrast to other web development frameworks, which would require pages of boilerplate code to deliver such straightforward functionality.

We create a subclass of `WATask` and implement the `go` method:

```
WATask << #GuessingGameTask
    package: 'SeasideBook'
```


```
GuessingGameTask >> go
    | number guess |
    number := 100 atRandom.
        [ guess := (self request: 'Enter your guess') asNumber.
        guess < number
            ifTrue: [ self inform: 'Your guess was too low' ].
        guess > number
            ifTrue: [ self inform: 'Your guess was too high' ].
        guess = number ] whileFalse.
    self inform: 'You got it!'
```



The method `go` randomly draws a number. Then, it asks the user to guess a number and gives feedback depending on the input number. The methods `request:` and `inform:` create components \(`WAInputDialog` and `WAFormDialog`\) on the fly, which are then displayed by the task. Note that unlike the components we've developed previously, this class has no `renderContentOn:` method, just the method `go`. Its purpose is to drive the user through a sequence of steps. 

Register the application \(as 'guessinggame'\) and give it a go. Figure *@ref:game-interaction@* shows a typical execution. 

![Guessing Game interaction. %width=80&anchor=ref:game-interaction](figures/GuessingGame.png )

Why not try modifying the game to count the number of guesses that were needed?

This example demonstrates that with Seaside you can use plain Smalltalk code \(conditionals, loops, etc.,\) to define the control flow of your application. You do not have to use yet another language or build a scary XML state-machine, as required in other frameworks. In some sense, tasks are simply components that start their life in a callback. 

Because tasks are indeed components \(`WATask` is a subclass of `WAComponent`\), all of the facilities available to components, such as `call:` and `answer:` messages, are available to tasks as well. This allows you to combine components and tasks, so your `LoginUserComponent` can call a `RegisterNewUserTask`, and so on.

!!note Important Tasks do not render themselves. Don't override `renderContentOn:` in your tasks. Their purpose is simply to sequence through other views.

!!note Important If you are reusing components in a task -- that is, you store them in instance variables instead of creating new instances in the `go` method -- be sure to return these instances in the #children method so that they are backtracked properly and you get the correct control flow.


### Hotel Reservation: Tasks vs. Components


To compare when to use a task or a component, let's build a minimal hotel reservation application using a task and a component with children. Using a task, it is easy to reuse components and build a flow. Here is a small application that illustrates how to do this. We want to ask the user to specify starting and ending reservation dates. We will define a new subclass of `WATask` with two instance variables `startDate` and `endDate` of the selected period.

```
WATask << #HotelTask
   slots: { #startDate . #endDate};
   package: 'Calendars'
```


We define a method `go` that will first create a calendar with selectable dates after today, then create a second calendar with selectable days after the one selected during the first interaction, and finally we will display the dates selected as shown in Figure *@ref:hotel@*.

```
HotelTask >> go
    startDate := self call: (WAMiniCalendar new
        canSelectBlock: [ :date | date > Date today ]).
    endDate := self call: (WAMiniCalendar new
        canSelectBlock: [ :date | startDate isNil or: [ startDate < date ] ]).
    self inform: 'from ' , startDate asString , ' to ' , 
        endDate asString , ' ' , (endDate - startDate) days asString , 
        ' days'
```


![ A simple reservation based on task. % width=80&anchor=ref:hotel](figures/hotel.png)


Note that you could add a confirmation step and loop until the user is OK with his reservation.

Now this solution is not satisfying because the user cannot see both calendars while making his selection. Since we can't render components in our task, it's not easy to remedy the situation. We could use the message `addMessage: aString` to add a message to a component but this is still not good enough. This example demonstrates that tasks are about flow and not about presentation.

```
HotelTask >> go
    startDate := self call: (WAMiniCalendar new
        canSelectBlock: [ :date | date > Date today ];
        addMessage: 'Select your starting date';
        yourself).
    endDate := self call: (WAMiniCalendar new
        canSelectBlock: [ :date | startDate isNil or: [ startDate < date ] ];
        addMessage: 'Select your leaving date';
        yourself).
    self inform: (endDate - startDate) days asString , ' days: from ' ,
        startDate asString , ' to ' , endDate asString , ' '
```


### Mini Inn: Embedding Components


Let's solve the same problem using component embedding. We define a component with two calendars and two dates. The idea is that we want to always have the two mini-calendars visible on the same page and provide some feedback to the user as shown by *@ref:resa@*.

```
WAComponent << #MiniInn
    slots: { #calendar1 . #calendar2 . #startDate . #endDate};
    package: 'Calendars'
```


Since we want to show the two calendars on the same page we return them as children.

```
MiniInn >> children
    ^ Array with: calendar1 with: calendar2
```


We initialize the calendars and make sure that we store the results of their answers.

```
MiniInn >> initialize
    super initialize.
    calendar1 := WAMiniCalendar new.
    calendar1
        canSelectBlock: [ :date | Date today < date ];
        onAnswer: [ :date | startDate := date ].
    calendar2 := WAMiniCalendar new.
    calendar2
        canSelectBlock: [ :date | startDate isNil or: [ startDate < date ] ];
        onAnswer: [ :date | endDate := date ]
```


Finally, we render the application, and this time we can provide some simple feedback to the user. The feedback is simple but this is just to illustrate our point.

```
MiniInn >> renderContentOn: html
    html heading: 'Starting date'.
    html render: calendar1.
    startDate isNil
        ifFalse: [ html text: 'Selected start: ' , startDate asString ].
    html heading: 'Ending date'.
    html render: calendar2.
    (startDate isNil not and: [ endDate isNil not ]) ifTrue: [ 
        html text: (endDate - startDate) days asString , 
           ' days from ' , startDate asString , ' to ' , 
            endDate asString , ' ' ]
```


![A simple reservation with feedback. % width=80&anchor=ref:resa](figures/resa.png)

### Summary

In this chapter, we presented tasks, subclasses of `Task`. Tasks are components that do not render themselves but are used to build application flow based on the composition of other components. We saw that the composition is expressed in plain Pharo.





