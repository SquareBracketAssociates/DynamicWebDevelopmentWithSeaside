## A Simple ToDo Application
    instanceVariableNames: 'title due done'
    classVariableNames: ''
    package: 'ToDo-Model'
    ^ title
    title := aString
    ^ due
    due := aDate asDate
    ^ done
    done := aBoolean
    self title: 'ToDo Item'.
    self due: Date tomorrow.
    self done: false.
    ^ self basicNew initialize
    ^ self done
    ^ self isDone not and: [ Date today > self due ]
   instanceVariableNames: 'title items'
    classVariableNames: 'Default'
    package: 'ToDo-Model'
   self items: OrderedCollection new
    self items add: aTodoItem
   ^ self items remove: aTodoItem
    ^ Default ifNil: [ Default := self new ]
   Default := nil
    "self initializeExamples"

    self default
        title: 'Seaside ToDo';
        add: (ToDoItem new
            title: 'Finish todo app chapter';
            due: '11/15/2007' asDate;
            done: false);
        add: (ToDoItem new
            title: 'Annotate first chapter';
            due: '04/21/2008' asDate;
            done: true);
         add: (ToDoItem new
            title: 'Watch out for UNIX Millenium bug';
            due: '01/19/2038' asDate;
            done: false)
    instanceVariableNames: ''
    classVariableNames: ''
    package: 'ToDo-View'
    "self initialize"
    WAAdmin register: self asApplicationAt: 'todo'
    ^ ToDoList default
   html text: self model title
    html heading: self model title
    html listItem
        class: 'done' if: anItem isDone;
        class: 'overdue' if: anItem isOverdue;
        with: anItem title
    self model items
        do: [ :each | self renderItem: each on: html ]
    html heading: self model title.
    html unorderedList: [ self renderItemsOn: html ]
    ^ 'body {
    color: #222;
    font-size: 75%;
    font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
}
h1 {
    color: #111;
    font-size: 2em;
    font-weight: normal;
    margin-bottom: 0.5em;
}
ul {
    list-style: none;
    padding-left: 0;
    margin-bottom: 1em;
}
li.overdue {
    color: #8a1f11;
}
li.done {
    color: #264409;
}'
    html listItem
        class: 'done' if: anItem isDone;
        class: 'overdue' if: anItem isOverdue;
        with: [
            html text: anItem title.
            html space.
            html anchor
                callback: [ self edit: anItem ];
                with: 'edit'.
            html space.
            html anchor
                callback: [ self remove: anItem ];
                with: 'remove' ]
    self inform: anItem title
    (self confirm: 'Are you sure you want to remove ' , anItem title printString , '?')
        ifTrue: [ self model remove: anItem ]
    html heading: self model title.
    html anchor 
        callback: [ self add ];
        with: 'add'.
    html unorderedList: [ self renderItemsOn: html ]
    self model add: ToDoItem new
   html heading: self model title.
    html form: [
        html anchor 
             callback: [ self add ];
             with: 'add'.
        html unorderedList: [ self renderItemsOn: html ].
        html submitButton: 'Save' ]
    html listItem
        class: 'done' if: anItem isDone;
        class: 'overdue' if: anItem isOverdue;
        with: [
            html checkbox
                value: anItem done;
                callback: [ :value | anItem done: value ].
            html render: anItem title.
            html space.
            html anchor
                callback: [ self edit: anItem ];
                with: 'edit'.
            html space.
            html anchor
                callback: [ self remove: anItem ];
                with: 'remove' ]
    instanceVariableNames: 'model'
    classVariableNames: ''
    package: 'ToDo-View'
    ^ model
    model := aModel
    html heading: 'Edit'.
    html form: [
        html text: 'Title:'; break.
        html textInput
            value: self model title;
            callback: [ :value | self model title: value ].
        html break.
        html text: 'Due:'; break.
        html dateInput
            value: self model due;
            callback: [ :value | self model due: value ] ]
    self call: (ToDoItemView new model: anItem)
    ^ 'body {
    color: #222;
    font-size: 75%;
    font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
}
h1 {
    color: #111;
    font-size: 2em;
    font-weight: normal;
    margin-bottom: 0.5em;
}'
    html heading: 'Edit'.
    html form: [
        html text: 'Title:'; break.
        html textInput
            value: self model title;
            callback: [ :value | self model title: value ].
        html break.
        html text: 'Due:'; break.
        html dateInput
            value: self model due;
            callback: [ :value | self model due: value ].
        html break.
        html submitButton
            callback: [ self answer: self model ];
            text: 'Save'.
        html cancelButton
            callback: [ self answer: nil ];
            text: 'Cancel' ]
    | edited |
    edited := self call: (ToDoItemView new model: anItem copy).
    edited isNil
        ifFalse: [ self model replace: anItem with: edited ]
    self items at: (self items indexOf: aTodoItem) put: anotherItem
   instanceVariableNames: 'editor'
   classVariableNames: ''
   package: 'ToDo-View'
    ^ Array with: editor
    html heading: self model title.
    html form: [
        html unorderedList: [ self renderItemsOn: html ].
        html submitButton
            text: 'Save'.
        html submitButton
            callback: [ self add ];
            text: 'Add' ].
    html render: editor
    editor := ToDoItemView new model: ToDoItem new
    editor := ToDoItemView new model: ToDoItem new.
    editor onAnswer: [ :value |
       value isNil
           ifFalse: [ self model add: value ].
       editor := nil ]