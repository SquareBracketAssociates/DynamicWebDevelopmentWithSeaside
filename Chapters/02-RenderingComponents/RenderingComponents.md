## Rendering Components 
@cha:rendering-components

In this chapter, you will learn the basics of displaying text and other information such as tables and lists with Seaside and its powerful XHTML manipulation interface. You will learn how to create a _component_ which could include text, a form, a picture, or anything else that you would like to display in a web browser. Seaside's component framework is one of its most powerful features and writing an application in Seaside amounts to creating and manipulating components. You will learn how to use Seaside's API, which is based on the concept of \`\`brushes'', to generate valid XHTML.

### Brush concept

One of the great features of Seaside is that you do not have to worry about manipulating HTML yourself and creating valid XHTML.  Seaside produces valid XHTML: it automatically generates HTML markup using valid tags, it ensures that the tags are nested correctly and it closes the tags for you.  

Let's look at a simple example: to force a line-break in HTML (for instance, to separate the lines of a postal address) you need to use a break tag: `<br/>`. Some people use `<br>` or `<br></br>`, and neither is valid in XHTML. Some browsers will accept these incorrect forms without a problem, but some will mark them as errors. If your content is getting passed on through RSS or Atom clients, it may fail in unexpected ways. You do not need to worry about any of this when using Seaside.

The basic metaphor used in Seaside for rendering HTML is one of painting on a _canvas_ using _brushes_. Methods such as `renderContentOn:` that are called to render content are passed an argument (by convention named `html`) that refers to the current canvas object. To render content on this object you can call its methods (or to use the correct Smalltalk terminology, you can pass it messages). In the simple example given above, to add a line-break to a document you would use `html break`. 

When you send a message to the canvas, you're actually asking it to start using a new _brush_. Each brush is associated with a specific type of HTML tag, and can be passed arguments defining more detail of what you want to be rendered. So to write out a paragraph of text, you would use `html paragraph with: 'This is the text'`. This tells the canvas to start using the paragraph brush (which causes `<p>` to be output), then output the text passed as the argument, and finally to finish using the brush (which causes `</p>` to be output).

Many brushes can be passed multiple messages before they are finished, by chaining the messages together with `;` (this is called _cascading_ messages in Pharo). For example, a generic _heading_ exists which can be used to generate HTML headings at various levels, by passing it a `level:` message with an argument specifying the level of heading required:

```
html heading
   level: 3;
   with: 'A third level heading'.
```

This will produce the HTML:

```
<h3>A third level heading</h3>
```
You can cascade as many messages as you need to each brush object.

You can easily tell Seaside to nest tags by using Pharo blocks:

```
html paragraph: [
    html text: 'The next word is '.
    html strong: 'bold' ].
```


This will produce the HTML:

```
<p>The next word is <strong>bold</strong></p>
```


Note that we've used a very handy shortcut here: many of the brush methods have an equivalent method that can be called with a single argument so instead of typing `html paragraph with: 'text'` you need only type `html paragraph: 'text'`.

This is a very brief introduction that will allow you to begin to experiment with how these techniques can be combined into a larger piece of content, as you will see in the following sections. 


### First component: Hello World

Our first Seaside component will simply display _Hello world_. Begin by creating a package called 'SeasideBook-Hello' and then create the class `ScrapBook` as a subclass of `WAComponent` as shown below.

```
WAComponent << #ScrapBook
    package: 'SeasideBook-Hello'
```


When we use the term _component_ in this text we generally mean an instance of a subclass of `WAComponent`. For now, just think of subclasses of `WAComponent` as \`\`visual components''. When it is time for a component to be displayed, Seaside sends it the message  `WAComponent>>renderContentOn:` with a single argument (by convention called `html`) which is an instance of the class `WARenderCanvas` (the \`\`canvas''). Think of the canvas as the medium on which you will paint your component. It provides a transparent interface to XHTML which makes it easy to produce text, anchors, images etc., in a modular way (i.e., attached to each component of your application). To start, we just want to show a simple text message. Fortunately the canvas supports a `text:` message for just this purpose, which we can use as shown below.

!!note Important Note that all the classes in Seaside are prefixed with `WA` which acts as a namespace. Do not use this prefix for your components. `WA` is intended for Seaside framework classes.

```
ScrapBook >> renderContentOn: html
   html text: 'Hello world'
