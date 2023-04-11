# Angular

## Reference

https://angular.io/docs

## Introduction

Angular is a framework for building Single Page Applications based on HTML and TypeScript. An Angular application is a collection of Angular modules, called *NgModules*. Of these, there's always one *root module* that provides bootstrapping functionality, and additional *feature modules*. Feature modules represent Angular components, each of which has *views*, defining user interface widgets, and *services*, providing non-user interface functionality.

Each Angular building block, modules, components and services, are decorated TypeScript classes. Each component is related to a *template* that defines a view by combining regular HTML with Angular *directives* and *bindings*; each service is related to its dependencies.

Every Angular application has a root module, to bootstrap the application, and one *root component* that acts as the bridge between the application's components hierarchy, and the actual client's DOM.

In templates, directives allow to perform program logic, while bindings allow the template to react to events and data changes: in particular, *event bindings* allow to react to inputs coming from the client (and typically updating data in response), while *property bindings* allow to react to data changes (and typically updating the user interface in response).

## Installation

Many development operations that we do with Angular are carried out through the Angular CLI:

```bash
npm install -g @angular/cli
```

Once the CLI is installed, we can bootstrap a new application by running:

```bash
ng new <app-name>
```

This command will create a new directory containing the new project. Angular ships with a local server to aid local development; to start the server go into the project folder and run:

```bash
ng serve --open
```

where the `--open` option automatically opens the default browser at the URL where this application is running, which would be `http://localhost:4200/`. Additionally, the local server automatically starts a watcher, so that as soon as we make a change to the codebase, the running application is updated in the browser.

## Basic structure

The default application that is built with the `new` command contains just the root module. The root module is defined in the `app.module.ts` file, which takes care of importing all other modules that are needed in the application, and wiring together custom components. The root module initially contains a single root component, called `AppComponent`. This component is implemented with at least three files:

- `app.component.ts`: defining the code of the component
- `app.component.html`: defining the template of the component
- `app.component.css`: defining the CSS specific to this component

The recommended architecture for Angular applications is the following:

- each feature resides in its own folder, has its own Angular module and root component
- each feature root component has its own router outlet and child routes, and its routes rarely cross with routes of other features

## Modules

Modules are used to gather together a set of related definitions, which can be other modules, components or other definitions:

```typescript
/// <reference types='leaflet-sidebar-v2' />
import { NgModule } from '@angular/core';
import { LeafletModule } from '@asymmetrik/ngx-leaflet';
import { MapComponent } from './map.component';

@NgModule({
  declarations: [
    MapComponent
  ],
  imports: [
    LeafletModule,
    NgxSidebarControlModule
  ],
  exports: [
    MapComponent
  ]
})
export class MapModule { }
```

The `imports` section is used to include the definitions provided by other modules, both third-party or custom ones. These definitions will be visible within the scope of the current module. The `declarations` section is used to declare an element to the Angular application: this means we're talking about new definitions that have not been declared elsewhere, for which we'd have used an import instead. The `exports` section contains elements that have been declared in this module, that we want other modules to be able to use as well.

Since Angular doesn't accept that the same element is declared multiple times, we need to be careful not to add the same component to the `declarations` section of different modules. If we need to use a component declared in module A, from module B, we should export it from module A. Then, once we import module A in module B, that component will also be visible from module B.

The first line starting with `///` is a *compiler directive*. In particular, the `reference` directive tells the Angular compiler to manually include types defined in the module provided. This is needed when a third-party modules extends another third-party module, adding typings to an already existing typing set: in this case the compiler cannot know that the typings going under the name of the original typing set should also looked for in other different modules, so we must tell it explicitly.

Angular modules are also used to include different TypeScript definitions, much like ES2015 modules. To do this, we can just `export` a definition from a TypeScript file, and `import` it where we need it:

```typescript
// src/app/hero.ts
export interface Hero {
  id: number;
  name: string;
}
```

```typescript
// src/app/heroes.component.ts
import { Hero } from '../hero';
```

We can create custom modules with the CLI:

```bash
ng generate module app-routing --flat --module app
```

Here the `--flat` option instructs the command to add the module's files to the project root, instead of creating it's own directory, like it normally does when we create components or services. The `--module=app` option tells the command to automatically register the new module with the root module.

To create a new module with routing support:

```bash
ng generate module heroes/heroes --module app --flat --routing
```

## Components

To create a new custom component, we use the CLI:

```bash
ng generate component heroes
```

Each component, besides the root one, comes in its own directory: this command, then, will create a new directory named `heroes`, including the usual component files.

Each component needs at least three metadata:

- `selector`: the CSS element selector, to locate the graphical widget of the component somewhere in the DOM,
- `templateUrl`: the location (usually local) of the HTML file defining the template for this component
- `styleUrls`: an array of locations for the stylesheets of this component

Any initialization logic for the component should go into the `ngOnInit()` method, which is called by the framework shortly after the component is created.

To use a component, we need to reference it from the template of a parent component. For example, if we want our new `heroes` component to be child of the root component, we should add a reference to it to the template of the app component:

```html
<app-heroes></app-heroes>
```

This is enough to trigger the creation of the heroes component, and all related logic.

To create a component inside an existing directory:

```bash
ng generate component crisis-center/crisis-center
```

## Binding

### Interpolation binding

The most basic feature of a template is *interpolation binding*. A template is linked to its component's class, so that within the template we can access all the component's properties (including private ones). To print the value of these properties on the template, we write:

```html
<h1>{{ title }}</h1>
```

this way we created a binding between some data (the `title` property of the component) and an interpolation inside the template.

### Two-way binding

On the other hand, *two-way binding* is used to bind a template element to a model data, in such a way that the element displays the model data, but also when the element is changed, the change is reflected to the model data as well:

```html
<div>
  <label>name:
    <input [(ngModel)]="hero.name" placeholder="name"/>
  </label>
</div>
```

Here we're using the `ngModel` directive, which is not available in the root module, rather we have to import the `FormsModule` to use it. Since this is an external module that we need to use in our application, we need to add a reference to it to the root module:

```typescript
// app.module.ts
...
import { FormsModule } from '@angular/forms';
...
@NgModule({
  ...
  imports: [
    ...
    FormsModule
  ]
})
```

Now, Angular will try to update the `hero` model of the heroes component as soon as we type something in the `input` element: in order for this to work, though, the `hero` model needs to have been declared mutable, i.e. without the `readonly` qualifier.

### Event binding

Another kind of Angular binding is the *event binding*, that is used to bind an event to a listener:

```html
<li *ngFor="let hero of heroes" (click)="onSelect(hero)">
```

here we make it so that when the user clicks on the `li` element, the `onSelect` listener method is executed, being passed the current `hero` loop variable as argument.

### Class binding

We can bind the value of an attribute of an HTML element to some logic:

```html
<li [class.selected]="hero === selectedHero">
```

Here we tell Angular that a class named `selected` should have the value obtained by evaluating the expression `hero === selectedHero`. In this case, since the expression produces a boolean value, the result will be that the class will be removed if the value is `false`, or added otherwise.

### Property binding

When we nest a component inside another one, we might want to pass some data from the parent component to the nested one. We can do this with *property binding*:

```html
<app-hero-detail [hero]="selectedHero"></app-hero-detail>
```

here the parent component has a `selectedHero` property defined, and we want to make that object available to the nested `hero-detail` component. To do this, we bind the `selectedHero` parent property to the `hero` property defined in the nested component. However, for this to work we need to declare `hero` in the nested component so that it expects its value to come from an external component, instead from itself:

```typescript
import { Input } from '@angular/core';
...
export class HeroDetailComponent implements OnInit {
  ...
  @Input() hero: Hero;
  ...
}
```

Property binding can be used to also make a service available inside a template:

```html
<div *ngIf="messageService.messages.length">
  ...
  <div *ngFor="let message of messageService.messages">{{ message }}</div>
</div>
```

For this to work, however, it's important that the service is injected as `public`, because Angular can only bind public properties:

```typescript
...
import { MessageService } from '../message.service'
...
export class MessagesComponent implements OnInit {
  ...
  constructor(
    public messageService: MessageService
  ) { }
  ...
}
```

## Directives

Directives are applied to HTML elements to perform some action. The simplest directive is `*ngFor` which is a *repeater* directive, and it's used to perform loops:

```html
<ul class="heroes">
  <li *ngFor="let hero of heroes">
    <span class="badge">{{ hero.id }}</span> {{ hero.name }}
  </li>
</ul>
```

The `*ngFor` directive will copy the whole `li` element it's applied to, once for each element of `heroes`, and it will provide the current item `hero` as a variable that can be used inside the element.

The `*ngIf` directive displays an HTML element only if the expression provided is evaluated to `true`:

```html
<div *ngIf="selectedHero">
...
</div>
```

## Template

### Pipes

Angular provides a selection of filters that can be applied to bindings in order to change their output. These are called *pipes* and are used like this:

```html
<h2>{{ hero.name | uppercase }} Details</h2>
```

In this case we're using the `UppercasePipe`, which transforms a string into its uppercase version.

### Observables

Sometimes we need to work with observables inside templates, for example looping over the set of data produced by an observable:

```html
<li *ngFor="let hero of heroes$ | async">
```

In Angular we use the convention that a variable whose name ends with `$` is an observable, like in `heroes$`. Angular directives cannot directly work with observables, so what we need to do is to use the `AsyncPipe` as in `heroes$ | async` to actually bind the `hero` variable to the values that will asynchronously be produced by the observable.

### Template variables

*Template variables* are a way to reference a template element from another part of the template:

```html
<input #phone placeholder="phone number" />
```

here we are creating a new `input` element, and we're assigning to it the reference `#phone`. Now, we can refer to this `input` element from other parts of the template:

```html
<button (click)="callPhone(phone.value)">Call</button>
```

## Forms

Handling forms includes capturing user inputs, validating them, creating a model for the data that has been inserted and updating it as the inputs change. Angular provides two way to create forms: reactive and template-driven:

- *reactive forms* provide direct access to form model, which makes them more scalable, reusable and testable, thus more suitable for complex forms
- *template-driven forms* rely on directives in the template to manipulate the model, and they're more suitable for simple forms

### Reactive forms

With reactive forms, we define the form model in the component class:

```typescript
import { Component } from '@angular/core';
import { FormControl } from '@angular/forms';

@Component({
  selector: 'app-reactive-favorite-color',
  template: `
    FavoriteColor: <input type="text" [formControl]="favoriteColorControl">
  `
})
export class FavoriteColorComponent {
  favoriteColorControl = new FormControl('');
}
```

here the form model is the `FormControl` instance, and it's connected to the template via the `[FormControl]` directive.

When a user types a new value int the input element, the element emits an "input" event containing the new value; this is translated to a call like `favoriteColorControl.setValue("blue")`, which in turn emits a new event `valueChanges` which is dispatched to any registered subscriber.

On the other hand, when the application changes the value of the control through the code, with a call like `favoriteColorControl.setValue("blue")`, again the control emits a `valueChanges` event containing the new value, which is dispatched to all observers. One of these observers is the form input element, which is then updated with the new value.

### Template-driven forms

