## REST
  squeaksource: 'Seaside30Addons';
  package: 'Seaside-REST-Core';
  package: 'Seaside-Pharo-REST-Core';
  package: 'Seaside-Tests-REST-Core';
  load.
   instanceVariableNames: ''
   classVariableNames: ''
   package: 'ToDo-REST'
   WAAdmin register: self at: 'todo-api'
   <get>
  	
   ^ String streamContents: [ :stream |
      ToDoList default items do: [ :each |
         stream nextPutAll: each title; crlf ] ]
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 71
Date: Sun, 20 Nov 2011 17:04:52 GMT
Server: Zinc HTTP Components 1.0

Finish todo app chapter
Annotate first chapter
Discuss cover design
   <get>

   self requestContext respond: [ :response |
      ToDoList default items do: [ :each |
         response contentType: 'text/plain'.
         response
            nextPutAll: each title;
            nextPutAll: String crlf ] ]
   <post>
	
   ToDoList default items
      add: (ToDoItem new
         title: self requestContext request rawBody;
         yourself).
   ^ 'OK'
OK
Finish todo app chapter
Annotate first chapter
Discuss cover design
Give REST a try
   <get>
   <produces: 'text/json'>
   
   ^ (Array streamContents: [ :stream |
      ToDoList default items do: [ :each |
         stream nextPut: (Dictionary new
            at: 'title' put: each title;
            at: 'done' put: each done;
            yourself) ] ])
      asJavascript
   <get>
   <produces: 'text/xml'>

   ^ WAXmlCanvas builder 
      documentClass: WAXmlDocument;
      render: [ :xml |
         xml tag: 'items' with: [ 
            ToDoList default items do: [ :each |
               xml tag: 'item' with: [
                  xml tag: 'title' with: each title.
                  xml tag: 'due' with: each due ] ] ] ]
 {"title": "Annotate first chapter", "done": true},
 {"title": "Discuss cover design", "done": false},
 {"title": "Give REST a try", "done": true}]
<items>
  <item>
    <title>Finish todo app chapter</title>
    <done>false</done>
  </item>
  <item>
    ... 
   <post>
   <consumes: '*/json'>

   | json |
   json := JSJsonParser parse: self requestContext request rawBody.
   ToDoList default items
      add: (ToDoItem new
         title: (json at: 'title');
         done: (json at: 'done' ifAbsent: [ false ]);
         yourself).
   ^ 'OK'
   -d '{"title": "Check out latest Seaside"}' \
   http://localhost:8080/todo-api
OK
   <get>
   	
   ^ String streamContents: [ :stream |
      ToDoList default items do: [ :each |
         stream nextPutAll: each title; crlf ] ]
   <get>

   | item |
   item := ToDoList default items
      detect: [ :each | each title = aTitleString ]
      ifNone: [ ^ 'unknown todo item' ].
   anActionString = 'isDone' ifTrue: [
      ^ item done
         ifTrue: [ 'done' ]
         ifFalse: [ 'todo' ] ].
   ...
   ^ 'invalid command'
unknown todo item
$ curl http://localhost:8080/todo-api/Discuss+cover+design/isDone
done
$ curl http://localhost:8080/todo-api/Annotate+first+chapter/isDone
todo
   <get>
   <path: '/search?query={aString}'>

   ^ String streamContents: [ :stream |
      ToDoList default items do: [ :each |
         (each title includesSubString: aString)
            ifTrue: [ stream nextPutAll: each title; crlf ] ] ]
Give REST a try
   instanceVariableNames: ''
   classVariableNames: ''
   package: 'ToDo-REST'
   (WAAdmin register: self asApplicationAt: 'todo')
      addFilter: ToDoFilter new
    <get>
    <produces: 'text/html'>

    ^ self noRouteFound: self requestContext
    <get>

    self requestContext request inspect.
    ^ String streamContents: [ :stream |
        ...
    <delete>
   	
    | item |
    item := ToDoList default items
       detect: [ :each | each title = aString ]
       ifNone: [ nil ].
    self requestContext respond: [ :response |
       item isNil
         ifTrue: [ response status: WARequest statusNotFound ]
         ifFalse: [ 
            ToDoList default remove: item.
            response status: WARequest statusOk ] ]