```


Great, we have a component but how do we get Seaside to serve it? For now, evaluate the following code in a workspace:

```
WAAdmin register: ScrapBook asApplicationAt: 'hello'
```


Now open your web browser and go to [http://localhost:8080/hello](http://localhost:8080/hello), and you should see something very like *@ref:helloworld@*.

![Hello World component. % width=100&anchor=ref:helloworld](figures/hello-world.png)

Seaside added XHTML markup for the skeletal structure of an XHTML document (`html`, `head` and `body` tags). OK, so what is happening here? Grossly simplified: When we request this URI, Seaside creates a new instance of our class for us and then sends it  `WAComponent>>renderContentOn:`. After being placed inside a skeleton XHTML document, the XHTML painted onto the canvas is then returned to the web browser to be displayed.

!!note Important Never invoke the method `renderContentOn:` directly, Seaside will do it for you.

You will never need to send your component the message `WAComponent>>renderContentOn:` since the Seaside framework takes care of that for you.  When it is time to paint your component, Seaside sends it `renderContentOn:`.  This is very similar to models used in most GUI frameworks where a component (or window) is told to paint itself whenever the windowing system deems necessary. Also, keep this in mind as you work with Seaside: a rendering method is just for displaying a component not changing its state.

!!note Important Your rendering method is just for painting the current state of your component, it shouldn't be concerned with changing that state.

### Fun with Seaside XHTML Canvas


Let's try making our `ScrapBook` component look a little more exciting. Redefine the method `renderContentOn:` as follows. Refresh your browser and you should see a situation similar to Figure *@ref:
*@ref:10images-horizontal@*).

```
ScrapBook >> renderContentOn: html
   html paragraph: 'Fun with Smalltalk and Seaside.'.
   html paragraph: [
        10 timesRepeat: [
            html image
                url: 'http://www.seaside.st/styles/logo-plain.png';
                width: 50.
            html horizontalRule ] ]
```


```
ScrapBook>>renderContentOn: html
    html paragraph: 'Fun with Smalltalk and Seaside.'.
    html paragraph: [
        10 timesRepeat: [
            html image
                url: 'http://www.seaside.st/styles/logo-plain.png';
                width: 50 ].
        html horizontalRule ]
```


![ScrapBook with vertical elements.%width=100&anchor=ref:10images-vertical](figures/10imagesHR.png)



![ScrapBook with horizontal elements.% width=100&anchor=ref:10images-horizontal](figures/10imagesHR2.png)


Using Seaside's canvas and brushes eliminates many of the errors that occur when manually manipulating tags.



### Rendering Objects


Let's take a moment to step back and review what we have learnt. Consider the following method:

```
ScrapBook >> renderContentOn: html
    html paragraph: 'A plain text paragraph.'.
    html paragraph: [
        html render: 'A paragraph with plain text followed by a line break. '.
        html break.
        html emphasis: 'Emphasized text '.
        html render: 'followed by a horizontal rule.'.
        html horizontalRule.
        html render: 'An image URI: '.
        html image url: 'http://www.seaside.st/styles/logo-plain.png' ]
```



There are four patterns that appear in this method.

1. `render: renderableObject`. This message adds the renderable object to the content of the component. It doesn't use any brushes, it just tells the object to render itself.
1. Message with zero or one argument. These create brushes. Just as a painter is able to use different tools to paint on a canvases, brushes represent the various tags we can use in XHTML, so `horizontalRule` will produce the tag `hr`. Some brushes take an argument such as `emphasis:` other don't. Section *@ref:rendering-components@* will cover this in depth. 
1. Composed messages. The expression `html image` creates an image brush, and then sends it a `url:` message to configure its attributes.
1. Block passed as arguments. Using a block (code delimited by `[` and `]`) allows us to say that the actions in the block are happening in the context of a given tag. In our example, the second paragraph takes an argument. It means that all the elements created within the block will be contained within the paragraph.


**About the `render:` message.** As you saw, we use the message `render:` to render objects. Modify the method `renderContentOn:` of your ScrapBook component as follows.

```
ScrapBook >> renderContentOn: html
    html paragraph: [
        html render: 'today: '.
        html render: Date today ]
