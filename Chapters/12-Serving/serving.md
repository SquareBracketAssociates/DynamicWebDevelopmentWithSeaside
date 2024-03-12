## Serving Files
    html image url: 'http://www.seaside.st/styles/logo-plain.png'
    html image resourceUrl: 'styles/logo-plain.png'
    html image form: aForm
    html image form: (EllipseMorph new 
       color: Color orange;
       extent: 200 @ 100;
       borderWidth: 3;
       imageForm)
    instanceVariableNames: ''
    classVariableNames: ''
    package: 'Serving-Files'
    super updateRoot: anHtmlRoot.
    anHtmlRoot stylesheet url: 'http://seaside.st/styles/main.css'
    html heading level: 1; with: 'Seaside'.
    html text: 'This component uses the Seaside style.'
MyFileLibrary addFileAt: '/path/to/background.png'
    html image url: (MyFileLibrary urlOf: #pictureJpg)
    instanceVariableNames: ''
    classVariableNames: ''
    package: 'Test'
    html image url: (CounterLibrary urlOf: #seasidePng).
    html heading: count.
    html anchor
        callback: [ self increase ];
        with: '++'.
    html space.
    html anchor
        callback: [ self decrease ];
        with: '--'
   super updateRoot: anHtmlRoot.
   anHtmlRoot stylesheet url: (CounterLibrary urlOf: #seasideCss)
<link rel="stylesheet" type="text/css" href="/files/CounterLibrary.css"/>
</head>
<body onload="onLoad()" onkeydown="onKeyDown(event)">
  <img alt="" src="/files/CounterLibrary/seaside.png"/>
  <h1>0</h1>
  <a href="http://localhost:8080/WebCounter?_s=UwGcN6vwGVmj9icD&amp;_k=D6Daqxer&amp;1">++</a>&nbsp;
  <a href="http://localhost:8080/WebCounter?_s=UwGcN6vwGVmj9icD&amp;_k=D6Daqxer&amp;2">--</a>
...
>>> 'AB'
>>> ByteString
>>> '[]'
>>>  the copyright character &copy;
>>> the u-umlaut character &uuml;
`WAKomEncoded startOn: 8081

<meta content="text/html;charset=utf-8"
http-equiv="Content-Type"/>