In template-driven forms, instead, the focus is shifted from the form model to the template:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-template-favorite-color',
  template: `
    Favorite Color: <input type="text" [(ngModel)]="favoriteColor">
  `
})
export class FavoriteColorComponent {
  favoriteColor = '';
}
```

Here we don't have direct access to the `FormControl` model, even though it's still automatically created and used to handle the data.

When a user types a new value into the input element, this element emits a new "input" event with the new value; this is again translated to a call to the `setValue()` method of the underlying `FormControl` instance, which again emits a new `valueChanges` with the new value, which is received by any registered observer. Then another call is made to `NgModel.viewToModelUpdate()`, which emits an `ngModelChange` event: this has the consequence that the `favoriteColor` property of the component is also updated.

On the other hand, when the `favoriteColor` component property is updated, a call is made to `NgModel.ngOnChanges()`, which queues an async task to update the underlying `FormControl` instance. This is done because the call to `NgModel` is done by a change detection mechanism that is always running, which creates tasks for all changes that are detected, and since they can be several, they are queued instead of executed directly. Only when change detection is completed, tasks are executed, including the new update to `FormControl`. This update in turn emits the new value through the `valueChanges` observable, which is received by any subscriber: one of these is the input element which updates its value.

## Service

To create a new service, we can again leverage the CLI:

```bash
ng generate service hero
```

which generates just a `hero.service.ts` file, containing the pre-defined `HeroService` class. This class is annotated with `@Injectable`, to signal that it must be able to be injected into other services or components, and that it must be able to let other services be injected into it.

In order to allow a service to be injected somewhere, we need to register a *provider* that knows how to create the new service to be injected in every circumstance. The provider needs to be registered with the *injector*, which is the object responsible to handle all the dependency injection that goes on during the application's execution.

When we run `ng generate service`, the command automatically registers a provider for the new service with the *root injector*: this is declared in the `providedIn` metadata of the `@Injectable` decorator of the service.

To perform the actual injection of our service, we just need to import it's type definition, and add it as a constructor argument of the service or component we want it to be injected in:

```typescript
...
import { HeroService } from '../hero.service';
...
export class HeroesComponent implements OnInit {
  ...
  constructor(
    private heroService: HeroService
  ) { }
  ...
}
```

Services should be used only from inside the listener of the `on-init` event, and not inside the constructor. The reason is that we don't have control of when our components will be created, because this falls into Angular's logic that takes also caching into account: this means that we cannot be sure of if and when the constructor will be called. Instead, we are sure that the `ngOnInit()` method will be called every time the component is initialized, even if it's built from a cache or something.

## Routing

Angular applications conventionally delegate routing logic to a dedicated module, called `app-routing`. This module, defined at the root level, has the responsibility to define all top-level routes, end to make the router available for the rest of the application:

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HeroesComponent } from './heroes/heroes.component';

const routes: Routes = [
  { path: 'heroes', component: HeroesComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

The `routes` constant will contain all top-level routes defined for the entire application. Currently, it only defines one route, accessible at the path `heroes`, that will display the `HeroesComponent` component.

Angular comes with a `RouterModule` that defines a singleton router that is able to listen to URL changes, and react to them by triggering the registered listeners. To use this router, we need to first import it in the current module, and then configure it telling it what's the list of routes it should be aware of: this is the purpose of `imports: [RouterModule.forRoot(routes)]`. Finally, the router just created is made available for the external application to use by the declaration: `exports: [RouterModule]`.

We can define a default route that will be used when no path is specified:

```typescript
{ path: '', redirectTo: '/dashboard', pathMatch: 'full' }
```

this means that when the path of the URL fully match the given path (which in this case is an empty string), then the router should redirect to the given path.

We can define a component that is used for any route:

```typescript
{ path: '**', component: PageNotFoundComponent }
```

Angular tries to apply the routes to the currently requested URL, in the order they have been defined. A common pattern is to define the wildcard '**' route as the last one, in order for it to be chosen only when all other defined routes don't match the URL, which is usually the case of a "page not found" situation.

We can pass generic data to the routes:

```typescript
{ path: 'heroes', component: HeroesListComponent, data: { animation: 'heroes' } },
```

This data can be retrieved from an instance of `RouterOutlet`:

```typescript
return outlet && outlet.activatedRouteData && outlet.activatedRouteData.animation;
```

### Router links

Angular provides the `routerLink` directive to point to the route path to which we want the router to navigate:

```html
<a routerLink="/heroes">Heroes</a>
```

We can assign a specific CSS class to the element that links to the route that is currently active:

```html
<a routerLink="/heroes" routerLinkActive="activebutton">Heroes</a>
```

here, whenever the application is displaying the `/heroes` route, this `a` element will be assigned the `activebutton` CSS class. If the link we provide is relative, the router will match multiple routes with that link:

```html
<a routerLink="./heroes" routerLinkActive="activebutton">Heroes</a>
```

in this case, if we are at `/heroes/my-hero`, the link will still be considered active. To prevent this, we can add an option to `routerLinkActive`:

```html
<a routerLink="./heroes" routerLinkActive="activebutton" [routerLinkActiveOptions]="{ exact: true }">Heroes</a>
```

the `exact` option tells the router whether to consider only exact route matches or not.

### Dynamic routes

We can define dynamic routes, containing parts that will match actual values provided in the real paths:

```typescript
{ path: 'detail/:id', component: HeroDetailComponent }
```

We can produce dynamic links with a `routerLink` directive like:

```html
<a [routerLink]="['/hero', hero.id]">
```

To get the `id` defined in such route, we need to use the `ActivatedRoute` module:

```typescript
import { ActivatedRoute } from '@angular/router';
...
export class HeroDetailComponent implements OnInit {
  ...
  constructor(
    ...
    private route: ActivatedRoute
    ...
  )
}
```

then we can access the `id` like:

```typescript
const id = +this.route.snapshot.paramMap.get('id');
```

### Location

The `Location` module allows us to perform actions with the current location. For example, we can navigate back to the previously browser route:

```typescript
...
import { Location } from '@angular/common';
...