```


Refresh your web browser, you should see a situation similar to Figure *@ref:date@*. The method`renderContentOn:` renders a string and the object representing the current date. It uses the method `render:`. Most of the time you will use the method `render:` to render objects or other components, see Chapter .


![Rendering object with the `render:` method.% width=100&anchor=ref:date](figures/date.png)


We use a block as the argument of the `paragraph:` because we want to specify that the string `'today '` and the textual representation of the current date are both within a paragraph. Seaside provides a shortcut for doing this. If you are sending only the message `render:` inside a block, just use the renderable object as a parameter instead of the block. The following two methods are equivalent and we suggest you to use the second form, see Figure *@ref:compact@*.

Two equivalent methods.

```
ScrapBook >> renderContentOn: html
    html paragraph: [ html render: 'today: ' ]
```


```
ScrapBook >> renderContentOn: html
    html paragraph: 'today: '
```


![Rendering object with the `render:` method.% width=100&anchor=ref:compact](figures/compact.png)


**About the method `text:`**. You may see some Seaside code using the message `WAHtmlCanvas>>text:` as follow.

```
renderContentOn: html
    html text: 'some string'.
    html text: Date today.
```


The method `text:` produces the string representation of an object, as would be done in an inspector. You can pass any object and by default its textual representation will be rendered. In Pharo and GemStone the method `text:` will call the method  `Object>>asString`. In any case, the string representation is generated by sending the message `Object>>printOn:` to that object. `html text: anObject` is an efficient short hand for the expression `html render: anObject asString` in Pharo.

**About the method `renderOn:`**. If you browse the implementation of  `WAHtmlCanvas>>render:` you will see that `render:` calls `renderOn:`. Do not conclude that you can send `renderOn:` in your `renderContentOn:` method.  `Object>>renderOn:` is an internal method which is part of a double dispatch between the canvas and an object it is rendering. Do not invoke it directly!


### Brush Structure

In the previous section you played with several brushes and painted a canvas with them. Now we will explain in detail how brushes work. A canvas provides a simple pattern for creating and using these brushes as shown in *@ref:brush-principle@*.

1. Tell the canvas what type of brush you are using.
1. Configure the brush, specifying any special options that it may use.
1. Render the contents of this brush. This is often done by passing an object such as a string or a block to `with:`. 


!!note It is not always necessary to send a brush the `with:` message. Do so only if you want to specify the contents of the body of the XHTML tag. Since this message causes the XHTML tag to be rendered, it should be sent _last_.

![Select brush, configure it, and render it.% width=100&anchor=ref:brushPrinciple](figures/brushPrinciple.pdf)


Here is an example:

```
html heading level: 1; with: 'Hello world'.
```


produces the following XHTML

```
<h1>Hello world</h1>
```


In this example

1. We first specify the brush (tag) we are using with `html heading`.
1. We then send that brush the `level: 1` message to indicate that this should be a level 1 heading.
1. We tell the brush the contents of the heading and cause it to be rendered using `with:`.


Here are some examples that show that it is not necessary to use `with:` if you do not specify the attributes of the brush.

**Just a brush**
```
html paragraph
```

_produces_
```
<p></p>
```


**A brush with implicit content**

```
html paragraph: 'foo'
```

_produces_ 
```
<p>foo</p>
```


**Setting the content explicitly**

```
html paragraph with: 'foo'
```

_produces_ 
```
<p>foo</p>
```


**Setting attributes directly**
```
html paragraph class: 'cool'; with: 'foo'
```

_produces_ 
```
<p class="cool">foo</p>
```


If no configuration of the brush is necessary, it is usually possible to specify it with a keyword parameter which becomes the contents of the tag. Thus the two following expressions are equivalent

```
html heading level: 1; with: 'Hello world'.
html heading: 'Hello World'.
```

since level 1 is the default level for a heading.

As you can see, there are two cases where you can write more compact code. These are summarized in *@ref:brush-principle-compact@*. There is a style checking tool called Slime that checks for such cases. Slime is explained in Chapter .


![Select brush, configure it, and render it. % width=90&anchor=ref:brush-principle-compact](figures/brushPrincipleCompact2.pdf)

The next section will show you what the other brushes are and how to find information about them within Seaside.


### Learning Canvas and Brush APIs

As we proceed and introduce new brushes, we will provide a table describing the parts of the Brush API that are essential for this book. Once you've used the table to find the brush to use for a given tag, you can read the source code for that brush class to find out how to use it. For example, if you want to produce a heading such as `<h1>Something great</h1>` you'll see that you should use the `WAHtmlCanvas>>heading` message which produces a brush instance of `WAHeadingTag` which has the following API:


| Methods in `WAHeadingTag` | Description |
| --- | --- |
| `level:` | Specify `anInteger` as the heading level for this brush. |
| `with:` | Render this heading tag with `anObject` as its body. |


To help you to find the correct brush, the brushes are presented from
the perspective of the HTML tags in the following table:


>HTML         Factory Selector    Brush Class
>
>a            anchor              WAAnchorTag
>a            map                 WAImageMapTag
>a            popupAnchor         WAPopupAnchorTag
>abbr         abbreviated         WAGenericTag
>acronym      acronym             WAGenericTag
>address      address             WAGenericTag
>big          big                 WAGenericTag
>blockquote   blockquote          WAGenericTag
>br           break               WABreakTag
>button       button              WAButtonTag
>caption      tableCaption        WAGenericTag
>cite         citation            WAGenericTag
>code         code                WAGenericTag
>col          tableColumn         WATableColumnTag
>colgroup     tableColumnGroup    WATableColumnGroupTag
>dd           definitionData      WAGenericTag
>del          deleted             WAEditTag
>dfn          definition          WAGenericTag
>div          div                 WADivTag
>dl           definitionList      WAGenericTag
>dt           definitionTerm      WAGenericTag
>em           emphasis            WAGenericTag
>fieldset     fieldSet            WAFieldSetTag
>form         form                WAGenericTag
>h1           heading             WAHeadingTag
>hr           horizontalRule      WAHorizontalRuleTag
>iframe       iframe              WAIframeTag
>img          image               WAImageTag
>input        cancelButton        WACancelButtonTag
>input        checkbox            WACheckboxTag
>input        fileUpload          WAFileUploadTag
>input        hiddenInput         WAHiddenInputTag
>input        imageButton         WAImageButtonTag
>input        passwordInput       WAPasswordInputTag
>input        radioButton         WARadioButtonTag
>input        submitButton        WASubmitButtonTag
>input        textInput           WATextInputTag
>ins          inserted            WAEditTag
>kbd          keyboard            WAGenericTag
>label        label               WALabelTag
>legend       legend              WAGenericTag
>li           listItem            WAGenericTag
>object       object              WAObjectTag
>ol           orderedList         WAOrderedListTag
>optgroup     optionGroup         WAOptionGroupTag
>option       option              WAOptionTag
>p            paragraph           WAGenericTag
>param        parameter           WAParameterTag
>pre          preformatted        WAGenericTag
>q            quote               WAGenericTag
>rb           rubyBase            WAGenericTag
>rbc          rubyBaseContainer   WAGenericTag
>rp           rubyParentheses     WAGenericTag
>rt           rubyText            WARubyTextTag
>rtc          rubyTextContainer   WAGenericTag
>ruby         ruby                WAGenericTag
>samp         sample              WAGenericTag
>script       script              WAScriptTag
>select       multiSelect         WAMultiSelectTag
>select       select              WASelectTag
>small        small               WAGenericTag
>span         span                WAGenericTag
>strong       strong              WAGenericTag
>sub          subscript           WAGenericTag
>sup          superscript         WAGenericTag
>table        table               WATableTag
>tag:         tag:                WAGenericTag
>tbody        tableBody           WAGenericTag
>td           tableData           WATableDataTag
>textarea     textArea            WATextAreaTag
>tfoot        tableFoot           WAGenericTag
>th           tableHeading        WATableHeadingTag
>thead        tableHead           WAGenericTag
>tr           tableRow            WAGenericTag
>tt           teletype            WAGenericTag
>ul           unorderedList       WAUnorderedListTag
>var          variable            WAGenericTag


!!note Pharo typically encourages explicit naming and avoids abbreviations -- the few seconds per day you save by typing an abbreviated method or variable name may often come back much later to haunt you or someone else reading your code as minutes or even hours spent trying to debug code with poor readability.


This book is not a complete Seaside reference. Once you're done reading it you will want to discover new brushes and brush options yourself. Let's take a few moments to describe how you would do that.

Suppose you know a specific XHTML tag you want to use and need to find the appropriate brush method. Some brush method names are the same as the corresponding XHTML tag name. For example you create a `div` tag using the `WAHtmlCanvas>>div` brush method. In other cases the brush name is the long form of the equivalent XHTML tag (`WAHtmlCanvas>>paragraph` creates a `p` tag, `WAHtmlCanvas>>unorderedList` creates a `ul` tag etc). This choice makes your methods a lot more readable than if the XHTML tags were used everywhere. Compare the following two code fragments.

```
This is not working code"
html p with: 'Hello world.'.
html ol with: [
   html li: 'Item 1'.
   html li: 'Item 2' ].
