## Tasks
    instanceVariableNames: ''
    classVariableNames: ''
    package: 'SeasideBook'
    | number guess |
    number := 100 atRandom.
        [ guess := (self request: 'Enter your guess') asNumber.
        guess < number
            ifTrue: [ self inform: 'Your guess was too low' ].
        guess > number
            ifTrue: [ self inform: 'Your guess was too high' ].
        guess = number ] whileFalse.
    self inform: 'You got it!'
   instanceVariableNames: 'startDate endDate'
   classVariableNames: ''
   package: 'Calendars'
    startDate := self call: (WAMiniCalendar new
        canSelectBlock: [ :date | date > Date today ]).
    endDate := self call: (WAMiniCalendar new
        canSelectBlock: [ :date | startDate isNil or: [ startDate < date ] ]).
    self inform: 'from ' , startDate asString , ' to ' , 
        endDate asString , ' ' , (endDate - startDate) days asString , 
        ' days'
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
    instanceVariableNames: 'calendar1 calendar2 startDate endDate'
    classVariableNames: ''
    package: 'Calendars'
    ^ Array with: calendar1 with: calendar2
    super initialize.
    calendar1 := WAMiniCalendar new.
    calendar1
        canSelectBlock: [ :date | Date today < date ];
        onAnswer: [ :date | startDate := date ].
    calendar2 := WAMiniCalendar new.
    calendar2
        canSelectBlock: [ :date | startDate isNil or: [ startDate < date ] ];
        onAnswer: [ :date | endDate := date ]
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