export class HeroDetailComponent implements OnInit {
  ...
  constructor(
    ...
    private location: Location
    ...
  )
}
```

at this point we can make a call like:

```typescript
this.location.back();
```

to navigate back to the previous route.

### Multiple routers

If we define components inside a specific module that supports routing, we'll have routing defined in more than one file: the application-level routing module, and the module-level routing module.

While in the root-level routing we'd use `RouterModule.forRoot()` to register routes, in a module-level routing we need to call `RouterModule.forChild()` instead.

With multiple routing modules it's important that these modules are imported in the right order in the root module:

```typescript
...
imports: [
  BrowserModule,
  FormsModule,
  HeroesModule,
  AppRoutingModule
],
...
```

here, `HeroesModule` contains its own routing module that registers the routes `/heroes` and `/hero/:id`, while `AppRoutingModule` registers the `/crisis-center` route, in addition to the empty path route and the 404 route. When the application is set up, routes are registered in the order they appear, taking into account also the order of modules: in this case, thus, first the routes defined by `HeroesModule` are registered, and then those defined by `AppRoutingModule`; this way, the empty route and 404 route are defined last, when all meaningful routes have already been tried. Had `AppRoutingModule` been defined before `HeroesModule`, the wildcard route would have been triggered also for the legitimate routes defined by `HeroesModule`, making them unreachable.

### ParamMap

The `ParamMap` tool provides features to handle routing parameters. A `ParamMap` maps routing parameter names to actual values; for example, if a route is defined with a path like `/hero/:id`, and then the user navigates to `/hero/11`, the `ActivatedRoute` will have a `ParamMap` that maps the `id` parameter to the `11` value.

The easiest scenario is when a different component is loaded each time: for example the user navigates to a list of posts, then to a specific post, then back to the list of posts. In this case each time the `ActivatedRoute` refers to a new component, so the new component will be instantiated in memory each time, and it's `ngOnInit` listener will be called, which is where usually we check `ActivatedRoute` to populate the template with data according to the current parameter.

In this case, we just need to access the initial value of the parameter map, which is done with the `snapshot` property:

```typescript
this.route.snapshot.paramMap.get('id')
```

However, it may happen that the user navigates from a component, to the same component with just different routing parameters, for example from `/hero/11` to `/hero/12`. In this case it makes no sense to unload the component from memory, and instantiate it back, being the same component. This is actually what the Angular router does out of the box: if navigate to a different route that is linked to the same component, the same component instance is reused. However, the `ActivatedRoute` needs to be updated with the new values of the parameters: to do this we need to detect changes to the `ParamMap` of the `ActivatedRoute` from within the same component. We can do this by avoiding taking the snapshot, and getting directly the `paramMap` from the route, with `this.route.paramMap`.

In this case the `ParamMap` instance is an `Observable`, so the first thing we can do is to register functions to be called every time the observable is resolved, which will happen as soon as the route changes, coming with new parameters. To register a function to be called on resolution, we can use the `pipe` method to register a list of functions to be applied in order to the resulting value.

When a `ParamMap` is resolved, it provides another instance of `ParamMap`:

```typescript
this.route.paramMap.pipe(
  switchMap((params: ParamMap) =>
    this.service.getHero(params.get('id')))
)
```

which can be used to retrieve the actual parameters, for example by calling `params.get('id')`.

Now, it can happen that the user triggers a request for a route like `/hero/11`, but then, while this request is still being processed, it requests a new route, like `/hero/12`. What we need in this case is to discard the original `ParamMap`, the one related to `/hero/11`, because we're not going to need it, and replace it with the new one related to `/hero/12`. This is exactly what the `switchMap` function does.

### Navigation

We can use the singleton `Router` instance to directly navigate where'd like to. We can build URLs optionally passing some data to it:

```typescript
this.router.navigate(['/heroes', { id: heroId, foo: 'foo'}])
```

this would produce the following URL: `/heroes;id=15;foo=foo`, which uses matrix URL notation because this data conceptually belongs to the child route, while query parameters would belong to the parent route. These parameters are still retrievable from the `ParamMap` of the destination component.

### Child routes

Child routes are used when components belong to other components instead of to the root component:

```typescript
routes = [
  {
    path: 'crisis-center',
    component: CrisisCenterComponent,
    children: [
      {
        path: '',
        component: CrisisListComponent,
        children: [
          {
            path: ':id',
            component: CrisisDetailComponent
          }
        ]
      }
    ]
  }
]
```

Here we're producing routes that respond to paths like `crisis-center/:id`, that we could've defined directly, without child routes. The main difference with child routes, though, is that this time the router display child routes into the `RouterOutlet` of the parent component, and not of the root component.

It's not necessary that a child route has a component associated to it, if it's only used to group other routes together:

```typescript
routes = [
  {
    path: 'admin',
    component: AdminComponent,
    children: [
      {
        path: '',
        children: [
          { path: 'crises', component: ManageCrisesComponent },
          ...
        ]
      }
    ]
  }
]
```

When referring to paths starting with the slash, like in `/crisis-center`, we are pinning this path to the root of the routes configuration. Instead, if we omit the slash, we're pinning the route to the current URL segment. It's usually better to use relative paths, because parent paths might change, in which case we'd have to update all absolute paths referring to that parent. The router understands directory-like syntax, so `./some-path` is equivalent to `some-path`, and `../` refers to the parent segment.

Relative links built with a `RouterLink` directive are able to understand what's the current route segment, in order to know what does the relative path mean, but relative links build inside the component must be enhanced with the current `ActivatedRoute`, because they have no way to know what's the current position:

```typescript
this.router.navigate(['../', { id: crisisId, foo: 'foo' }], { relativeTo: this.route });
```

### Outlets

Even with routing, Angular applications are still single-page applications, so even when we navigate to a different URL, we're not really issuing any request to a server, nor loading new pages. This means that when the router "navigates" to a new route, it will just print the resulting HTML of the page in the current page. However, we need to tell it precisely what's the place of the page where to print the pages it navigates to. This is the purpose of the `<router-outlet></router-element>` element, that thus need to be added somewhere in the main application template.

In order for the `router-outlet` component to be visible in the template, we need to have exported `RouterModule` from the routing module.

Every other HTML element present after the `router-outlet` element will be present in all routes.

Every router has a single unnamed outlet, but it can have any number of named outlets, to which different routes can be attached, named *secondary routes*:

```html
<router-outlet name="popup"></router-outlet>
```

To specify that a route belongs to a certain outlet:

```typescript
{ path: 'compose', component: ComposeMessageComponent, outlet: 'popup' },
```

To build a link that targets a specific outlet:

```html
<a [routerLink]="[{ outlets: { popup: ['compose'] } }]">Contact</a>
```

When using secondary routes, the router keeps track of both the primary, and the secondary navigation, by adding the secondary one between parentheses, like `http://.../crisis-center(popup:compose)` and `https://.../heroes(popup:compose)`. So we can keep navigating with the primary routes, while we remember the same position in the secondary navigation.