```


```
Working version"
html paragraph with: 'Hello world.'.
html orderedList with: [
   html listItem: 'Item 1'.
   html listItem: 'Item 2' ].
```


If you can't guess the brush method name just open a class browser on the canvas class `WARenderCanvas`. Keep in mind that the method you're interested in may be in a superclass, see the hierarchy below.

Normally, the brush configuration methods that set tag attributes, use the same name as the attribute. So, for example, to set the `altText` attribute for an `IMG` (image) tag you'd send the `image` brush the `WAImageTag>>altText:` message. If you don't know the tag attribute you need to open a class browser on the specific brush class. Once again, keep in mind that the method you're interested in may be in a superclass. In addition to tag attributes, many of the Seaside brushes support convenience methods and common Javascript hacks (like setting the focus of an input field). The best way to find these is to use your  tools.

When you first begin using Seaside your canvas and brush vocabulary will be limited and it might take you a few minutes to find what you're looking for. After a while you'll discover that there is a significant shared API (implemented in the abstract superclasses) and that you are already familiar with many of the brushes. Also helpful is the autocompletion mechanism in the development environment.

> WABrush
>    WACompound
>       WADateInput
>       WATimeInput
>    WATagBrush
>       WAAnchorTag
>          WAImageMapTag
>          WAPopupAnchorTag
>       WAAudioTag
>       WABasicFormTag
>          WAFormTag
>       WABreakTag
>       WACanvasTag
>       WACollectionTag
>          WADatalistTag
>          WAListTag
>             WAOrderedListTag
>             WAUnorderedListTag
>          WASelectTag
>             WAMultiSelectTag
>       WACommandTag
>       WADetailsTag
>       WADivTag
>       WAEventSourceTag
>       WAFieldSetTag
>       WAFormInputTag
>          WAAbstractTextAreaTag
>             WAColorInputTag
>             WAEmailInputTag
>             WASearchInputTag
>             WASteppedTag
>                WAClosedRangeTag
>                   WANumberInputTag
>                   WARangeInputTag
>                   WATimeInputTag
>                WADateInputTag
>                WADateTimeInputTag
>                WADateTimeLocalInputTag
>                WAMonthInputTag
>                WAWeekInputTag
>             WATelephoneInputTag
>             WATextAreaTag
>             WATextInputTag
>                WAPasswordInputTag
>             WAUrlInputTag
>          WAButtonTag
>          WACheckboxTag
>          WAFileUploadTag
>          WAHiddenInputTag
>          WARadioButtonTag
>          WASubmitButtonTag
>             WACancelButtonTag
>             WAImageButtonTag
>       WAGenericTag
>          WAEditTag
>       WAHeadingTag
>       WAHorizontalRuleTag
>       WAIframeTag
>       WAImageTag
>       WAKeyGeneratorTag
>       WALabelTag
>       WAMenuTag
>       WAMeterTag
>       WAObjectTag
>       WAOptionGroupTag
>       WAOptionTag
>       WAParameterTag
>       WAProgressTag
>       WARubyTextTag
>       WAScriptTag
>       WASourceTag
>       WATableCellTag
>          WATableColumnGroupTag
>             WATableColumnTag
>          WATableDataTag
>             WATableHeadingTag
>       WATableTag
>       WATimeTag
>       WAVideoTag

### Rendering Lists and Tables


We will modify our `ScrapBook` to display the site contents using a list. We want to use an ordered list so we'll ask the canvas for an `WAHtmlCanvas>>orderedList` brush, which answers with an instance of  `WAListTag`. Inside the body of that tag we want a collection of list item tags (`li`) which we get with the canvas'  `WAHtmlCanvas>>listItem:` method. We use the short form so we don't have to use `with:` for each list item.

```
ScrapBook >> renderContentOn: html
    html heading level: 1; with: 'Hello world'.
    html paragraph: 'Welcome to my Seaside web site.  In the
        future you will find all sorts of applications here
        such as:'.
    html orderedList with: [
        html listItem: 'Calendars'.
        html listItem: 'Todo lists'.
        html listItem: 'Shopping carts'.
        html listItem: 'And lots more...' ]
