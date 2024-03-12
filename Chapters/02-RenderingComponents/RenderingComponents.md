## Rendering Components 
   level: 3;
   with: 'A third level heading'.
    html text: 'The next word is '.
    html strong: 'bold' ].
    instanceVariableNames: ''
    classVariableNames: ''
    package: 'SeasideBook-Hello'
   html text: 'Hello world'
   html paragraph: 'Fun with Smalltalk and Seaside.'.
   html paragraph: [
        10 timesRepeat: [
            html image
                url: 'http://www.seaside.st/styles/logo-plain.png';
                width: 50.
            html horizontalRule ] ]
    html paragraph: 'Fun with Smalltalk and Seaside.'.
    html paragraph: [
        10 timesRepeat: [
            html image
                url: 'http://www.seaside.st/styles/logo-plain.png';
                width: 50 ].
        html horizontalRule ]
    html paragraph: 'A plain text paragraph.'.
    html paragraph: [
        html render: 'A paragraph with plain text followed by a line break. '.
        html break.
        html emphasis: 'Emphasized text '.
        html render: 'followed by a horizontal rule.'.
        html horizontalRule.
        html render: 'An image URI: '.
        html image url: 'http://www.seaside.st/styles/logo-plain.png' ]
    html paragraph: [
        html render: 'today: '.
        html render: Date today ]
    html paragraph: [ html render: 'today: ' ]
    html paragraph: 'today: '
    html text: 'some string'.
    html text: Date today.
html heading: 'Hello World'.
html p with: 'Hello world.'.
html ol with: [
   html li: 'Item 1'.
   html li: 'Item 2' ].
html paragraph with: 'Hello world.'.
html orderedList with: [
   html listItem: 'Item 1'.
   html listItem: 'Item 2' ].
    html heading level: 1; with: 'Hello world'.
    html paragraph: 'Welcome to my Seaside web site.  In the
        future you will find all sorts of applications here
        such as:'.
    html orderedList with: [
        html listItem: 'Calendars'.
        html listItem: 'Todo lists'.
        html listItem: 'Shopping carts'.
        html listItem: 'And lots more...' ]
    html heading: 'Hello world'.
    html paragraph: 'Welcome to my Seaside web site.  In the
        future you will find all sorts of applications here
        such as:'.
    html orderedList: [
        html listItem: 'Calendars'.
        html listItem: 'Todo lists'.
        html listItem: 'Shopping carts'.
        html listItem: 'And lots more...' ]
    ^ #('Calendars' 'Todo lists' 'Shopping carts' 'And lots more...')
    html heading:  'Hello world'.
    html paragraph: 'Welcome to my Seaside web site.  In the
        future you will find all sorts of applications here
        such as:'.
    html orderedList: [
        self items do: [ :each | html listItem: each ] ]
    html heading: 'Hello world'.
    html paragraph: 'Welcome to my Seaside web site.  In the
        future you will find all sorts of applications here
        such as:'.
    html table: [
        html tableRow: [
            html tableHeading: 'Calendars'.
            html tableData: '1/1/2006'.
            html tableData: 'Track events, holidays etc'] .
        html tableRow: [
            html tableHeading: 'Todo lists'.
            html tableData: '5/1/2006'.
            html tableData: 'Keep track of all the things to remember to do.' ].
        html tableRow: [
            html tableHeading: 'Shopping carts'.
            html tableData: '8/1/2006'.
            html tableData: 'Enable your customers to shop online.' ] ]
    ^ 'h1 { text-align: center; }'
    " code from previous example "
    html span
        class: 'error';
        with: 'This site does not work yet'
    ^ 'h1 { text-align: center; }
   span.error { background-color: red; }'
    attributes1; 
    attributes2;
    with: anObject
html paragraph with: 'today'.
html paragraph: 'today'.