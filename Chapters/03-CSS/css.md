## CSS in a Nutshell
@cha:css

In this chapter we present CSS in a nutshell and show how Seaside helps you to use CSS in your applications. The goal of the chapter is not to replace CSS tutorials, many of which can be found on the web. Rather, the goal is to establish some basic principles and show how Seaside facilitates the decoupling of information and its visual presentation in web browsers. A clear separation between the page components and their rendering is really central to Seaside. Sometimes this frustrates newcomers because Seaside does not use template mechanisms for rendering. However, the Seaside approach allows the clear separation between the responsibilities of the web designer and the web developer. The developer is not responsible for rendering and layout of the application, this is the job of the web designer.

The idea behind CSS is to decouple the presentation from the document itself. The tags in a document are interpreted using a CSS (Cascading Style Sheet) which defines the layout and style of the rendered document. In the context of Seaside, the component rendering methods generate XHTML and the CSS associated with the application specifies how such components should be displayed and placed on the page.

### CSS Principles


Basically, a CSS specification contains a set of rules. A rule is a description of a stylistic aspect of one or more elements. A rule is composed of a _selector_ and a _declaration_. In the following rule `h1` is a selector which specifies that the following declaration `color: red` will be applied to all the first-level headings, see Figure *@cssstru@*. The rule has the effect that all the first level headings will be red.

```
h1 { color: red; }
```


A declaration is composed of two parts separated by a colon and ended with a semicolon. The first part is the _property_ being specified, and the second is the _value_ assigned to that property.

We can group multiple CSS selectors to share the same property. The following rules:

```
h1 { color: red; }
h2 { color: red; }
h3 { color: red; }
```


are equivalent to this single rule:

```
h1, h2, h3 { color: red; }
```


Similarly, it is possible to assign several values to a single selector. For example, the following rule changes the alignment and color of headings.

```
h1 { color: red; text-align: center; }
```



![Essential CSS structural elements. %width=80&anchor=cssstru](figures/CSSStructure.pdf )



Most declarations are inherited from higher levels of your document tree. CSS property values assigned to one element are transferred down the tree to its descendants. For example, a property value set in the body of a document will be propagated to all its children, which may then redefine the value locally. This is true for color, font, etc., but not for other properties like width, height, and positioning, for which inheriting would not make sense.

While CSS declarations can be embedded in your document using a `style` tag, it is a good practice to have your CSS in a separate file. Then, if you were writing your XHTML by hand, you would add a reference to your CSS file in the head section of your document as follows.

```
<head>
<link rel="stylesheet" type="text/css" href="mystyle.css" />
</head>
```


It's not necessary to write this in Seaside though. The CSS will be served by using either the Seaside file library or with Apache, see Chapter *@cha:serving-files@*. For rapid prototyping, you can define the CSS of a component by overriding its  `WAComponent>>style` method.

### CSS Selectors


CSS allows you to select individual elements of an XHTML document, or groups of elements that share some property. Let's look at the different kind of selectors and how they can be used.

#### Tag Selector


The tag selector applies to specific XHTML tags, as we saw in the previous examples. The selector consists simply of the tag name as it appears in the XHTML source, but without the angle brackets. The following example removes the underlining from all anchor elements and changes the base font-size of the text within the page to 20 points. The `body` tag is one of the top-level tags automatically created by Seaside enclosing the whole page.

```
a { text-decoration: none; }
body { font-size: 20pt; }
```


#### Class Selector 

The class selector is by far the most often used CSS selector. It starts with a period and usually defines a visual property that can be added to XHTML tags. The following CSS fragment defines the `center` and the `highlight` classes.

```
.center { text-align: center; }
.highlight { color: yellow; }
```


To use class selectors, simply set the `WAGenericTag>>class:` attribute of the tag you want to change. Here we associate the class selector `.center` to a given div element and `.highlight` to a given paragraph.

```
html div
    class: 'center';
    with: 'Seaside is cool'.
html paragraph
    class: 'highlight';
    with: 'Highlighted text'
```


The generated XHTML code looks like this:

```
<div class="center">Seaside is cool</div>
<p class="highlight">Highlighted text</p>
```