```


Let's use our earlier suggestions to write this code more succinctly. We'll use `heading:` instead of `html heading level1`, and we'll use the ordered list shortcut `WAHtmlCanvas>>orderedList:`. We can use this last shortcut since we aren't configuring the ordered list tag.

```
ScrapBook >> renderContentOn: html
    html heading: 'Hello world'.
    html paragraph: 'Welcome to my Seaside web site.  In the
        future you will find all sorts of applications here
        such as:'.
    html orderedList: [
        html listItem: 'Calendars'.
        html listItem: 'Todo lists'.
        html listItem: 'Shopping carts'.
        html listItem: 'And lots more...' ]
```


Open this component in your web browser and you should see something similar to *@ref:hello-world-list@*.

![A list of items. % width=100&anchor=ref:hello-world-list](figures/hello-world-list.png)

As good Pharoers following the DRY (Don't Repeat Yourself) principle, we can refactor this method to avoid an explicit enumeration as follows. This demonstrates the power of having a programmatic way to specify the component contents.

```
ScrapBook >> items
    ^ #('Calendars' 'Todo lists' 'Shopping carts' 'And lots more...')
```


```
ScrapBook >> renderContentOn: html
    html heading:  'Hello world'.
    html paragraph: 'Welcome to my Seaside web site.  In the
        future you will find all sorts of applications here
        such as:'.
    html orderedList: [
        self items do: [ :each | html listItem: each ] ]
