## Embedding Components

@cha:embedding

Building reusable components and frameworks is the goal of all developers in almost all parts of their applications. The dearth of truly reusable (canned) component libraries for most of the existing web development frameworks is a good indication that this is difficult to do.

Seaside is among the few frameworks poised to change this. It has a solid component model giving one all of the mechanisms necessary to develop well encapsulated components and application development frameworks. We have seen in  that components can be sequenced. In this chapter we show how to embed one component inside another component. In *@ref:decorations@*, we will see how to decorate a component to add functionality or change its appearance and as such reuse behavior. With a good component model, the possibility to display components and create new ones by reusing existing ones, writing Seaside applications is very similar to writing GUI applications.

We will start by writing an application which embeds one component, then refactor it into an application built out of two components. Finally we will discuss reuse and show how component decoration support this. We will show how components can be plugged together using events to maximize reuse.

### Principle: Component Children


When we want to display several components on the same page, we can embed the components into each other. Usually a Seaside application consists of a main component which is usually called the root component. All the child components of the root component and their recursive children form a tree of nested components.

Child components are no different to other components or the root component. Note that the component tree of an application might change during the lifetime of a session. Through user interaction new components can be shown and old ones can be hidden.

Seaside requires that each component knows and declares all its visible child components using the method `WAComponent>>children`. This allows Seaside to know in advance what components will be visible when building the HTML and configure and trigger some event on these components before the actual rendering happens.

Note that Seaside does not require children to know their parents and the framework also does not provide this information. When instantiating the components such a link can be easily established, but we do not suggest doing so as it would introduce strong coupling between the child component and its parent. For example, it would no longer be easily possible to use the same component in a different context.

Here are the steps that should be performed to embed components within another.

1. The parent component initializes the child components in the method `initialize`.
1. The parent component defines a method named `children` that returns all its direct child component instances, regardless of how and where they are store.
1. The parent component renders its child components in the method `renderContentOn:` using `html render: aComponent`.



### Example: Embedding an Editor


We will build a new variation of our contact list manager. What we'd like to do is adapt our contact manager so that we see the item editor on the same page as the contact list item. That is, we want to embed the editor on the same page as the address list itself. While we could adapt the previous component to embed a component, we prefer to define a new component from scratch.

![Embedding a ContactView into another component. %width=80&anchor=ref:listAndItem](figures/iAddress-ListAndEditor.png )


We already have a working editor component so let's just add it to a new `IAddress` component. That is, we're going to embed the `ContactView` component within the `IAddress` component.

**Create the class.** First we create the class of the new component `IAddress`.

```
WAComponent << #IAddress
    slots: { #editor}; 
    package: 'iAddress'
```


