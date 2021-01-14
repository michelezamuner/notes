# Client interface

## Handling input

### Client communication

Clients are distinguished from drivers on the basis that clients send messages to the service application, while drivers are those the service application sends messages to. Thus, in the application-driver communication the application acts as a client.

For example, a command line client will initiate the communication with the application by running a command, representing the first message that is sent, which might contain some data in the form of command line arguments. The application might then  produce some output, usually on `STDOUT` or `STDERR`, and in the case of an interactive command the process won't exit then, but the client will have the possibility to send another message, usually through `STDIN`. In general, though, input and output are not necessarily strictly alternated, meaning that the client can send inputs at any time, and the application can produce outputs at any time, unless the design of the specific application states otherwise.

Likewise, in a network application the client will establish a new connection with the server mediator by sending a message carrying some data, which will be sent by the mediator to the application as a message, and the application will usually produce some output that will be turned into a network response by the mediator. In the general case the connection will be long-lasting, and the client will be able to send multiple messages over the same connection.

### Input and output boundaries

The interface of a port is composed of two kinds of sub-interfaces: the input boundary and the output boundary. The input boundary is in general a number of message types that the port can understand. These are a generalization of the communication situations that can arise. In the case of a native call for example, the message will just be the call itself to a specific method of the code interface, with the given arguments. The output boundary will be the set of possible messages that can be generated as outputs in different situations in response to the execution of the use case that lies behind the port.

Input messages can usually be distinguished between commands and queries: where commands are expected to produce side effects, while queries are expected to cause some output to be produced with the predefined output message types. In the most simple cases the input and output implementations will be located in the same place in the client adapter, and thus the same object will immediately receive an output (a "response") as soon as it has sent the input (the "request").

The most famous example is probably HTTP, where the message type is a combination of the URI and the HTTP method, and the message data would gather together the request query and body, while the output message would then of course contain the response status and body.

### Inlets, outlets and widgets

The client will have to implement components that are able to talk to the input and output boundaries of the application. A client component that implements the input boundary is an inlet; a client component that implements an output boundary is an outlet.

A generalization of a client component would bare both input and output responsibilities. From this, input-only or output-only components, that is inlets and outlets, would be considered edge cases. A component that is both an inlet and an outlet is a widget. The user interface of a client is thus generally composed of widgets.

### Stateful communication

When multiple users/devices of the same type are connected to the software system at the same time, we need to handle this concurrent situation. The most simple scenario is when the system runs over a single process, and then there's only a single client instance that needs to handle the user inputs one at a time.

However, since the communication between the user and the client is generally a long lasting stream of messaging, this communication ends up being also stateful, meaning that what the client and the application will have to do at each moment will depend on the specific messages that they had exchanged since then, and that the application will have to behave differently when called by the same client over different user connections.

To allow this, we at least have to duplicate the client components that are stateful, so that there's a different instance of each component sending messages to the application, which are the translations of the inputs of the different parallel users communicating over different connections.