```


Let's create a table of expected delivery dates.  We suggest you perform a similar refactoring of the following method which illustrates `tableRow:` and `tableData:`.

```
ScrapBook >> renderContentOn: html
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
```


Notice that we generate table text entries in a fashion that is very similar to what we did in the list example. Note also that we used  `WAHtmlCanvas>>tableHeading:` to designate a cell that represents a row header.

![A table of items. % width=100&anchor=ref:hello-world-table](figures/hello-world-table.png)




### Style Sheets

The visual design of most modern web applications is controlled with Cascading Style Sheets (CSS). Seaside provides a simple method to add a style sheet to a component.  Override the `WAComponent>>style` method in your component to return a CSS style sheet as a string, for example as follows:

```
ScrapBook>>style
    ^ 'h1 { text-align: center; }'
```



Now, refresh your browser and you should see a centered \`\`Hello world''. Bring up the halo on this component and click the pencil. Notice that you can edit the component's style method here. If you save your changes then the component's style method will be updated.

Use of the  `WAComponent>>style` method is discouraged for anything but writing sample code and rapid development while experimenting with component styles.  There are several reasons for this:

- It places too much emphasis on presentation at the component level and makes it difficult to reuse components in applications with a different \`\`look''.  The same motivation for having XHTML separate from CSS applies to separating style documents from components.
- Having many small `style` methods increases the load time of your pages. Each `WAComponent>>style` method is loaded as a separate document.
- Having a `style` method at the component level may give you the false impression that this style only applies to that component. In fact, CSS style sheets are loaded in the XHTML `head` section of the document and apply to the entire document, which means you have to be careful to get your CSS selectors right. It can be very difficult to track down an errant style selector when it is hidden in a component.
- It makes it more difficult to work with other non-Smalltalk developers to style your documents.


As you work through this book use the `style` method but keep in mind that a more complex application would use an external style sheet as described in Chapter *@ref:/book/fundamentals/css@* or file library style sheet as described in Chapter *@ref:/book/in-action/serving-files@*.

If you've used CSS regularly then you're already familiar with using `div` (block elements) and `span` (inline elements) with the `class` attribute to help you select specific parts of a document to style.

Here's how one would, for example, give a red background to error text:


```
ScrapBook >> renderContentOn: html
    " code from previous example "
    html span
        class: 'error';
        with: 'This site does not work yet'
