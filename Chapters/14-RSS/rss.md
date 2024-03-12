## A Really Simple Syndication
<rss version="2.0">
   <channel>
      <title>Seaside ToDo</title>
      <link>http://localhost:8080/todo</link>
      <description>There are always things left to do.</description>
      <item>
         <title>Smalltalk</title>
         <description>(done) 5 March 2008</description>
      </item>
      <item>
         <title>Seaside</title>
         <description>5 September 2008</description>
      </item>
      <item>
         <title>Scriptaculous</title>
         <description>7 September 2008</description>
      </item>
   </channel>
</rss>
	githubUser: 'SeasideSt' project: 'Seaside' commitish: 'master' path: 'repository';
	baseline: 'Seaside3';
	loads: #('RSS' 'RSS Tests' 'RSS Examples')
    instanceVariableNames: ''
    classVariableNames: ''
    package: 'ToDo-RSS'
    (WAAdmin register: RRRssHandler at: 'todo.rss')
        rootComponentClass: self
<rss version="2.0">
    <channel>
    </channel>
</rss>
    ^ ToDoList default
    self renderChannelOn: rss
    rss title: self model title.
    rss link: 'http://localhost:8080/todo'.
    rss description: 'There are always things left to do.'
    self renderChannelOn: rss.
    self model items
        do: [ :each | self renderItem: each on: rss ]
    rss item: [
        rss title: aToDoItem title.
        rss description: [
            aToDoItem done
                ifTrue: [ rss text: '(done) ' ].
            rss render: aToDoItem due ] ]
    <title>Smalltalk</title>
    <description>(done) 5 March 2008</description>
</item>
    super updateRoot: aHtmlRoot.
    aHtmlRoot link
        beRss;
        title: self model title;
        url: 'http://localhost:8080/todo.rss'
    super updateRoot: aHtmlRoot.
    aHtmlRoot link
        beRss;
        relationship: 'alternate';
        title: self model title;
        url: 'http://localhost:8080/todo.rss'