Multiple classes can be added to a single XHTML tag. So the following code will display a single text that is centered and highlighted:

```
html div
    class: 'center';
    class: 'highlight';
    with: 'Centered and Highlighted'
```


The generated XHTML code looks like this:

```
<div class="center highlight">Centered and Highlighted</div>
```


Often CSS classes are used to conditionally highlight certain elements of a page depending on the state of the application. Seaside provides the convenience method 
 `WAGenericTag>>class:if:` to make this as concise as possible. The following code snippet creates ten `div` tags and adds the CSS class `even` only if the condition `index even` evaluates to `true`. This is shorter than to manually build an `ifTrue:ifFalse:` construct.

```
1 to: 10 do: [ :index |
    html div
        class: 'even' if: index even;
        with: index ]
```


#### Pseudo Class Selector


Pseudo classes are similar to CSS classes, but they don't appear in the XHTML source code. They are automatically applied to elements by the web browser, if certain conditions apply. These conditions can be related to the content, or to the actions that the user is carrying out while viewing the page. To distinguish pseudo classes from normal CSS selectors they all start with a colon.

```
:focus { background-color: yellow; }
:hover { font-weight: bold; }
```


The first rule specifies that elements \(typically input fields of a form\) get a yellow background, if they have focus. The second rule specifies that all elements will appear in bold while the mouse cursor hovers over them.

The following table gives a brief overview of pseudo classes supported by most of today's browsers:


| `:active` | Matches an activated element, e.g. a link that is clicked. |
| `:first-child` | Matches the first child of another element. |
| `:first-letter` | Matches the first character within an element. |
| `:first-line` | Matches the first line of text within an element. |
| `:focus` | Matches an element that has the focus. |
| `:hover` | Matches an element below the mouse pointer. |
| `:link` | Matches an unvisited link. |
| `:visited` | Matches a visited link. |

#### Reference or ID Selector


A reference or ID is the name of _a particular_ XHTML element on the page. Thus, the given style will affect only the element with the unique ID `error` \(if defined\). The ID selector is indicated by prefixing the ID with a `#` character:

```
#error { background-color: red; }
```


To create a tag with the given ID use the following Seaside code:

```
html div
    id: 'error';
    with: 'Some error message'
```


The generated XHTML code looks like this:

```
<div id="error">Some error message</div>
```


There are a couple of issues to be aware of when using IDs in your XHTML. IDs have to be unique on a XHTML page. If you use the same ID multiple times, some web browsers may not render your page as you expect, or may even refuse to render it at all. Furthermore some Javascript libraries dynamically apply their own IDs to identify page elements and these may override your carefully chosen IDs, causing your styling to fail in mysterious ways. 

!!important So, to avoid invalid XHTML and conflicts with JavaScript code, do not use IDs for styling. Exclusively use CSS classes for styling, even if the particular style is used only once.

#### Composed Selectors 


CSS selectors can be composed in various ways to give you more control over which page elements they select.

**Conjunction.** If you concatenate selectors without any spaces, it means that the matching element must satisfy all given selectors. This is generally used to refine the application of class or pseudo-class selectors. For example, we can write `div.error` in the style sheet and this will affect only the tags that also have the class `error`.

 Similarly `a:hover` will only apply when the user moves their mouse over anchor tags. It might be tempting to specify multiple classes with `.error.highligth`. Even though this is part of the CSS standard, it does not work in older versions of Internet Explorer.
 
**Descendant.**  Another possibility is to combine selectors with a space. This finds all elements that match the first term, then searches within each for descendant elements that match the second term \(and so on\). For example, `div .error` selects all the elements _within_ a tag that have the CSS class `error`. Elements that have the class `error` but no tag as one of its parents,
are not affected.

**Child.** Yet another possibility is to combine two selectors with `>`. For example,`div > .error`. This selects all the tags with the class `error` that are direct children of a tag. Again this does not work in older versions of Internet
Explorer.

There are a few more selectors available in modern web browsers to allow other criteria such as matching adjacent siblings. Since these selectors are not widely implemented in web browsers yet, we don’t discuss them here.