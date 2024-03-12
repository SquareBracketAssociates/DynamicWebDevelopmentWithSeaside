## Calling Components
    self call: (ContactView new 
        contact: aContact;
        yourself)
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
    self answer
    | view |
    view := ContactView new.
    view contact: aContact.
    self call: view.
    self inform: 'Thanks for editing  ' , aContact name
    html form: [
        self renderNameOn: html.
        self renderEmailOn: html.
        self renderGenderOn: html.
        self renderSendUpdatesOn: html.
        self renderDateOn: html.
        self renderSaveOn: html ]
        html submitButton on: #cancel of: self. " <-- added "
        html submitButton on: #save of: self
    self answer: true
    self answer: false
    | view |
    view := ContactView new.
    view contact: aContact.
    (self call: view)
        ifFalse: [ ^ nil ].
    self inform: 'Thanks for editing ' , aContact name
    | view copy |
    view := ContactView new.
    copy := aContact copy.
    view contact:  copy.
    (self call: view) 
        ifTrue: [ Contact removeContact: aContact; addContact: copy ]
    "Display a dialog with aString to the user until he clicks the ok button."
    
    ^ self wait: [ :cc | self inform: aString onAnswer: cc ]
    ^ Array with: count
    ^ Array with: self
    | view |
    view := ContactView new.
    view contact: aContact.
    self show: view.
    self inform: 'Thanks for editing  ' , aContact name
    | view |
    view := ContactView new.
    view contact: aContact.
    view onAnswer: [ :answer |
        self inform: 'Thanks for editing  ' , aContact name ].
    self show: view
    | view |
    view := ContactView new.
    view contact: aContact.
    self show: view onAnswer: [ :answer |
        self inform: 'Thanks for editing  ' , aContact name ]
  | a b c |
  a := self call: A.
  b := self call: B.
  c := self call: C.
  ...
  self show: A onAnswer: [ :a |
    self show: B onAnswer: [ :b |
      self show: C onAnswer: [ :c |
        ... ] ] ]
  [ self call: A ]
     whileTrue: [ self call: B ]
  self show: A onAnswer: [ :a |
    a ifTrue: [
      self show: B onAnswer: [ :b |
        self go ] ] ]