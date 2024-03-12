## JQuery
html jQuery: 'div.hint'.                 "(2)"
html jQuery id: 'foo'.                   "(2)"
html jQuery: '#foo'.                     "(3)"
html jQuery: #foo.                       "(4)"
html jQuery: '*'.
html jQuery all.
html jQuery new.
html jQuery html: [ :r | r div ].
html jQuery: [ :r | r div ].
html jQuery: (html javascript alert: 'Hello').
aQuery siblings: 'div'.
aQuery next: 'div'.
aQuery nextAll: 'div'.
aQuery previous: 'div'.
aQuery previousAll: 'div'.
aQuery children: 'div'.
aQuery parent: 'div'.
aQuery parents: 'div'.
aQuery closest: 'div'.
aQuery removeClass: 'important'.
aQuery toggleClass: 'important'.
aQuery cssAt: 'color' put: '#ff0'.
aQuery attributeAt: 'href' put: 'http://www.seaside.st/'.
aQuery prepend: [ :r | r div ].
aQuery append: [ :r | r div ].
aQuery after: [ :r | r div ].
aQuery show: 1 second.
aQuery hide: 1 second.
    onClick: (html jQuery: 'div') remove;
    with: 'Remove DIVs'
    onClick: (html jQuery this) remove;
    with: 'Remove Myself'
anAjax script: [ :s | s alert: 'Hello' ].
anAjax trigger: [ :p | ... ] passengers: aQuery.
anAjax callback: [ :v | ... ] value: anObject.
     onClick: (html jQuery: 'div.help') toggle;
     with: 'About jQuery'.
 
html div
    class: 'help';
    style: 'display: none';
    with: 'jQuery is a fast and ...'
html div
    id: (id := html nextId);
    script: (html jQuery new dialog
         title: 'Lightbox Dialog';
         modal: true);
     with: [ self renderDialogOn: html ]
html anchor
     onClick: (html jQuery id: id) dialog open;
     with: 'Open Lightbox'
     id: (id := html nextId);
     with: child.

html anchor
     onClick: ((html jQuery id: id) load
         html: [ :r | 
             child := OtherComponent new;
             r render: child ]);
     with: 'Change Component'
...
html div id: #time.

html anchor
    onClick: (html jQuery ajax script: [ :s |
        s << (s jQuery: #date)
             html: [ :r | r render: Date today ].
        s << (s jQuery: #time)
             html: [ :r | r render: Time now ] ]);
     with: 'Update'
    self renderHeadingOn: html.    "<-- added." 
    html form: [
        html unorderedList
            id: 'items';
            with: [ self renderItemsOn: html ].
        html submitButton
            text: 'Save'.
        html submitButton
            callback: [ self add ];
            text: 'Add' ].
    html render: editor
    html heading
        onClick: html jQuery this effect highlight;
        with: self model title.
    | helpId |
    helpId := html nextId.
    (html heading)
        class: 'helplink';
        onClick: ((html jQuery id: helpId)
            slideToggle: 1 seconds);
        with: self model title.
    (html div)
        id: helpId;
        class: 'help';
        style: 'display: none';
        with: 'The ToDo app enhanced with jQuery.'
	^ '
	.help {
	    padding: 1em;
	    margin-bottom: 1em;
	    border: 1px solid #008aff;
	    background-color: #e6f4ff;
	}
	.helplink {
	    cursor: help;
	}

	body {
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