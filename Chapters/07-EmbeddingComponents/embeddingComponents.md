## Embedding Components
    instanceVariableNames: 'editor'
    classVariableNames: ''
    package: 'iAddress'
    ^ Contact contacts
    super initialize.
    editor := ContactView new.
    editor contact: self contacts first
    ^ Array with: editor
    Contact addContact: aContact
    | name emailAddress |
    name := self request: 'Name'.
    emailAddress := self request: 'Email address'.
    self addContact: (Contact name: name emailAddress: emailAddress)
    editor contact: aContact
    (self confirm: 'Are you sure that you want to remove this contact?')
        ifTrue: [ Contact removeContact: aContact ]
    html form: [
        self renderTitleOn: html.
        self renderBarOn: html.
        html table: [ 
            html tableRow: [
                html tableHeading: 'Name'.
                html tableHeading: 'Address' ].
            self renderDatabaseRowsOn: html ].
        html horizontalRule.
        html render: editor ]
    html heading level: 2; with: 'iAddress'
   html anchor
       callback: [ self askAndCreateContact ]; 
       with: 'Add contact'
    self contacts do: [ :contact |
        html tableRow: [ self renderContact: contact on: html ] ]
    html tableData: [
        html anchor
            callback: [ self editContact: aContact ];
            with: aContact name].
    html tableData: aContact emailAddress.
    html tableData: [
        html anchor
            callback: [ self removeContact: aContact ];
            with: '--' ]
    instanceVariableNames: 'parent'
    classVariableNames: ''
    package: 'iAddress'
    parent := aParent
        html anchor
            callback: [parent editContact: self contact];
            with: self contact name.
        html space.
        html text: self contact emailAddress.
        html space.
        html anchor
            callback: [parent removeContact: self contact];
            with: '--'.
        html break.
    instanceVariableNames: 'contactViewers editor'
    classVariableNames: ''
    package: 'iAddress'
    ^ Contact contacts
    super initialize.
    editor := ContactView new.
    contactViewers := IdentityDictionary new.
    self contacts do: [ :each | self addContactViewerFor: each ]
    contactViewers
        at: aContact
        put: (ReadOnlyOneLinerContactView new
            contact: aContact;
            parent: self; " <-- added "
            yourself)
     | name emailAddress |
     name := self request: 'Name'.
     emailAddress := self request: 'Email address'.
     self addContact: (Contact name: name emailAddress: emailAddress)
    editor := anEditor
    ^ contactViewers values
    contactViewers values
        do: [ :each | html render: each. html break ]
    Contact addContact: aContact.
    self addContactViewerFor: aContact
    contactViewers removeKey: aContact.
    Contact removeContact: aContact
    instanceVariableNames: 'list'
    classVariableNames: ''
    package: 'iAddress'
    super initialize.
    list := PluggableContactListView new.
    list editor: editor  " <-- added "
    ^ super children , (Array with: list)
    html anchor
        callback: [ list askAndCreateContact ]; 
        with: 'Add contact'
    html form: [
        self renderTitleOn: html.
        self renderBarOn: html.
        html break.
        html render: list.
        html horizontalRule.
        html render: editor ]
    super initialize.
    editor := ContactView new.
    editor contact: self contacts first.
    editor onAnswer: [ :answer | self inform: 'Saved' ]   " <-- added "
    "should be called during action phase"
    methods :=  #(renderNameOn: renderEmailOn:)
   methods do: [ :each | self perform: each with: html ] 
    self embedded
        ifTrue: [ aBlock value ]
        ifFalse: [ html form: aBlock ]
    self renderContent: html body: [
        "lots of form elements get rendered here"
        self renderSaveOn: html ]
    html form: [
        "some form elements here, followed by our embeddable Component:"
        html render: (EmbeddableFormComponent new beEmbedded; yourself) ]
    addDecoration: aDecoration; 
    yourself)
    super initialize.
    editor := ContactView new.
    editor contact: self contacts first.
    editor addMessage: 'Editing...'.  " <-- added "
    editor onAnswer: [ :answer | self inform: 'Saved', answer printString ]
    html render: (AnotherComponent new addMessage: 'Another Component'; yourself)
    super initialize.
    self addDecoration: (WAFormDecoration new buttons: #(cancel save)).
    self addDecoration: (WAWindowDecoration new title: 'Zork Contacts')
    "Replace the current page with aComponent."

    WARenderLoop new
        call: (aComponent
            addDecoration: (WAWindowDecoration new
                cssClass: self cssClass;
                title: self label;
                yourself);
            yourself)
        withToolFrame: false
    form := WAFormDecoration new buttons: self buttons.
    self addDecoration: form
    ^ #(ok)
    super initialize.
    self addDecoration: (WAFormDecoration new buttons: #(cancel save))
    self renderNameOn: html.
    self renderEmailOn: html.
    self renderGenderOn: html.
    self renderSendUpdatesOn: html.
    self renderDateOn: html.
    super initialize.
    form := WAFormDecoration new buttons: self buttons.
    self addDecoration: form.
    self validateWith: [ :answer | answer validate ].
    self addMessage: 'Please enter the following information'.
    ^ announcer ifNil: [ announcer := Announcer new ]
    instanceVariableNames: 'child'
    classVariableNames: ''
    package: 'iAddress'
    ^ self new
        child: aChild;
        yourself
    child := anChild
    ^ child
    super initialize.
    self session announcer on: RemoveChild do: [:it | self removeChild: it child]
    self children remove: aChild
    self session announcer announce: (RemoveChild child: self)