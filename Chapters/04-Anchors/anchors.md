## Anchors and Callbacks
    instanceVariableNames: ''
    classVariableNames: ''
    package: 'SeasideBook-Anchors'
    html anchor
        url: 'http://www.seaside.st';
        with: 'Seaside Website'
    instanceVariableNames: 'count'
    classVariableNames: ''
    package: 'SeasideBook-Anchors'
    super initialize.
    count := 0.
    count := count + 1
    html text: count.
    html break.
    html anchor
        callback: [ self anchorClicked ];
        with: 'click to increment'
    instanceVariableNames: 'name emailAddress'
    classVariableNames: 'Database'
    package: 'iAddress'
    ^ emailAddress
    emailAddress := aString
    ^ name
    name := aString
    ^ self new
          name: nameString;
          emailAddress: emailString;
          yourself
    Database := OrderedCollection new
        add: (self name: 'Bob Jones' emailAddress: 'bob@nowhere.com');
        add: (self name: 'Steve Smith' emailAddress: 'sm@somewhere.com');
        yourself
    "Answers an OrderedCollection of the contact information instances."

    Database isNil ifTrue: [ self createSampleDatabase ].
    ^ Database
    self contacts add: aContact
    self contacts remove: aContact
    instanceVariableNames: ''
    classVariableNames: ''
    package: 'iAddress'
    html render: aContact name; render: ' '; render: aContact emailAddress
    html unorderedList: [
        Contact contacts do: [ :contact |
            html listItem: [ self renderContact: contact on: html ] ] ]
    html anchor
        callback: [ self addContact ];
        with: 'Add contact'.
    html unorderedList: [
        Contact contacts do: [ :contact |
            html listItem: [ self renderContact: contact on: html ] ] ]
    | name emailAddress |
    name := self request: 'Name'.
    emailAddress := self request: 'Email address'.
    Contact addContact: (Contact name: name emailAddress: emailAddress)
    html text: aContact name , ' ' , aContact emailAddress.
    html text: ' ('.
    html anchor
       callback: [ self removeContact: aContact ];
       with: 'remove'.
    html text: ')'
    Contact removeContact: aContact
    (self confirm: 'Are you sure that you want to remove this contact?')
        ifTrue: [ Contact removeContact: aContact ]
    html text: aContact name.
    html space.
    html anchor
        url: 'mailto:' , aContact emailAddress;
        with: aContact emailAddress.
    html text: ' ('.
    html anchor
        callback: [ self removeContact: aContact ];
        with: 'remove'.
    html text: ')'