To clear a secondary route we should:

```typescript
this.router.navigate([{outlets: { popup: null }}]);
```

Which is like the previous link, but this time we pass `null` as a path, which signifies that we should remove the view attached to the secondary outlet, and the secondary routes from the URLs.

### Route guards

A route guard is invoked whenever a route is requested, and has the purpose of applying some business logic to determine if the user can proceed navigating to that route. A guard will return `true` if the user can proceed with navigation, `false` if the user is refused to proceed, or a `UrlTree` if the user needs to be redirected somewhere else. Guards must implement these interfaces:

- `CanActivate` for guards that check if the user can navigate to a route
- `CanActivateChild` for guards that check if the user can navigate to a child route
- `CanDeactivate` for guards that check if the user can navigate away from the current route
- `Resolve` for guards that need to retrieve data before a new route is activated
- `CanLoad` for guards that check if a feature module can be loaded asynchronously

Guards are checked in this order:

- `CanDeactivate` and `CanActivateChild` from the deepest child to the top
- `CanActivate` from the top to the deepest child
- `CanLoad` if the feature module is loaded asynchronously

If any guard returns `false`, any pending guard that didn't complete its asynchronous check yet will be canceled, and the entire navigation is canceled.

To create a new guard for our application:

```bash
ng generate guard auth/auth
```

```typescript
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';

@Injectable({
  providedIn: 'root',
})

export class AuthGuard implements CanActivate {
  canActivate(next: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean {
    console.log('AuthGuard#canActivate called');
    return true;
  }
}
```

The `ActivatedRouteSnapshot` contains the route that the user wants to navigate to, and the `RouterStateSnapshot` contains the state that the router will be in, should the guard check be successful.

to apply a guard to a route, it's enough to add a parameter to the route definition:

```typescript
{ path: 'admin', component: AdminComponent, canActivate: [ AuthGuard ], ... }
```

## E2E tests