In some cases, however, some component will be stateless, because all the information needed to carry out the task will be contained in each request, and in these cases we can avoid duplication. The edge case is that of Web applications, which are notoriously stateless by design, and as such employ all stateless controllers (if we don't consider cookies and sessions of course).

### Widgets relations

It's important to underline that widgets should only have responsibilities that are related to the handling of the messages, and that every specific computation should be delegated to application or domain services. This way we also avoid the need to set up relations between widgets. When a widget receives an input, it sends a message to a specific application use case, which in turn uses other application or domain services to perform its computation. If a different use case needs to perform part of the same computation when called by a different widget, it will just reuse the related services, without the originating widget ever needing to know about it. This way the two widgets are left with just the responsibility of handling the different inputs, and translating them to the proper use case calls.

A debatable point is whether widgets should have parent-children relations, like the widgets of graphical interfaces, that can be contained one in another. From a first thought though, this seems to only be related to the specific graphical concerns of GUIs, where widgets have a geometrical appearance, and as such can share the same screen space. In a general understanding of widgets as input and output handlers, there seems to be no reason to add such kind of relation.

It can happen that the same message should trigger more than one use case at once, or, reworded, that multiple use cases might be interested in the same message type. In this case we can just apply the standard solution of implementing a publish-subscriber design where multiple use cases can subscribe to the same message type, and messages are sent to the subscribed use cases by a dipatcher.

## Handling output

### Segregation of input and output

The most common circumstance is that clients can handle both input and output: a CLI application can both get inputs from the command arguments and `STDIN`, and send outputs to `STDOUT` and `STDERR`, which are all connected to the same user device, which is the console; likewise in a Web application the browser can both take user inputs and display outputs, and the same can be said for GUI applications. However, we can easily think of cases where a client can only handle input, like that of a sensor device that can just send the data that it detected, or a feed that just sends some data regularly, without caring of what will be done with it.

In general, then, the application cannot expect that the output can be sent back to the same client that produced the input. If the inlets are responsible to receive the input and trigger an application reaction from it, outlets are responsible to forward the output data from the application to the proper output devices: these are called "presenters" in the Clean Architecture for example. The communication interface, again, will then be defined in terms of inlets and outlets, input boundaries and output boundaries.

This still happens even with streaming communication, where user connections with clients stay open for a indeterminate amount of time. Rather than seeing the streaming communication as an alternate request-response sequence, we need to realize that there are distinct input streams and output streams, and if the application waits for a new message to come from the input stream before sending a new one into the output stream, that's just a specific design of the application itself: nothing prevents an application from writing autonomously new messages to some output streams at random times without waiting for any action from the client.

### Output differentiation

While a use case interactor performs its application logic, it may not need to perform a single presentation (output), for two reasons:
- there are many different kinds of presentations, that will likely need to be handled differently, like a main successful output, and an error output
- there are some presentations that will be produced immediately, like input validation results, and some that may be produced asynchronously, like command execution results

For this reason, the interactor will in general have to define several different outlet interfaces, instead of just one, and juggle the various outlets according to the application logic that is unfolding.

If the communication between client and application is happening through native runtime calls, the coupled and synchronous nature of the communication allows us to let the input boundary to immediately "return" a result from the call, which is in fact the most used outlet implementation of the most common and simple architectures, like those seen on the Web.

This would actually work, and keep the dependency inversion principle respected, since the interactor still knows nothing of the details of output handling, however:
- we need to stuff all possible response cases into a single response object, instead of cleanly separate the various cases
- the inlet will need to parse the response, understand what case it's about, and handle it differently, taking much more responsibility than just having to translate the input data from the adapter format to the application format; additionally, the inlet would really be just unpacking the same information that was packaged by the interactor, i.e. that a certain outlet needs to be called in a certain output situation, and the inlet cannot deviate from this logic either, otherwise it would be taking presentation responsibility
- the single response object could be returning asynchronous output, in addition to synchronous one, so the inlet will also need to check which part of the response should be treated as asynchronous (like attaching a callback to a promise), and which as synchronous
- even if the interactor was a coroutine (avoiding the problem of creating a single response for all cases), the inlet would still need to check the kind of output at each response, and do the job of the interactor of associating an outlet to that kind of output, while the interactor already knows this information in the first place, in addition to still having to handle asynchronous output

For these reason, although returning the response from the interactor is technically feasible (and in certain scenarios it's certainly the simplest solution), it cannot be chosen as the general approach to use: rather it should be regarded as a special case, that works best only in specific situations.

### Outlets and views

Each specific outlet that can be used to implement the output boundary represents a specific output model, which is characterized by the set of presentation fields, each of a specific data type, that it supports, or in other words, the kind of view model it will produce. For example, a "screen" view model might convert the application response to a certain set of fields of certain types, with the intent of having them being displayed to a screen, while a "print" view model might produce from the same application response a different set of fields, of different types, specialized for being sent to a printer.

Still, the same view model, representing a certain output model, can be rendered on the selected output device in different ways: a "screen" output model, containing fields belonging to a screen representation, could still be rendered as an HTML page, or as TXT file, or also as a graph. All these alternatives are represented by different views. This means that a specific outlet component, for example the "screen" outlet component, will define a "screen" view interface, that will then be implemented by multiple actual "screen" view instances, like HTML screen view, graph screen view, etc., one of which will then be selected and injected in the outlet, to be used for the actual rendering.

## Binding boundaries

Since the use case interactor must depend on specific implementations of the output boundary, i.e. of the outlets, we need to be able to bind these implementations at the moment of the creation of the interactor. This is made more complicated by the fact that the specific outlet implementation might depend on the specific user request: for example a Web request might specify what kind of resource representation is required, like PDF instead of HTML, meaning that the PDF view of the resource outlet must be injected instead of the HTML one.

In the most simple architectures, the Web controller has the additional responsibility of handling the service output, and thus can select the correct view just in time. However, in the most general case we want the inlet to only care about input handling, and not being concerned with any output issue.

The general solution to this is using conditional binding, where the DI container is configured to bind one view implementation or another according to a parameter that will be available at runtime. This, however, means that the whole stack of inlet, interactor, outlets and views must be rebuilt for every request, since the requested views can be different. This can however be optimized for example with caching at the container level. In more simple architectures the DI container is replaced with a "router".

## Communication mediators

In the most simple case the application interface, i.e. the port, will be an actual code interface: an object implementing that interface will be injected in the client object during the runtime, and the client will "send messages" to the application by calling the code interface methods.

More in general, the communication between the client and the application will go through a mediator component. The mediator can have several forms: a publish-subscribe dispatcher running in the same process, a message broker running as a separate process and communicating over TCP with AMQP, a Web server communicating via HTTP REST, etc. We can in general assume that there will always be a mediator, because in the case of the native calls the mediator would be embedded with the runtime environment which is making the code calls work.

When considering different types of mediators, we're dealing with the following dimensions: logical coupling, concurrency and asynchronicity.
- native runtime calls:
	- when new messages are added to the interface, the client modules must be updated: maximum coupling
	- in the same process there can be only one client and one application running: no concurrency
	- the message is sent by the client to the application synchronously: no asynchronicity
- same-process dispatcher:
	- the client doesn't need to be updated if new messages are supported by the interface: minimum coupling
	- the same client message can be caught by multiple ports, but there'll only be one client: half concurrency
	- the message is sent by the client to the application synchronously: no asynchroniciy
- message bus/queue service/TCP server:
	- the clients don't need to be updated if new messages are supported by the interface: minimum coupling
	- many clients can send messages to the mediator, and many workers can handle them: maximum concurrency
	- application workers can pick up messages at any time: maximum asynchronicity

So we'll be able to choose a different type of communication mediator according to the specific requirements we have in terms of logical coupling, concurrency and asynchronicity.