```


```
ScrapBook >> style
    ^ 'h1 { text-align: center; }
   span.error { background-color: red; }'
```


This book is not about XHTML or CSS but *@ref:/book/fundamentals/css@* provides a quick overview of CSS. We do have a few suggestions:

- Use `span` and `div` with CSS classes to mark content generated by your components. Quite often, components render themselves entirely within a `div` whose CSS class is the name of the component's Smalltalk class. This makes it easier for CSS developers to locate the XHTML elements that they want to style. Come up with conventions that work for you and stick to them.
- Your component's `renderContentOn:` method should be simple. Do not include style information, otherwise it will be difficult to customize the way in which your component is displayed.
- Your component's `renderContentOn:` method should be short. If your method gets longer than a couple of lines create intention-revealing helper methods following the naming pattern `renderSomethingOn:`. This is especially useful if you start to use conditional statements and loops. This technique will also allow you to override certain aspects of the rendering process in subclasses of your component.
- Use component `style` method only when the style elements are very specific to that component. Otherwise use style libraries as discussed later in Chapter .
- Try to use style sheets to structure your document's overall presentation, rather than XHTML tables. There are many good CSS references which show you how to lay out pages.


### Summary

We have shown that you can specify the visual aspect of your component using the Seaside brush API.  These brushes will make it easy to produce valid XHTML code. We have also demonstrated that you can use all the power of Smalltalk to specify your content, and that all the visual aspects of your application can be specified using CSS.

The method `renderContentOn:` is automatically invoked by Seaside. It allows you to specify the output of the application. Brushes are used to paint XHTML tags onto a canvas object. Blocks are used to create a scope within tags. For example:

```
html paragraph with: [ html render: 'today' ]
```


renders the string `'today'` within `<p>today</p>`. A brush is an expression of the form:

```
html brush 
    attributes1; 
    attributes2;
    with: anObject
```

Code can be made more compact in two ways.

1. When the nested expression is a single object you can avoid blocks. The expression `html paragraph with: [ html render: 'today' ]` is equivalent to `html paragraph with: 'today'`.
1. When you don't need to configure the brush's attributes, you don't need to use `with:`. The expression `html paragraph with: 'today'` is equivalent to `html paragraph: 'today'`.


In conclusion the following three code snippets create exactly the same output. For readability and to avoid having to type unnecessary code, we usually choose the shortest version possible:

```
html paragraph with: [ html render: 'today' ].
html paragraph with: 'today'.
html paragraph: 'today'.
```
