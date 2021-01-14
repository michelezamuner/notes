# Angular for Enterprise-Ready Web Applications

Angular follows the MV* pattern, which is a combination of the better-known MVC and MVVM ones. The MV* pattern relies upon the following layers:
- view layer: containing the raw templates that build up the application's presentation
- view model and controller: containing input/output logic, meaning routes definitions to handle URL inputs, and formatted data to be injected into the view templates
- service and model: services implementing client business logic, and models representing client data concepts

MV* applications are usually expected to interact with external services like RESTful APIs, GraphQL APIs, LocalStorage, etc., in order to access functionality and send or receive data.

In Angular, the ViewModel layer is implemented via bindings. In traditional MVC applications, the Controller has to call some kind of `render()` method in order to trigger the generation of the new view from the existing template, and the data provided by the controller. With bindings, a connection between a template element and an application model is always active. Depending on the kind of binding that has been setup, the model can be updated as soon as the template element changes, or the template element can be updated as soon as the model changes, or both. Since this connection is always active and handled by the underlying Angular framework, there's no need to explicitly trigger any template rendering.

The most basic unit of an Angular application is the component. A *component* is a combination of a TypeScript class and an Angular template. The template might contain HTML, CSS and TypeScript snippets, and is bound to its companion class through bindings, allowing the two to communicate seamlessly.

Templates have the following responsibilities:
- defining view elements that will make up the structure of the resulting Web pages
- binding component data to certain elements
- applying basic presentational logic like simple data formatting
- displaying graphic effects

Since a lot of code that make up these functionality can be reused across different components, Angular provides means to encapsulate chunks of code that provide specific pieces of functionality together:
- directives
- pipes
- user controls
- inclusion of other components

The component class is meant to take care of wiring together business services with more complex presentational logic. Services are external to components, and are meant to contain client business logic. Services are provided to components by means of dependency injection, which is provided out of the box by the Angular framework.

Going up an abstraction level, we find modules. *Modules* are collections of related application pieces, that might be components, services, directives, etc., that further help sorting out the application structure and organization.

Each Angular application must have a *root module*, containing a *root component*. Most simple applications will only need a single module, the root one, where we can define some other components in addition to the root one. The root module is equivalent to the main function of traditional applications: it's meant to be the most "dirty" one, and the most coupled one to other modules, since it's purpose is exactly that of wiring together all other dependencies in order to produce a fully working application, while allowing those other dependencies to remain as decoupled as possible from each other.

Angular by default employs a traditional imperative approach to the application execution, but it's also well integrated with the RxJS library, which allows to use a reactive approach. With reactive programming, everything is regarded as a stream of data, starting from user events. If a user clicks a button, a new `click` event is piped through the stream; now, if the user clicks the same button many times in a row before any following action has been performed, we want to discard all the additional useless clicks, so we pipe the stream to a `throttle` function, that takes everything that comes from the original stream, and produces a new stream, where it will forward all events coming from the original stream, but waiting 250 ms in between. Any consumer of the stream coming out of the `throttle` function will then receive a new event every 250 ms only. At this point we might pipe a `map` function to produce an asynchronous list of events that can be looped over; then we can pipe a `filter` function to trigger some code only when certain kinds of events happen. The norm with reactive programming is hence to rely on data streams, and not on any state that persists between different operations.

The policy for loading resources on Angular is based on the router configuration. All routes defined in the root router are eagerly loaded as soon as the user navigates to the application for the first time. To allow a set of resources to be loaded lazily, i.e. only when they're actually needed in response to some user action, we'd need to create a feature module featuring its own child router, and provide the new functionality under routes defined in the new child router. Under the roof, Angular will package this new module as a separate file, that can be downloaded separately and at a second time by the browser.

In Angular applications, components are generally short-lived, because they're unloaded from memory every time the user navigates to a different route: for this reason, we usually do't worry too much about maintaining state in the form of properties of the component class, because they're quickly going to be removed from memory. On the other hand, services that are injected into component classes are singletons, so they stay in memory even when the component class is destroyed: for this reason we must be very careful with storing state in services, because that might add memory consumption up very quickly. Only when we're building native applications of Progressive Web Applications we'll need to be able to store the entire application state, in case connectivity is not guaranteed, in order to save and resume the application state.

Besides these advices, the standard solution to manage state in Angular applications, provided by the NgRx library, is the Flux pattern. NgRx uses RxJS to provide a fully reactive state management solution; the power of this solution is mostly allowing developers to handle more than input sources that produce data that should be handled by the state: this is mostly the case for PWAs, while traditional Web applications usually need to deal with only APIs and user inputs, and for these cases NgRx might be overkill.

NgRx is based on the existence of a stream of actions, where events are sent from all sources. One source that dispatches actions to the action stream are components; another one are effects, which abstract events coming from APIs for example. Actions that are present on the stream can be listened to by subscribers: these subscribers can be reducers or again effects. Reducers and effects can execute code upon detection of new actions from the stream: reducers can interact with the store, while effects will interact with the API again. The store is where the application state is managed, and the components can read data from the store via selectors.

## Angular CLI

Angular comes with a CLI tool that automates a lot of repetitive operations that are often required when developing an Angular application. It's discouraged to install the CLI globally, though, since the tool is under active development, and it's best to leave every project with the CLI version it was developed with, even after some time. To bootstrap a new project with a local CLI:
```bash
npx @angular/cli new <project-name>
```

This will download `@angular/cli` and use it to create a new project in the folder `<project-name>`. Additionally, the CLI binary will be added as a local development dependency of the new project, in order for us to be able to keep using it during the development of the project. To run the CLI locally:
```bash
npx ng <command>
```


## References

- "Angular for Enterprise-Ready Web Applications Second Edition" by Douguhan Uluca, Packt 2020