We add an instance variable, `editor`, to the class `IAddress` which is a reference to the editor that we will embed within our `IAddress` component. It is not always necessary to maintain a reference to an embedded component: we could also create the component on the fly (as soon as it is returned as part of the component's children). In our case, since the elements of our application are stateful objects, it is better to reuse components, taking advantage of the fact that they can store state, rather than recreate them. We will revisit this issue in *@ref:coupling@*.

**Initialize instances.** The `initialize` method creates the editor and gives it a default contact to edit. We can then reuse the editor to edit other contacts, avoiding the need to create a new editor every time we want to edit something.

```
IAddress>>contacts
    ^ Contact contacts
```


```
IAddress >> initialize
    super initialize.
    editor := ContactView new.
    editor contact: self contacts first
```


!!note Important **Specifying the component's children.** Any component that uses embedded children components should make Seaside aware of this by returning an array of those components. This is necessary because Seaside needs to be able to figure out what components are embedded within your component; Seaside needs to process all the callbacks for all of the components that may be displayed, before it starts any rendering of those components. Unless you add the children components to the parent's `children` method, the first Seaside will know about your children components is when you reference them in your rendering methods.


**Specify the children.** Here is how to define the method `children` which returns an array containing the editor that is accessible via the instance variable `editor`.

```
IAddress >> children
    ^ Array with: editor
```


**Specify some actions.** Now define methods to create, add, remove and edit a contact.

```
IAddress >> addContact: aContact
    Contact addContact: aContact
```


```
IAddress >> askAndCreateContact
    | name emailAddress |
    name := self request: 'Name'.
    emailAddress := self request: 'Email address'.
    self addContact: (Contact name: name emailAddress: emailAddress)
```



```
IAddress >> editContact: aContact
    editor contact: aContact
```


```
IAddress >> removeContact: aContact
    (self confirm: 'Are you sure that you want to remove this contact?')
        ifTrue: [ Contact removeContact: aContact ]
```


![Embedding a ContactView into another component with Halos. %width=80&anchor=ref:ref:list-editor-with-halo](figures/iAddress-ListAndEditorWithHalos.png )


Embedding a ContactView into another component with Halos.

**Some rendering methods.** We use a table to render the current contact list and let the user edit a contact by clicking on the name link.


```
IAddress >> renderContentOn: html
    html form: [
        self renderTitleOn: html.
        self renderBarOn: html.
        html table: [ 
            html tableRow: [
                html tableHeading: 'Name'.
                html tableHeading: 'Address' ].
            self renderDatabaseRowsOn: html ].
        html horizontalRule.
        html render: editor ]
```


```
IAddress >> renderTitleOn: html
    html heading level: 2; with: 'iAddress'
```


```
IAddress >> renderBarOn: html
   html anchor
       callback: [ self askAndCreateContact ]; 
       with: 'Add contact'
```


```
IAddress >> renderDatabaseRowsOn: html
    self contacts do: [ :contact |
        html tableRow: [ self renderContact: contact on: html ] ]
```


```
IAddress >> renderContact: aContact on: html
    html tableData: [
        html anchor
            callback: [ self editContact: aContact ];
            with: aContact name].
    html tableData: aContact emailAddress.
    html tableData: [
        html anchor
            callback: [ self removeContact: aContact ];
            with: '--' ]
```


Register the application as "iAddress" and try it out. Make sure that the editor is doing its job. Activate the halos. You'll notice that there is a separate embedded halo around the editor component, see *@ref:list-editor-with-halo@*. It is very helpful to inspect the state of a component in a running application (or view the rendered HTML.)

!!note Our simple implementation of `IAddress>>editContact:` will save changes even when you press _cancel_. See  to understand how you can change this.


### Components All The Way Down


The changes to your code in this section are presented purely to help you explore the embedding of components: they are not an example of good UI design and are not required to progress with the following sections.

Let's define a component that manages our list of contacts using components all the way down. Figure *@ref:pluggable-halos@* shows that we will build our application out of two components: `PluggableContactListView` and `ContactView`. In addition, `PluggableContactListView` will be composed of several `ReadOnlyOneLinerContactView` components. We really get a tree of components. This exercise shows that a component should be designed to be pluggable. It also shows how to plug components together.

% +pluggable-halos|width=75%+
![With components all the way down. %width=80&anchor=ref:pluggable-halos](figures/PluggableHalos.png )

PluggableHalos

**A Minimal Contact Viewer.** We would like to have a compact contact viewer. First, we will subclass the `ContactView` class to create the class `ReadOnlyOneLinerContactView`. This class has an instance variable `parent` which will hold a reference to the component that will contain it, since it should know how to invoke the contact editor.

```
ContactView << #ReadOnlyOneLinerContactView
    slots: { #parent}; 
    package: 'iAddress'
```


```
ReadOnlyOneLinerContactView >> parent: aParent
    parent := aParent
```


When the user clicks on the contact name, we want the associated user object to pass itself to the parent for editing. A similar action should occur when removing a contact from the database. Note that this component does not include a form. This is because only one form should be present on a page at any time, so a component is much more reusable if it does not define a form.

```
ReadOnlyOneLinerContactView >> renderContentOn: html
        html anchor
            callback: [parent editContact: self contact];
            with: self contact name.
        html space.
        html text: self contact emailAddress.
        html space.
        html anchor
            callback: [parent removeContact: self contact];
            with: '--'.
        html break.
```



**Class Definition.** Now we define the class `PluggableContactListView`. Since this component will embed all the contact viewer components, we add an instance variable `contactViewers`, that will refer to them. We also define an instance variable to refer to the editor that will show the detailed information of the currently selected contact.

```
ContactListView << #PluggableContactListView
    slots: { #contactViewers . #editor};
    package: 'iAddress'
```


We will use an identity dictionary to keep track of the contact viewer associated with each contact. Initializing our top level component consists of first creating the editor and then creating the viewers for the existing contacts.

```
PluggableContactListView >> contacts
    ^ Contact contacts
```


```
PluggableContactListView >> initialize
    super initialize.
    editor := ContactView new.
    contactViewers := IdentityDictionary new.
    self contacts do: [ :each | self addContactViewerFor: each ]
```


```
PluggableContactListView >> addContactViewerFor: aContact
    contactViewers
        at: aContact
        put: (ReadOnlyOneLinerContactView new
            contact: aContact;
            parent: self; " <-- added "
            yourself)
```


```
PluggableContactListView >> askAndCreateContact
     | name emailAddress |
     name := self request: 'Name'.
     emailAddress := self request: 'Email address'.
     self addContact: (Contact name: name emailAddress: emailAddress)
```


```
PluggableContactListView >> editor: anEditor
    editor := anEditor
```


**Children accessing and rendering.** We have now to specify that the contact viewers are embedded within the `PluggableContactListView` and how to render them.

```
PluggableContactListView >> children
    ^ contactViewers values
```


```
PluggableContactListView >> renderContentOn: html
    contactViewers values
        do: [ :each | html render: each. html break ]
```


We define a couple of methods to manage contacts.

```
PluggableContactListView >> addContact: aContact
    Contact addContact: aContact.
    self addContactViewerFor: aContact
```


```
PluggableContactListView>>removeContact: aContact
    contactViewers removeKey: aContact.
    Contact removeContact: aContact
```


**Plugging everything together.** Now we are ready to define a new version of `IAddress`. We simply subclass `IAddress` and pay attention to the fact that the list is now a component. So we initialize it, add it as part of the children of the component and render it.

```
IAddress << #IAddressTwoComponents
    slots: { #list };
    package: 'iAddress'
```


We pass the list component to the editor which has already been initialized in `IAddress`. We must also invoke the list's `askAndCreateContact` method, since it is the list that manages the creation of contacts. The rendering of the component includes a form in which the other components are embedded.

```
IAddressTwoComponents >> initialize
    super initialize.
    list := PluggableContactListView new.
    list editor: editor  " <-- added "
```


```
IAddressTwoComponents>>children
    ^ super children , (Array with: list)
```


```
IAddressTwoComponents>>renderBarOn: html
    html anchor
        callback: [ list askAndCreateContact ]; 
        with: 'Add contact'
```


```
IAddressTwoComponents >> renderContentOn: html
    html form: [
        self renderTitleOn: html.
        self renderBarOn: html.
        html break.
        html render: list.
        html horizontalRule.
        html render: editor ]
```


Note of course that embedding such an editor under the list of contacts is not a really good UI design. We just use it as a pretext to illustrate component embedding.

### Intercepting a Subcomponent’s Answer



Components may be designed to support both standalone and embedded use. Such components often produce answers (send `self answer:`) in response to user actions. When the component is standalone the answer is returned to the caller, but if the component is embedded the answer is ignored unless the parent component arranges to intercept it. In our example application the editor provides an answer when the user presses the ''Save'' button (i.e. in `ContactView>>save`) but this answer is ignored. It is easy to change our application to make use of this information; let's say we want to give the user confirmation that their data was saved. To accomplish this, change `IAddress>>initialize` and add the  `WAComponent>>onAnswer:` behaviour:

```
IAddress >> initialize
    super initialize.
    editor := ContactView new.
    editor contact: self contacts first.
    editor onAnswer: [ :answer | self inform: 'Saved' ]   " <-- added "
```


Now restart your application (press ''New Session'') and try it out. When you press the save button in the editor you should get a dialog tersely notifying you that your data is saved. Note that the component answer is passed into the block (although we didn't use it in this example). 

The `onAnswer:` method is an important protocol for handling components and their answer.


### A Word about Reuse


Suppose you wanted a component that shows only the name and email of our `ContactView` component. There are no special facilities in Seaside for doing this, so you may be tempted to use template methods and specialized hooks in the subclasses. This may lead to the definition of empty methods in subclasses and may force you to define as many subclasses as situations you want to cover, for example, if you want to create a `SimpleContactView` and a `ReadyonlyContactView`. See Figure *@ref:readonly@* below.

An alternative approach is to build more advanced components using the messages `perform:` or `perform:with:` with a list of method selectors to be sent:

```
setOutlineForm
    "should be called during action phase"
    methods :=  #(renderNameOn: renderEmailOn:)
```


```
renderContentOn: html
   methods do: [ :each | self perform: each with: html ] 
```


You can also define a component whose rendering depends on whether it is embedded. Here is an example where the rendering method does not wrap its content in a form tag when the component is in embedded mode (i.e., when it would expect its parent to have already created a form in which to embed this component).  A better way of doing this would be to use a `FormDecorator` as shown in XXX.

```
EmbeddableFormComponent>>renderContent: html body: aBlock
    self embedded
        ifTrue: [ aBlock value ]
        ifFalse: [ html form: aBlock ]
```



```
EmbeddableFormComponent>>renderContentOn: html
    self renderContent: html body: [
        "lots of form elements get rendered here"
        self renderSaveOn: html ]
```


This would then be embedded by another component using code like:

```
ContainingComponent >> renderContentOn: html
    html form: [
        "some form elements here, followed by our embeddable Component:"
        html render: (EmbeddableFormComponent new beEmbedded; yourself) ]
```



If you need more sophisticated dynamic control over the rendering of your component, you may want to use _Magritte_ with Seaside. Magritte is a framework that allows you to define _descriptions_ for your domain objects. It then uses these descriptions to perform automatic actions (loading, saving, generating SQL...). The Magritte/Seaside integration allows one to automatically generate forms and Seaside components from domain object described with Magritte descriptions, see XXXX. Magritte also offers ways to construct different views on the same objects and so the possibility to create multiple varieties of components: either by selecting a subset of fields to display, or by offering read-only or editable components. As such, it is an extremely useful addition to plain Seaside.


![A readonly view of the ContactView. %width=80&anchor=ref:readonly](figures/readonlyContactView.png )



### Decorations
@ref:decorations

Sometimes, we would like to reuse a component while adding something to it, such as an information message or extra buttons. Seaside has facilities for doing this. In Seaside parlance, this is called ''decorating'' a component. Note that this is _not_ implemented using the design pattern of the same name, but rather as a _Chain of Responsibility_. This means that decorations form a chain of special components into which your component is inserted and that a given message passes through the chain of decorators.

Decorations can be added to any component by calling `WAComponent>>addDecoration:`. Decorations are used to change the behavior or the look of the decorated component.

A component decoration is static in the sense that it should not change after the component has been rendered. Thus, a decoration should be attached to a component either just after it (the decorator) is created, or just before the component is passed as argument of a `call:` message.


```
self call: (aComponent
    addDecoration: aDecoration; 
    yourself)
```


There are three kinds of decorations:

- **Visual Decorations.** These change a visual aspect of the decorated component:  `WAMessageDecoration` renders a heading above the component;  `WAFormDecoration` renders a form with buttons around the component; and  `WAWindowDecoration` renders a border with a close widget around the component.
- **Behavioral Decorations.** These allow you to add some common behaviors to your components:  `WAValidationDecoration` allows you to add validation of the answer-argument and the display of an error message.
- **Internal Decorations.** These support internal logic that you will use when building complex applications:  `WADelegation` is used to implement the `call:` message;  `WAAnswerHandler` is used to handle the `answer:` message.




#### Visual Decorations


**Message Decorations.**  `WAMessageDecoration` renders a heading above the component using the message `WAComponent>>addMessage:`. As an example we add a message to the `ContactView` component by sending it `addMessage:`, see *@ref:add-message@*.

```
IAddress >> initialize
    super initialize.
    editor := ContactView new.
    editor contact: self contacts first.
    editor addMessage: 'Editing...'.  " <-- added "
    editor onAnswer: [ :answer | self inform: 'Saved', answer printString ]
```





![Adding a message around a component. % width=80&anchor=ref:add-message](figures/AddMessage.png)


Note that the `WAComponent>>addMessage:` returns the decoration, so you may want to also use the `yourself` message if you need to refer to the component itself:

```
SomeComponent >> renderContentOn: html
    html render: (AnotherComponent new addMessage: 'Another Component'; yourself)
```


**Window Decoration.** `WAWindowDecoration` renders a border with a close widget around the component.

The following example adds a window decoration to the `ContactView` component. To see it in action, use the contacts application implemented by the `ContactList` component (probably at [http://localhost:8080/contacts](http://localhost:8080/contacts). The result of clicking on an edit link is shown in *@ref:window-decoration@*.

```
ContactView >> initialize
    super initialize.
    self addDecoration: (WAFormDecoration new buttons: #(cancel save)).
    self addDecoration: (WAWindowDecoration new title: 'Zork Contacts')
```



![Decorating a component with a window. % width=80&anchor=ref:window-decoration](figures/WindowDecoration.png)



!!note You may see that your _Save_ and _Cancel_ buttons are duplicated: you can remove this duplication by commenting out the `self renderSaveOn: html` line from `ContactView>>renderContentOn:`.

It is much more common to add a window decoration when calling a component rather than when initializing it. The following example illustrates a common idiom that Seaside programmers use to decorate a component when calling it. It uses a decoration to open a component on a new page.

```
WAPlugin >> open: aComponent
    "Replace the current page with aComponent."

    WARenderLoop new
        call: (aComponent
            addDecoration: (WAWindowDecoration new
                cssClass: self cssClass;
                title: self label;
                yourself);
            yourself)
        withToolFrame: false
```



**Form Decoration.** A  `WAFormDecoration` places its component inside an HTML form tag. The message `WAFormDecoration>>buttons:` should be used to specify the buttons of the form. The button specification is a list of strings or symbols where each string/symbol is the label (first letter capitalized) for a button and the name of the component callback method when button is pressed.

The component that a  `WAFormDecoration` decorates must implement the method `WAFormDecoration>>defaultButton`, which returns the string/symbol of the default button (the one selected by default) of the form. For each string/symbol specified by the  `WAFormDecoration>>buttons:` method, the decorated component must implement a method of the same name, which is called when the button is selected.

!!note Important Be sure not to place any decorators between `WAFormDecoration` and its component, otherwise the `defaultButton` message may fail.

You can examine the source of  `WAFormDialog` and its subclasses to see the use of a FormDecoration to manage buttons:

```
WAFormDialog >> addForm
    form := WAFormDecoration new buttons: self buttons.
    self addDecoration: form
```


```
WAFormDialog >> buttons
    ^ #(ok)
```


**Using Decorations in the Contacts application.** You can add a  `WAFormDecoration` to `ContactView` as follows: define an `initialize` method to add the decoration, and remove the superfluous rendering calls from `renderContentOn:`, to leave simpler code and an unchanged application (see Figure *@ref:form-contact@*).

```
ContactView>>initialize
    super initialize.
    self addDecoration: (WAFormDecoration new buttons: #(cancel save))
```


```
ContactView>>renderContentOn: html
    self renderNameOn: html.
    self renderEmailOn: html.
    self renderGenderOn: html.
    self renderSendUpdatesOn: html.
    self renderDateOn: html.
```

We chose `cancel` and `save` as our button names since these methods were already defined in the class, but we could have used any names we wanted as long as we implemented the corresponding methods.

![Using a decoration to add buttons and form to a ContactView. % width=80&anchor=ref:form-contact](figures/formContact.png)



#### Behavioral Decorators


**Validation.** A `WAValidationDecoration` validates its component form data when the component returns using `WAComponent>>answer` or `WAComponent>>answer:`. This decoration can be added to a component via the method `WAValidationDecoration>>validateWith:` as shown below.

```
SampleLoginComponent >> initialize
    super initialize.
    form := WAFormDecoration new buttons: self buttons.
    self addDecoration: form.
    self validateWith: [ :answer | answer validate ].
    self addMessage: 'Please enter the following information'.
```



If the component returns via `answer:`, the `answer:` argument is passed to the validate block. If the component returns using `answer` the sender of `answer` is passed to the validate block.

!!todo Talk about `WADelegation` and `WAAnswerHandler`.

**Accessing the component.** To access the component when you have only a reference to its decoration you can use the message  `WADecoration>>decoratedComponent`.

### Component Coupling 


Here is an interesting question that often comes up when writing components, and one which we faced when embedding our components: ''How do the components communicate with each other in a way that doesn't bind them together explicitly?'' That is, how does a child component send a message to its parent component without explicitly knowing who the parent is? Designing a component to refer to its parent \(as we did\) is not always an ideal solution, since the interfaces of different parents may be different, and this would prevent the component from being reused in different contexts.

Another approach is to adopt a solution based on explicit dependencies, also called the _change/update mechanism_. Since the early days of Smalltalk, it has provided a built-in dependency mechanism based on a change/update protocol--this mechanism is the foundation of the MVC framework itself. A component registers its interest in some event and that event triggers a notification.


**Announcements.** Perhaps the most flexible and powerful approach is to use announcements. While the original dependency framework relied on symbols for the event registration and notification, announcements promote an object-oriented solution; i.e. events are standard objects.


The main idea behind the framework is to set up common announcers and to let clients register to send or receive notifications of events. An _event_ is an object representing an occurrence of a specific event. It is the place to define all the information related to the event occurrence. An _announcer_ is responsible for registering interested clients and announcing events. In the context of Seaside, we can define an announcer in a session. For more information on sessions see XXXX.

**An Example.** Here is an example taken from Ramon Leon's very good Smalltalk blog (at [http://onsmalltalk.com/](http://onsmalltalk.com/)). This example shows how we can use announcements to manage the communication between a parent component and its children as for example in the context of a menu and its menu items.

First we add a reference to a new `Announcer` to our session:

```
MySession >> announcer
    ^ announcer ifNil: [ announcer := Announcer new ]
```


Second a subclass of an `Announcement` is created for each event of interest, here child removal:

```
Announcement << #RemoveChild
    slots: { #child};
    package: 'iAddress'
```


Each subclass can have additional instance variables and accessors added to hold any extra information about the specific announcement such as a context, the objects involved etc. This is why announcement objects are both more powerful and simpler than using symbols.

```
RemoveChild class >> child: aChild
    ^ self new
        child: aChild;
        yourself
```


```
RemoveChild >> child: anChild
    child := anChild
```


```
RemoveChild >> child
    ^ child
```

Any component interested in this announcement registers its interest by sending the announcer the message `on: anAnnouncementClass do: aBlock` or `on: anAnnouncementClass send: aSelector to: anObject`. You can also ask an announcer to `unsubcribe:` an object.

!!note The messages `on:do:` and `on:send:to:` are strictly equivalent to the messages `subscribe: anAnnouncementClass do: aValuable` (an object understanding `value`) and `subscribe: anAnnouncementClass send: aSelector to: anObject`. 

In the following example, when a parent component is created, it expresses interest in the `RemoveChild` event and specifies the action that it will perform when such an event happens.


```
Parent >> initialize
    super initialize.
    self session announcer on: RemoveChild do: [ :it | self removeChild: it child ]
```


```
Parent >> removeChild: aChild
    self children remove: aChild
```


And any component that wants to fire this event simply announces it by sending in an instance of that custom announcement object:

```
Child >> removeMe
    self session announcer announce: (RemoveChild child: self)
```


!!note advanced Note that depending on where you place the announcer, you can even have different sessions sending events to each other, or different applications.

**Pros and cons.** Announcements are not always the best way to establish communication between components and you have to decide the exact design you want. On one hand, announcements let you create loosely coupled components and thus maximize reusability. On the other hand, they introduce additional complexity when you may be able solve your communication problem with a simple message send.

### Summary


In this chapter we have seen how to embed components to build up complex functionality. In particular, we have learned:

- To embed a component in another one, the parent component should just answer the component as one of its children. Its `children` method should return the direct children components.
- Each component may render its immediate children in its own render method by calling various methods and possibly the `render:` method.
- A component may be reused with decorations. Decorations are components which add visual aspects or change component behavior.







