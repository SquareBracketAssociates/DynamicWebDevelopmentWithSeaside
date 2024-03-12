## Forms
    instanceVariableNames: 'contact'
    classVariableNames: ''
    package: 'iAddress'
    ^ contact ifNil: [ contact := Contact contacts first ]
    contact := aContact
    html form: [
        html text: 'Name: '.
        html textInput
            callback: [ :value | self contact name: value ];
            value: self contact name.
        html break.
        html text: 'Email address: '.
        html textInput
            callback: [ :value | self contact emailAddress: value ];
            value: self contact emailAddress.
        html break.
        html submitButton
            callback: [ self save ];
            value: 'Save']
    "For now let's just display the contact information"
    self inform: self contact name , '--' , self contact emailAddress
   callback: [ :value | self contact name: value ];
   with: self contact name
   callback: [ self save ];
   value: 'Save'
    html form: [
        html text: 'Name:'.
        html textInput on: #name of: self contact.
        html break.
        html text: 'Email address:'.
        html textInput on: #emailAddress of: self contact.
        html break.
        html submitButton on: #save of: self ]
    instanceVariableNames: 'name emailAddress gender'
    classVariableNames: 'Database'
    package: 'iAddress'
    ^ gender ifNil: [ gender := #Male ]
    ^ self gender = #Male
    ^ self gender = #Female
    gender := #Male
    gender := #Female
    html form: [
        html text: 'Name:'.
        html textInput on: #name of: self contact.
        html break.
        html text: 'Email address:'.
        html textInput on: #emailAddress of: self contact.
        html break.

        "Drop-Down Menu"
        html text: 'Gender: '.
        html select 
            list: #(#Male #Female);
            selected: self contact gender;
            callback: [ :value |
                value = #Male
                    ifTrue: [ self contact beMale ]
                    ifFalse: [ self contact beFemale ] ].
        html break.

        html submitButton on: #save of: self ]
    self inform: self contact name , 
        '--' , self contact emailAddress , 
        '--' , self contact gender
    html form: [
        html text: 'Name:'.
        html textInput on: #name of: self contact.
        html break.
        html text: 'Email address:'.
        html textInput on: #emailAddress of: self contact.
        html break.

        "List Box"
        html text: 'Gender: '.
        html select
            size: 2;
            list: #(#Male #Female);
            selected: self contact gender;
            callback: [ :value | 
                value = #Male
                    ifTrue: [ self contact beMale ]
                    ifFalse: [ self contact beFemale ] ].
        html break.
        html submitButton on: #save of: self]
    | group |
    html form: [
        html text: 'Name:'.
        html textInput on: #name of: self contact.
        html break.
        html text: 'Email address:'.
        html textInput on: #emailAddress of: self contact.
        html break.

        "Radio Buttons"
        html text: 'Gender: '.
        group := html radioGroup.
        group radioButton
            selected: self contact isMale;
            callback: [ self contact beMale ].
        html text: 'Male'.
        group radioButton
            selected: self contact isFemale;
            callback: [ self contact beFemale ].
        html text: 'Female'.
        html break.

        html submitButton on: #save of: self ]
    instanceVariableNames: 'name emailAddress gender requestsEmailUpdates'
    classVariableNames: 'Database'
    package: 'iAddress'
    ^ requestsEmailUpdates ifNil: [ requestsEmailUpdates := false ]
    requestsEmailUpdates := aBoolean
    | group |
    html form: [
        html text: 'Name:'.
        html textInput on: #name of: self contact.
        html break.
        html text: 'Email address:'.
        html textInput on: #emailAddress of: self contact.
        html break.
        html text: 'Gender: '.
        group := html radioGroup.
        group radioButton
            selected: self contact isMale;
            callback: [ self contact beMale ].
        html text: 'Male'.
        group radioButton
            selected: self contact isFemale;
            callback: [ self contact beFemale ].
        html text: 'Female'.
        html break.

        "Checkbox"
        html text: 'Send email updates: '.
        html checkbox
            value: self contact requestsEmailUpdates;
            callback: [ :value | self contact requestsEmailUpdates: value ].
        html break.

        html submitButton on: #save of: self ]
    self inform: self contact name ,
        '--' , self contact emailAddress ,
        '--' , self contact gender ,
        '--' , self contact requestsEmailUpdates printString
        callback: [ :value | self contact birthdate: value ];
        with: self contact birthdate.
    html break.
    | group |
    html form: [
        html text: 'Name:'.
        html textInput on: #name of: self contact.
        html break.
        html text: 'Email address:'.
        html textInput on: #emailAddress of: self contact.
        html break.
        html text: 'Gender: '.
        group := html radioGroup.
        group radioButton
            selected: self contact isMale;
            callback: [ self contact beMale ].
        html text: 'Male'.
        group radioButton
            selected: self contact isFemale;
            callback: [ self contact beFemale ].
        html text: 'Female'.
        html break.
        html text: 'Send email updates: '.
        html checkbox
            value: self contact requestsEmailUpdates;
            callback: [ :value | self contact requestsEmailUpdates: value ].
        html break.

        "Date Input"
        html text: 'Birthday: '.
        html dateInput
            callback: [ :value | self contact birthdate: value ];
            with: self contact birthdate.
        html break.

        html submitButton on: #save of: self ]
    self inform: self contact name ,
        '--' , self contact emailAddress ,
        '--' , self contact gender ,
        '--' , self contact requestsEmailUpdates printString ,
        '--' , self contact birthdate printString
    html form multipart; with: [
        html fileUpload
            callback: [ :value | self receiveFile: value ].
        html submitButton: 'Send File' ]
    | stream |
    stream := (FileDirectory default directoryNamed: 'uploads')
        assureExistence;
        forceNewFileNamed: aFile fileName.
    [ stream binary; nextPutAll: aFile rawContents ] 
        ensure: [ stream close ]
    html form multipart; with: [
        html fileUpload
            callback: [ :value | file := value ].
        html submitButton: 'Send File' ].
    file notNil ifTrue: [
        html anchor
            callback: [ self downloadFile ];
            with: 'Download' ]
    self requestContext respond: [ :response |
        response
            contentType: file contentType;
            document: file rawContents asString;
            attachmentWithFileName: file fileName ]