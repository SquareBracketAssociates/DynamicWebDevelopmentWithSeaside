## A Really Simple Syndication
@cha:rss


RSS is a special XML format used to publish frequently updated content, such as blog posts, news items, or podcasts. Users don't need to check their favorite website for updates. Rather, they can subscribe to a URL and be notified about changes automatically. This is can be done using a dedicated tool called _feed reader_ or _aggregator_, but most web browsers integrate this capability as part of their core functionality. 

The RSS XML format is very much like XHTML, but much simpler. As standardized in the [RSS 2.0 Specification](http://cyber.law.harvard.edu/rss/rss.html), RSS essentially is composed of two parts, the _channel_ and the _news item_ specifications. While the channel describes some general properties of the news feed, the items contain the actual stories that change over time. Below we see an example of such a feed.  We will see how the same feed is presented within a feed reader.

```
<?xml version="1.0" encoding="utf-8"?>
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
```


### Creating a News Feed


There is a Seaside package extension that helps us to build such feeds in a manner similar to what we used to build XHTML for component rendering. 

Load the packages `RSS-Core`, `RSS-Examples`, and `RSS-Tests-Core` from the Seaside repository.

```
Metacello new
	githubUser: 'SeasideSt' project: 'Seaside' commitish: 'master' path: 'repository';
	baseline: 'Seaside3';
	loads: #('RSS' 'RSS Tests' 'RSS Examples')
```


Let's create a news feed for our todo items.

**Define the Feed Component.** The package defines a root class named `RRComponent` that allows you to describe both the news feed channel \(title, description, language, date of publication\) and also the news items. Therefore, the next step is to create a new subclass of `RRComponent` named `ToDoRssFeed`. This will be the entry point of our feed generator. In our example, we don't need extra instance variables.

```
RRComponent << #ToDoRssFeed
    package: 'ToDo-RSS'
```


**Register the Component as Entry Point.** Next we need to register the component at a fixed URL. The aggregator will use this URL to access the feed. We do this by adding a class side initialize method. Don't forget to evaluate the code.

```
ToDoRssFeed class >> initialize
    (WAAdmin register: RRRssHandler at: 'todo.rss')
        rootComponentClass: self
```

	
At this point we can begin to download our feed at [http://localhost:8080/todo.rss](http://localhost:8080/todo.rss), however it is mostly empty except for some standard markup as shown by the following RSS file.

!!note Your browser may be set up to handle RSS feeds automatically, so you may have difficulty in examining the raw source.

```
<?xml version="1.0" encoding="utf-8"?>
<rss version="2.0">
    <channel>
    </channel>
</rss>
```



### Render the Channel Definition


Next we create the contents of the feed. To do so we need to access our model and pass the data to the RSS renderer. As a first step we render the required tags of the channel element.

```
ToDoRssFeed >> model
    ^ ToDoList default
```


```
ToDoRssFeed >> renderContentOn: rss
    self renderChannelOn: rss
```


```
ToDoRssFeed >> renderChannelOn: rss
    rss title: self model title.
    rss link: 'http://localhost:8080/todo'.
    rss description: 'There are always things left to do.'
```


A full list of all available tags is available in the following table.


| RSS Tag | Selector | Description |
| --- | --- | --- |
| `title` | `title:` | The name of the channel \(required\). |
| `link` | `link:` | The URL to website corresponding to the channel \(required\). |
| `description` | `description:` | Phrase or sentence describing the channel \(required\). |
| `language` | `language:` | The language the channel is written in. |
| `copyright` | `copyright:` | Copyright notice for content in the channel. |
| `managingEditor` | `managingEditor:` | Email address for person responsible for editorial content. |
| `webMaster` | `webMaster:` | Email address for person responsible for technical issues. |
| `pubDate` | `pubDate:` | The publication date for the content in the channel. |
| `lastBuildDate` | `lastBuildDate:` | The last time the content of the channel changed. |
| `category` | `category:` | Specify one or more categories that the channel belongs to. |
| `generator` | `generator:` | A string indicating the program used to generate the channel. |

### Rendering News Items


Finally, we want to render the todo items. Each news item is enclosed within a `item` tag. We will display the title and show the due date as part of the description. Also we prepend the string `(done)`, if the item has been completed.

```
ToDoRssFeed >> renderContentOn: rss
    self renderChannelOn: rss.
    self model items
        do: [ :each | self renderItem: each on: rss ]
```


```
ToDoRssFeed >> renderItem: aToDoItem on: rss
    rss item: [
        rss title: aToDoItem title.
        rss description: [
            aToDoItem done
                ifTrue: [ rss text: '(done) ' ].
            rss render: aToDoItem due ] ]
```

		
Doing so will generate the required XML structure for the item tag.

```
<item>
    <title>Smalltalk</title>
    <description>(done) 5 March 2008</description>
</item>
```


At the minimum, a title or a description must be present. All the other sub-elements are optional. 


| RSS Tag | Selector | Description |
| --- | --- | --- |
| `title` | `title` | The title of the item. |
| `link` | `link` | The URL of the item. |
| `description` | `description` | Phrase or sentence describing the channel. |
| `author` | `author` | The item synopsis. |
| `category` | `category` | Includes the item in one or more categories. |
| `comments` | `comments` | URL of a page for comments relating to the item. |
| `enclosure` | `enclosure` | Describes a media object that is attached to the item. |
| `guid` | `guid` | A string that uniquely identifies the item. |
| `pubDate` | `pubDate` | Indicates when the item was published. |
| `source` | `source` | The RSS channel that the item came from. |

### Subscribe to the Feed


Now we have done all that is required to let users subscribe. Figure *@feed-vienna@*  shows
how the feed is presented to the user in the feed reader when the URL was added manually.

![The ToDo Feed subscribed. % width=70&anchor=feed-vienna](figures/feed-vienna.png)

One remaining thing to do is to tell the users of our todo application where they can subscribe to the RSS feed. Of course we could simply put an anchor at the bottom our web application, however there is a more elegant solution. We override the method  `WAComponent>>updateRoot:` in our Seaside component to add a link to our feed into the XHTML head. Most modern web browser will pick up this tag and show the RSS logo in the toolbar to allow people to register for the feed with one click.

```
ToDoListView >> updateRoot: aHtmlRoot
    super updateRoot: aHtmlRoot.
    aHtmlRoot link
        beRss;
        title: self model title;
        url: 'http://localhost:8080/todo.rss'
```

	
Note the use of the message `beRss` tells the web browser that the given link points to an RSS feed.

In Firefox you may have to add the relationship property to make the rss logo visible.
 
```
updateRoot: aHtmlRoot
    super updateRoot: aHtmlRoot.
    aHtmlRoot link
        beRss;
        relationship: 'alternate';
        title: self model title;
        url: 'http://localhost:8080/todo.rss'
```

	
### Summary


In Seaside you don't manipulate tags directly. The elegant generation of RSS feeds nicely shows how the canvas can be extended to produce something other than XHTML. In particular, it is important to see that Seaside is not limited to serve XHTML but can be extended to serve SVG, WAP and RSS.