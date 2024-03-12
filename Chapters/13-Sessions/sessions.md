## Managing Sessions
    callback: [ self show: (WAInspector current on: self session) ];
    with: 'Inspect Session'
   instanceVariableNames: 'user'
   classVariableNames: ''
   package: 'SeasideBook'
    user := aString
    user := nil
    ^ user isNil not
    ^ user
    | application |
    application := WAAdmin register: self asApplicationAt: 'miniInn'.
    application preferenceAt: #sessionClass put: InnSession
    self session login: (self request: 'Enter your name:')
    self session logout
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
    html text: 'Dear ' , self session user, ', you can benefit from our special prices!'
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
anApplication cache expiryPolicy configuration 
    at: #cacheTimeout put: 1200
    super unregistered.
    user := nil
    user := nil.
    self expire
    user := nil.
    self expire.
    self redirectTo: 'http://www.seaside.st'