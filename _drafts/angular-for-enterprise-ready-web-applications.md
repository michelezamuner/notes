# Angular for Enterprise-Ready Web Applications


## Introduction

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

Each Angular application must have a *root module*, containing a *root component*. Most simple applications will only need a single module, the root one, where we can define some other components in addition to the root one. The root module is equivalent to the main function of traditional applications: it's meant to be the most "dirty" one, and the most coupled to other modules, since its purpose is exactly that of wiring together all other dependencies in order to produce a fully working application, while allowing those other dependencies to remain as decoupled as possible from each other.

Angular by default employs a traditional imperative approach to the application execution, but it's also well integrated with the RxJS library, which allows to use a reactive approach. With reactive programming, everything is regarded as a stream of data, starting from user events. If a user clicks a button, a new `click` event is piped through the stream; now, if the user clicks the same button many times in a row before any following action has been performed, we want to discard all the additional useless clicks, so we pipe the stream to a `throttle` function, that takes everything that comes from the original stream, and produces a new stream, where it will forward all events coming from the original stream, but waiting 250 ms in between. Any consumer of the stream coming out of the `throttle` function will then receive a new event every 250 ms only. At this point we might pipe a `map` function to produce an asynchronous list of events that can be looped over; then we can pipe a `filter` function to trigger some code only when certain kinds of events happen. The norm with reactive programming is hence to rely on data streams, and not on any state that persists between different operations.

The policy for loading resources on Angular is based on the router configuration. All routes defined in the root router are eagerly loaded as soon as the user navigates to the application for the first time. To allow a set of resources to be loaded lazily, i.e. only when they're actually needed in response to some user action, we'd need to create a feature module featuring its own child router, and provide the new functionality under routes defined in the new child router. Under the roof, Angular will package this new module as a separate file, that can be downloaded separately and at a second time by the browser.

In Angular applications, components are generally short-lived, because they're unloaded from memory every time the user navigates to a different route: for this reason, we usually don't worry too much about maintaining state in the form of properties of the component class, because they're quickly going to be removed from memory. On the other hand, services that are injected into component classes are singletons, so they stay in memory even when the component class is destroyed: for this reason we must be very careful with storing state in services, because that might add memory consumption up very quickly. Only when we're building native applications or Progressive Web Applications we'll need to be able to store the entire application state, in case connectivity is not guaranteed, in order to save and resume the application state.

Besides these advices, the standard solution to manage state in Angular applications, provided by the NgRx library, is the Flux pattern. NgRx uses RxJS to provide a fully reactive state management solution; the power of this solution is mostly allowing developers to handle more than one input sources that produce data that should be handled by the state: this is mostly the case for PWAs, while traditional Web applications usually need to deal with only APIs and user inputs, and for these cases NgRx might be overkill.

NgRx is based on the existence of a stream of actions, where events are sent from all sources. One source that dispatches actions to the action stream are components; another one are effects, which abstract events coming from APIs for example. Actions that are present on the stream can be listened to by subscribers: these subscribers can be reducers or again effects. Reducers and effects can execute code upon detection of new actions from the stream: reducers can interact with the store, while effects will interact with the API again. The store is where the application state is managed, and the components can read data from the store via selectors.


## Installation

Angular comes with a CLI tool that automates a lot of repetitive operations that are often required when developing an Angular application. It's discouraged to install the CLI globally, though, since the tool is under active development, and it's best to leave every project with the CLI version it was developed with, even after some time. To bootstrap a new project with a local CLI:
```shell
npx @angular/cli new <project-name>
```

This will download `@angular/cli` and use it to create a new project in the folder `<project-name>`. Additionally, the CLI binary will be added as a local development dependency of the new project, in order for us to be able to keep using it during the development of the project. To run the CLI locally:
```shell
npx ng <command>
```


## Components basics

A component is responsible to fully implement a self-contained user interface widget, that is as much independent as possible from the rest of the application, and thus can be easily debugged and reused. To create a new component:
```shell
npx ng generate component current-weather
```

or in short:
```shell
npx ng g c current-weather
```

This command will generate the following files, which are the necessary building blocks for every component:
- `current-weather.component.css`: CSS specific for this component
- `current-weather.component.html`: HTML template of this component, defining its graphical interface and bindings (it represents the View part of the component)
- `current-weather.component.spec.ts`: unit test file for the component logic
- `current-weather.component.ts`: contains the user interface logic for this component (it represents the ViewModel part of the component)

The core of a component is its class, because it binds together all other parts as well:
```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector:. 'app-current-weather',
  templateUrl: './current-weather.component.html',
  styleUrls: ['./current-weather.component.css'],
})
export class CurrentWeatherComponent implements OnInit {
  constructor() {}
  ngOnInit() {}
}
```

Every component needs to be registered with the root module, `AppModule`, in order to be used. This is automatically done by the `ng g c` command:
```typescript
...
import { CurrentWeatherComponent } from './current-weather/current-weather.component';
...
@NgModule({
  declarations: [
    ...
    CurrentWeatherComponent,
  ],
}),
...
```

It's enough to import the component class because, as we've seen before, the class itself knows how to handle all other component files as well.

When we run an Angular application, the first things that happen is that the `index.html` file and the `main.ts` files are loaded (of course `main.ts` will be transpiled to JavaScript and bundled into a single script, but its logic is the first to be executed). The `index.html` template contains a reference to the `AppComponent`, in the form of the `<app-root></app-root>` directive. The `main.ts` bootstrap script, then, has the purpose of loading the `AppComponent`, so that the `<app-root></app-root>` directive actually does something. While the `AppComponent` is being loaded, all the other components that depend on it are loaded as well, and since the `AppComponent` is the root component, meaning that every other component are children of it, the whole application is loaded in response.

Once we created the new `CurrentWeatherComponent`, we need to add it to the existing `AppComponent`, in order for it to be loaded and used:
```html
<div style="text-align:center">
  <h1>
    LocalCast Weather
  </h1>
  <div>Your city, your forecast, right now!</div>
  <h2>Current Weather</h2>
  <app-current-weather></app-current-weather>
</div>
```

When the application is being loaded, Angular needs to go through the entire graph of modules, components and various dependencies, loading all of them in the correct order. All this computation is performed before HTTP and DOM can be accessed by the code, meaning that between the time the component class is instantiated (`constructor()` is called), and the time the component can actually start doing something meaningful, there is some delay: for this reason we should never try to access the DOM, or make calls to the Web, before the whole application has been loaded. To help with this, Angular provides the `onInit` hook, that we can use via the `ngOnInit()` method. What happens is that after the application has completed loading, Angular takes all classes enhanced with the `@Component()` decorator, and which implement the `OnInit` interface, and call their `ngOnInit()` method. This ensures us that every code that runs inside that method will be safely able to access the DOM and the Web.

While Angular imposes a strict structure on the definition of views and view-models, developers are free to define models as they want. The Angular CLI however still provides some scaffolding tool to help building models as well. For example we can create and interface with:
```shell
npx ng g interface ICurrentWeather
```

This will create the `icurrent-weather.ts` in the `app` folder:
```typescript
export interface ICurrentWeather {
}
```

For now, we prefer to keep several interfaces in a single file, so we'll rename `icurrent-weather.ts` to `interfaces.ts`, and update it to:
```typescript
export interface ICurrentWeather {
  city: string;
  country: string;
  date: Date;
  image: string;
  temperature: number;
  description: string;
}
```

This `ICurrentWeather` type will represent the current weather that will need to be displayed by the `CurrentWeatherComponent`, so we need this component to hold a reference to an object of that type:
```typescript
import { Component, OnInit } from '@angular/core';
import { ICurrentWeather } from '../interfaces';

@Component({
  selector: 'app-current-weather',
  templateUrl: './current-weather.component.html',
  styleUrls: ['./current-weather.component.css'],
})
export class CurrentWeatherComponent implements OnInit {
  current: ICurrentWeather;

  constructor() {
    this.current = {
      city: 'Bethesda',
      country: 'US',
      date: new Date(),
      image: 'assets/img/sunny.svg',
      temperature: 72,
      description: 'sunny',
    } as ICurrentWeather;
  }
  ngOnInit() {}
}
```

We can now access this model from our view:
```html
<div>
  <div>
    <span>{{ current.city }}, {{ current.country }}</span>
    <span>{{ current.date | date: 'fullDate' }}</span>
  </div>
  <div>
    <img [src]="current.image">
    <span>{{ current.temperature | number: '1.0-0' }}°F</span>
  </div>
  <div>
    {{ current.description }}
  </div>
</div>
```

The template can directly access component's properties with the `{{ ... }}` syntax, and apply "pipes" to those values to format them before they're displayed.


## Calling an external API

We want to get actual weather data from an external API, which is Open Weather Map. To access this API, we'd call an URL like this:
```
http://api.openweathermap.org/data/2.5/weather?q=London,uk&appid=<appid>
```

Since we need to provide an `appid`, we have not only to store this ID somewhere in our application, but also to differentiate between the ID for the production environment, and the ID for the development environment. Angular by default distinguishes between a production environment, and the default one, and does this by providing two sources of environment variables. Let's add the `appid` to the default environment, by editing `src/environments/environments.ts`:
```typescript
export const environment = {
  production: false,
  appId: '<appid>',
  baseUrl: 'http://',
};
```

Since accessing this service is part of the application business logic, we implement it into a separate service:
```shell
npx ng g s weather --flat=false
```
This command generates a new service called `WeatherService` inside the new `weather` folder. In particular, services are generated by default in the root module, unlike components that are by default generated inside their own folder, but passing the `--flat=false` option we instruct the Angular CLI to generate the module inside its own folder. The new service has this form:
```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class WeatherService {
  constructor() {}
}
```

Here `proidedIn: 'root'` means that this service will be instantiated as soon as the application is loaded, instead of waiting for the actual components that need it to be instantiated. Alternatively, we could jave just imported this service class in the root module, to have the same effect. Services that are not declared with `providedIn: 'root'` need to be declared in the `providers` section of the module where they're used (can also be the root module), and are instead instantiated only the first time they are requested by a component, and become singleton after that.

To perform API calls, we'll use the `HttpClient` Angular module, that needs to be imported in the root module:
```typescript
...
import { HttpClientModule } from '@angular/common/http';
...
@NgModule({
  ...
  imports: [ ..., HttpClientModule ],
  ...
})
```

Then we can import the `HttpClient`, which is provided by `HttpClientModule`, inside our new `WeatherService`:
```typescript
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';

@Injectable()
export class WeatherService {
  constructor(
    private httpClient: HttpClient
  ) {}
}
```

Since `HttpClient` is strongly typed, we need to prepare an interface that matches the structure of the API that we're going to call, but including only the data that we're interested in. Additionally, since this interface is only used to talk to the API, we can directly define it inside the `weather.service.ts` file, instead of creating its own file:
```typescript
interface ICurrentWeatherData {
  weather: [
    {
      description: string,
      icon: string
    }
  ],
  main: {
    temp: number
  },
  sys: {
    country: string
  },
  dt: number,
  name: string
}
```

Now let's add a new method to make the actual API call:
```typescript
import { ..., HttpParams } from '@angular/common/http';
import { environment } from '../../environments/environment';
...
export class WeatherService {
  ...
  getCurrentWeather(city: string, country: string) {
    const uriParams = new HttpParams()
      .set('q', `${city},${country}`)
      .set('appid', environment.appId);

    return this.httpClient
      .get<ICurrentWeatherData> (
        `${environment.baseUrl}api.openweathermap.org/data/2.5/weather?`,
        { params: uriParams }
      );
  }
}
...
```

Now, to be able to show the API data in our `CurrentWeatherComponent`, we need to inject the `WeatherService` into the component, and use its new method:
```typescript
...
import { WeatherService } from '../weather/weather.service.ts';
...
constructor(
  ...
  private readonly weatherService: WeatherService
) {}
...
ngOnInit() {
  this.weatherService.getCurrentWeather('Bethesda', 'US')
    .subscribe((data) => this.current = data);
}
...
```

The call to `getCurrentWeather()` returns an object of type `Observable<ICurrentWeatherData>`. Observables are the core of the `RxJS` library, and represent event emitters, to which observers can register to react to events that will be generated. In particular, this object will emit objects of type `ICurrentWeatherData`. The `httpClient.get<>()` method, additionally, when it's called just prepares the observable for future use, meaning that no HTTP request is made at the time `httpClient.get<>()` is called. Only when the first listener is registered, then the actual call is performed, in order to avoid making unnecessary calls in case no observer is actually used. Thus, the actual HTTP call is only made when `.subscribe()` is called on the observable. The `data` parameter that is passed to the listener, is then of the type declared by the service: `ICurrentWeatherData`, and once it's available we store it in our component property.

Since the property `current` is bound to the component's template, every time this property changes, for example as soon as the API sends a response back with the weather data, the template is automatically updated.

There's one last fix to make in order for this to work: we're retrieving data of type `ICurrentWeatherData`, but our `current` property is of type `ICurrentWeather`. To do this, we can change the implementation of the service in order to map the data from one type to the other:
```typescript
...
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
import { ICurrentWeather } from '../interfaces';
...
export class WeatherService {
  ...
  getCurrentWeather(city: string, country: string): Observable<ICurrentWeather> {
    ...
    return this.httpClient
      .get<ICurrentWeatherData>(
        `${environment.baseUrl}api.openweathermap.org/data/2.5/weather`,
        { params: uriParams }
      )
      .pipe(map(data => this.transformToICurrentWeather(data)));
  }

  private transformToICurrentWeather(data: ICurrentWeatherData): ICurrentWeather {
    return {
      city: data.name,
      country: data.sys.country,
      date: data.dt * 1000,
      image: `http://openweathermap.org/img/w/${data.weather[0].icon}.png`,
      temperature: this.convertKelvinToFahrenheit(data.main.temp),
      description: data.weather[0].description,
    }
  }

  private convertKelvinToFahrenheit(kelvin: number): number
  {
    return kelvin * 9 / 5 - 459.67;
  }
}
...
```

Since we're mapping `ICurrentWeatherData.dt` to `ICurrentWeather.date`, we need to convert the latter to `number`:
```typescript
export interface ICurrentWeather {
  ...
  date: number,
  ...
}
```

We're doing this instead of creating a new `Date` out of the original timestamp, because we're applying the `DatePipe` anyway in the template, which can work with timestamps as well; however, the original API timestamp is in seconds, and not in milliseconds, so we need also to multiply by 1000.

By calling the `RxJS`'s `map` operator we can apply the given function to all objects that are produced by the observable, the moment they're produced, and return a new observable that produces the results of this function application: in this case the first observable produces `ICurrentWeatherData` objects, and the second produces `ICurrentWeather` objects


## Null guards

When dealing with asynchronous operations we always run into the situation where a variable or property stays `null` or `undefined` until it's populated by incoming data. If we cannot devise sound default values for these variables, we should check if they have a value before trying to use them.

To accomplish this, we use the `*ngIf` Angular directive, to ignore parts of the template if a certain condition is not met:
```html
<div *ngIf="!current">
  no data
</div>
<div *ngIf="current">
  ...
</div>
```


## Unit tests

Angular unit tests are not actually pure unit tests when it comes to component testing, because they load entire components and render their views, to test them against the DOM, so there's this external dependency. This is necessary to be able to test bindings between the component logic and its template, which go through the DOM itself.

By default Angular uses the Jasmine testing framework, whose test suites are defined in files with the `spec.ts` extension. Unit tests in Angular use the `TestBed` component, that is responsible to wire everything up in order to create the components that are going to be tested, including provisioning modules, injecting dependencies, creating mocks, triggering Angular life-cycle events and executing template logic. While using `TestBed` then, the resulting tests are not really unit tests, due to the amount of different parts that are put together.

The Angular CLI creates stubs for unit tests when we generate certain types of items, like components and services. For example, the stub for `current-weather.component.spec.ts` is:
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';

import { CurrentWeatherComponent } from './current-weather.component';

describe('CurrentWeatherComponent', () => {
  let component: CurrentWeatherComponent;
  let fixture: ComponentFixture<CurrentWeatherComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ CurrentWeatherComponent ]
    })
    .compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(CurrentWeatherComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

Here the first `beforeEach()` declares and compiles the dependencies of the component that we want to test, while the second `beforeEach()` creates the actual component, and starts the listener that checks for changes in the component, in order to be able to test bindings.

To run all unit tests, we just need to execute:
```shell
npm test
```

This will open a new instance of the Chrome browser where the test application will be loaded and exercised, and test results will be displayed.

The first step in the execution of a component's unit test is the compilation of its dependencies. For this we need to define a testing module, which looks alike real Angular modules, and in particular it supports three configuration options: `declarations`, `providers`and `imports`.

Declarations are needed to build components that are needed to the current test. Usually simple tests will only need to declare only one component: the one that is under test, but in more complex scenarios we might need more than one. For example if we are creating a control (sub-component) that works only in the context of a containing component, to be able to test the control we of course need to load its component as well. Beware that, since declared items will be compiled, all their dependencies will need to be imported as well, so it's easy to end up having to deal with a lot of missing dependencies when declaring additional components. However, the `TestBed` testing framework is designed so that it's not needed to load sub-components to test a component: this is the reason why, for example, the unit test for the root component doesn't need to declare the other components that are used inside the root one as well.

Providers are needed to instruct the dependency injection container on how to instantiate dependencies that are declared in the component under test. If we added dependencies to our component, we need to update the unit test as well with mock versions of those dependencies.

Imports are needed to import external modules that contain functionality that our test needs. However, it's rarely the case that we should import external modules in Angular unit tests, because this would make them even less isolated.

When we add a new component, and use it in the root component, unit tests for the root component will start complaining with an error like:
```
ERROR: 'NG0304: 'app-current-weather' is not a known element:1. If 'app-current-weather' is an Angular component, then verify that it is part of this module.
2. If 'app-current-weather' is a Web Component then add 'CUSTOM_ELEMENTS_SCHEMA' to the '@NgModule.schemas' of this component to suppress this message.'
```

This means that the unit test for the root component does not know how to resolve the `app-current-weather`. Instead of adding a declaration for the actual `CurrentWeatherComponent` in the test, that would require to load all its dependencies as well, we can instead inject a mock component:
```typescript
...
import { Component } from '@angular/core';
...
@Component({
  selector: 'app-current-weather',
  template: '',
})
class CurrentWeatherComponentMock {}
...
await TestBed.configureTestingModule({
  declarations: [
    ...
    CurrentWeatherComponentMock
  ],
}).compileComponents();
...
```

Next, the tests for the `CurrentWeatherComponent` and for the `WeatherService` fail because they depend on `HttpClient`, which is not defined in the unit tests. Instead of injecting a real instance of `HttpClient`, let's first fix `CurrentWeatherComponent` by injecting a mock for `WeatherService`:
```typescript
...
import { createSpyObj } from 'jasmine-core';
import { of } from 'rxjs';
import { WeatherService } from '../weather/weather.service';
...
describe('CurrentWeatherComponent', () => {
  ...
  let weatherServiceMock: jasmine.SpyObj<WeatherService>;

  beforeEach(async () => {
    const weatherServiceSpy = jasmine.createSpyObj(WeatherService.name, ['getCurrentWeather']);
    await TestBed.configureTestingModule({
      ...
      providers: [
        {
          provide: WeatherService, useValue: weatherServiceSpy,
        },
      ],
    }).compileComponents();
    weatherServiceMock = TestBed.inject(WeatherService) as jasmine.SpyObj<WeatherService>;
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(CurrentWeatherComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    weatherServiceMock.getCurrentWeather.and.returnValue(of());
    fixture.detectChanges();
    expect(component).toBeTruthy();
  });
});
...
```

Here we first need to prepare a mock value for the weather service using the `jasmine.createSpyObj` utility, and configure it to be injected by the `CurrentWeatherComponent` wherever a `WeatherService` object is required. Next we fetch the actual spy that we injected from the component by calling `TestBed.inject(WeatherService)`, and we store it in the global variable `weatherServiceMock`, to be available for each test. Of course we need to create a different mock object before each test, otherwise the mock configuration of one test would impact the other tests. For this reason, we also need `ngOnInit()` to be called on the component only after we set up the mock, otherwise the method will still have no service available: because of this we move the call to `fixture.detectChanges()` from the second `beforeEach()` to the test itself, after the mock setup. The mock setup itself just instructs the mock `getCurrentWeather` method to return an empty `Observable`, created by calling `of()` of RxJS.

We can improve our test suite by adding another two tests. First we need to create some fake weather data, in `src/app/weather/weather.service.fake.ts`:
```typescript
import { ICurrentWeather } from '../interfaces';

export const fakeWeather: ICurrentWeather = {
  city: 'Bethesda',
  country: 'US',
  date: 1485789600,
  image: '',
  temperature: 280.32,
  description: 'light intensity drizzle',
};
```

Then we can proceed adding the two steps:
```typescript
...
import { By } from '@angular/platform-browser';
import { fakeWeather } from '../weather/weather.service.fake';
...
it ('should get currentWeather from weatherService', () => {
  weatherServiceMock.getCurrentWeather.and.returnValue(of());
  fixture.detectChanges();
  expect(weatherServiceMock.getCurrentWeather).toHaveBeenCalledTimes(1);
});

it('should eagerly load currentWeather in Bethesda from weatherService', () => {
  weatherServiceMock.getCurrentWeather.and.returnValue(of(fakeWeather));
  fixture.detectChanges();

  expect(component.current).toBeDefined();
  expect(component.current.city).toEqual('Bethesda');
  expect(component.current.temperature).toEqual(280.32);

  const debugEl = fixture.debugElement;
  const titleEl: HTMLElement = debugEl.query(By.css('span')).nativeElement;
  expect(titleEl.textContent).toContain('Bethesda');
});
...
```

The first test just asserts that the `getCurrentWeather` method is called only once by the `CurrentWeatherComponent`. The second test first prepares the service mock to return the fake weather data, and then proceeds to test that both the component logic, and the component template are correct. First we check that the `current` property of the component is populated with the correct data, and then we check that the first HTML element of type `span` contains the expected string.

We can make our code more robust by defining an interface for our Weather Service: to do this, let's define a `IWeatherService` interface in `weather.service.ts`, and let `WeatherService` implement it:
```typescript
...
export interface IWeatherService {
  getCurrentWeather(city: string, country: string): Observable<ICurrentWeather>;
}

export class WeatherService implements IWeatherService {
  ...
}
...
```

The unit test for the `WeatherService` is still failing because no `HttpClient` has been configured to be injected in the service during the test. Instead of create another mock for it, in this case we can leverage the mock that's already provided by Angular, `HttpClientTestingModule`:
```typescript
...
import { HttpClientTestingModule } from '@angular/common/http/testing';
...
TestBed.configureTestingModule({ imports: [ HttpClientTestingModule ] })
...
```


## Integration tests

The Angular CLI uses the Protractor and WebDriver libraries to run integration tests that simulate a user interacting with the application. When a new Angular application is created, the CLI automatically bootstraps a default integration test, located at `e2e/src/app.e2e-spec.ts`:
```typescript
import { AppPage } from './app.po';
import { browser, logging } from 'protractor';

describe('workspace-project App', () => {
  let page: AppPage;

  beforeEach(() => {
    page = new AppPage();
  });

  it('should display welcome message', async () => {
    await page.navigateTo();
    expect(await page.getTitleText()).toEqual('local-weather-app app is running!');
  });

  afterEach(async() => {
    // Assert that there are no errors emitted from the browser
    const logs = await browser.manage().logs().get(logging.Type.BROWSER);
    expect(logs).not.toContain(
      jasmine.objectContaining({
        level: logging.Level.SEVERE,
      }) as logging.Entry;
    );
  });
});
```

This test depends upon the "page object" `e2e/src/app.po`:
```typescript
import { browser, by, element } from 'protractor';

export class AppPage {
  navigateTo(): Promise<unknown> {
    return browser.get(browser.baseUrl);
  }

  async getTitleText(): Promise<string> {
    return element(by.css('app-root .content span')).getText();
  }
}
```

A classic issue with Web integration tests is that the Web part of an application changes very frequently, making integration tests very fragile, since they need to constantly be kept up to date with the templates. To handle this issue, here we're using the Humble Object pattern, moving all parts of the test that depend on the actual form of the template into a separate object, which is the "page object" `app.po`. This way, every time there's a change in the frontend, only the page object will need to be updated, and the actual test file will remain untouched.

Integration tests can be executed by running:
```shell
npm run e2e
```

If you run in an error like: `E/launcher - Error: Error: spawn Unknown system error -86`, you'll need to fix an inconsistency with the webdriver library on OSX. To do this, update `webdriver-manager` to the latest version:
```json
...
"webdriver-manager": "^12.1.8"
...
```

If all goes well, at the first run of the integration tests you should see an error like: `Failed: No element found using locator: By(css selector, app-root .content span)`. This is due to the fact that the original test is tailored to the predefined template of the `AppComponent` which we changed while developing our app. To fix this, we should update the page object:
```typescript
...
async getTitleText(): Promise<string> {
  return element(by.css('app-root div h1')).getText();
}
...
```

Next we'll get an error like `Expected 'LocalCast Weather' to equal 'local-weather-app app is running!'.`, because in the actual integration test we're expecting the title to contain the original string, instead of the new one we added. Change the test to:
```typescript
it('should display welcome message', async () => {
  await page.navigateTo();
  expect(await page.getTitleText()).toEqual('LocalCast Weather');
});
```


## Production release

When releasing an app to a production environment, we need to optimize the bundle size by removing redundant, unused and inefficient code, as well as pre-compiling sections of code. To build the application for production:
```shell
ng build --prod
```

In addition to this, we need to update the environment variables we're using, adding the production versions in `environment.prod.ts`.

Next, we need to configure our CI environment so that the builds are performed by calling `npm ci` (for "clean install") instead of `npm install`. `npm ci` is an installation command optimized for CI environments: in particular it deletes all current contents of `node_modules`, removing any manual change to packages or any manually-added packages, it reads directly `package-lock.json`, skipping `package.json`, it never changes any of the two, it's faster than `npm install`, and discards certain user-related packages.

In a CI environment we also need to run unit tests with coverage checking, and disabling the watch-mode:
```shell
npm test -- --code-coverage --watch=false
```

After building an Angular project, the build output is located in the `dist` folder. Everything in this folder is a static file, and as such can directly be deployed to a Web server. For example, we can add a couple of commands to automatically deploy to Vercel in `package.json`:
```json
...
"scripts": {
  ...
  "build:prod": "ng build --prod",
  "prevercel:publish": "npm run build:prod",
  "vercel:publish": "vercel --platform-version 2 dist/local-weather-app"
}
...
```


## Angular Material

Angular Material is a high-quality UI toolkit developed closely to Angular itself, thus providing optimum integration. To install Angular Material in our project:
```shell
npx ng add @angular/material
```

We can choose one of the suggested prebuilt theme, like `indigo-pink`, and to set up global Angular Material typography, and browser animations. After installation is completed, `index.html` will have been changed with new icons library and default font, like:
```html
...
<head>
  ...
  <link href="https://fonts.googleapis.com/css?family=Roboto:300,400,500&display=swap" rel="stylesheet">
  <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
</head>
```

The `angular.json` file will also have been changed with the default theme:
```json
...
"styles": [
  "./node_modules/@angular/material/prebuilt-themes/indigo-pink.css",
  "src/styles.css"
],
...
```

If we decide to change the Material base theme, we should update the theme referenced by the `styles` element above.

The `src/styles.css` will have been changed with the default global styles:
```css
html,
body {
  height: 100%;
}
body {
  margin: 0;
  font-family: Roboto, "Helvetica Neue", sans-serif;
}
```

If we chose to install animations as well, the root module will have been updated as well:
```typescript
...
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
...
@NgModule({
  ...
  imports: [
    ...
    BrowserAnimationsModule
  ],
  ...
})
```

Now we can add a dedicated module to configure everything related to Angular Material:
```shell
npx ng g m material --flat -m app
```

This will create a new root-level module, `MaterialModule`, and add it to the root module as well. In the new `MaterialModule` that has been created, we can remove the `CommonModule` import, and add some modules that we'll need:
```typescript
import { NgModule } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
import { MatToolbarModule } from '@angular/material/toolbar';

@NgModule({
  declarations: [],
  imports: [ MatButtonModule, MatToolbarModule, MatIconModule ],
  exports: [ MatButtonModule, MatToolbarModule, MatIconModule ],
})
export class MaterialModule { }
```

We can augment Angular Material with Angular Flex Layout to handle pages layout using the modern CSS3 Flexbox:
```shell
npm i @angular/flex-layout
```

and add `FlexLayoutModule` to the root module:
```typescript
...
import { FlexLayoutModule } from '@angular/flex-layout';
...
imports: [ ..., FlexLayoutModule ],
...
```

Now we can use a Material Toolbar to contain our application title: let's thus replace our `<h1>...</h1>` in the template with `<mat-toolbar color="primary"><span>...</span></mat-toolbar>`. Next we can encapsulate the various chunks of information we want to display into different Material Cards; to do this we first need to import Material Card as well in our Material module:
```typescript
...
import { MatCardModule } from '@angular/material/card';
...
imports: [ ..., MatCardModule ],
exports: [ ..., MatCardModule ],
...
```

and update our template like this:
```html
...
<div style="text-align:center">
  <mat-toolbar color="primary">
    <span>LocalCast Weather</span>
  </mat-toolbar>
  <div>Your city, your forecast, right now!</div>
  <mat-card>
    <h2>Current Weather</h2>
    <app-current-weather></app-current-weather>
  </mat-card>
</div>
```

Next, remove `style="text-align:center"` and surround `<mat-card>` with the following layout:
```html
...
<div fxLayout="row">
  <div fxFlex></div>
  <div fxFlex="300px">
    ...
  </div>
  <div fxFlex></div>
</div>
```

This way we created three slots in a single row, only the second one of the two carrying content: now, since the three slots are equally sized, the end result is that the middle one will be displayed at the center of the row.

Material Card also provides widgets for header, title and content:
```html
...
<mat-card>
  <mat-card-header>
    <mat-card-title>Current Weather</mat-card-title>
  </mat-card-header>
  <mat-card-content>
    <app-current-weather></app-current-weather>
  </mat-card-content>
</mat-card>
...
```

Next, since all Material elements natively support Flex Layout, we can configure flex directly in the Material Card:
```html
...
<div fxLayout="row">
  <div fxFlex></div>
  <mat-card fxFlex="300px">
    ...
  </mat-card>
  <div fxFlex></div>
</div>
...
```

The Material Card Title only defines the layout of the text that represents a title, but does not handle its typography. To control it, and let our title appear a bit more like a title, we can use the Material Typography classes:
```html
...
<mat-card-title>
  <span class="mat-headline">Current Weather</span>
</mat-card-title>
...
```

Now we can fix the layout of the tagline too:
```html
...
<div fxLayoutAlign="center">
  <span class="mat-caption">Your city, your forecast, right now!</span>
</div>
...
```

Now we can move to fixing the display of the Current Weather component:
```html
<div fxLayout="row">
  <div fxFlex="66%">{{ current.city }}, ...</div>
  <div fxFlex>{{ current.date | date: 'fullDate' }}</div>
</div>
<div fxLayout="row">
  <div fxFlex="66%"><img [src]='current.image'></div>
  <div fxFlex>{{ current.temperature | number: '1.0-0' }}°F</div>
</div>
<div>
  {{ current.description }}
</div>
```

At this point our App Component unit test is failing because we changed the element containing the title:
```typescript
expect(compiled.querySelector('mat-toolbar').textContent).toContain('LocalCast Weather');
```

Also our Current Weather Component unit test is failing because we changed the element containing the title:
```typescript
...
const titleEl: HTMLElement = debugEl.query(By.css('.mat-title')).nativeElement;
expect(titleEl.textContent).toContain('Bethesda');
...
```

End-to-end test are also failing because we changed the HTML element of the application title:
```typescript
...
async getTitleText(): Promise<string> {
  return element(by.css('app-root mat-toolbar span')).getText();
}
...
```

Finally, we should add automatic Accessibility tests for our application:
```shell
npm i -D pa11y pa11y-ci http-server
```

Let's add a script to run these tests to `package.json`:
```json
...
"scripts": {
  ...
  "test:a11y": "pa11y --standard Section508 http://localhost:5000"
}
...
```

On the first run we'll get one accessibility error: our `img` is missing the `alt` attribute. Let's fix it with:
```html
...
<img style="zoom: 175%" [src]="current.image" [alt]="current.description">
...
```

To enable accessibility tests on our CI pipeline, first we need to add a `.pa11yci` configuration file:
```json
{
  "default": {
    "timeout": 1000,
    "page": {
      "viewport": {
        "width": 320,
        "height": 480
      }
    }
  },
  "urls": [
    "https://local-weather-app.michelezamuner.vercel.app/"
  ]
}
```

then we need to add a CI command to `package.json`:
```json
...
"scripts": {
  ...
  "test:a11y:ci": "pa11y-ci"
}
...
```

and finally we can call this command from our CI pipeline.


## Forms

Every input field that we want to use in our Web application should be contained inside a `<form>` tag. There are two types of forms in Angular:
- *template-driven forms*: the form logic is mostly inside the template, thus making the HTML bigger and less readable, and the component harder to test
- *reactive forms*: the behavior of the forms is defined in the component class, thus making it more easily testable

To start creating reactive forms in our application, we should first import `FormsModule` and `ReactiveFormsModule` in the root module:
```typescript
...
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
...
imports: [ ..., FormsModule, ReactiveFormsModule ],
...
```

In particular, `FormsModule` is needed to create template-driven forms, while `ReactiveFormsModule` is needed to create reactive forms. Since at the beginning we just want to add a simple input field, we'll also use `FormsModule`.

Next, we're going to leverage Angular Material to get the form inputs, so we add two new modules to the Material Module:
```typescript
...
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
...
imports: [ ..., MatFormFieldModule, MatInputModule ],
exports: [ ..., MatFormFieldModule, MatInputModule ],
...
```

We now need a new component to host the functionality to search for a specific city:
```shell
npx ng g c citySearch --module=app.module
```

The new component template should be this:
```html
<form>
  <mat-form-field appearance="outline">
    <mat-label>City Name or Postal Code</mat-label>
    <mat-icon matPrefix>search</mat-icon>
    <input matInput aria-label="City or Zip" [formControl]="search">
    <mat-hint>Specify country code like 'Paris, US'</mat-hint>
  </mat-form-field>
</form>
```

In the component class we need to add a `search` property initialized to a new form control:
```typescript
...
import { FormControl } from '@angular/forms';
...
export class CitySearchComponent implements OnInit {
  ...
  search = new FormControl();
  ...
}
```

In a Angular reactive form each input is modelled as a control, via the `FormControl` class. We can also use `FormArray` to represent repetitive input fields where the same field is duplicated because each instance is linked to a different item of a list, and `FormGroup` to group together different inputs, each of which will be a `FormControl` or a `FormArray`. Angular also provides `FormBuilder` to simplify the creation and management of these inputs.

We should now add the newly created search component to the existing App component, after the tagline and before the results card:
```html
<div fxLayoutAlign="center">
  <app-city-search></app-city-search>
</div>
```

We want to allow users to search for a specific location given its zip code. Currently in our `WeatherService` we assume we'll be given a city name, so we should replace it with a generic `search` argument, which can take both a string and a number; then, in case we received a zip code we can pass it to the API as a query argument:
```typescript
...
public getCurrentWeather(search: string|number, country?: string): Observable(ICurrentWeather) {
  let uriParams = new HttpParams();
  if (typeof search === 'string') {
    uriParams = uriParams.set('q', country ? `${search},${country}` : search);
  } else {
    uriParams = uriParams.set('zip', `${search}`);
  }

  return this.sendGetCurrentWeatherRequest(uriParams);
}

private sendGetCurrentWeatherRequest(uriParams: HttpParams): Observable<ICurrentWeather> {
  uriParams = uriParams.set('appid', environment.appId);

  return this.httpClient.get<ICurrentWeatherData>(
    `${environment.baseUrl}api.openweathermap.org/data/2.5/weather`,
    { params: uriParams }
  )
  .pipe(map(data => this.transformToICurrentWeather(data)));
}
...
```

We also extracted the common code to send a generic get request to the API given a set of query arguments. We can leverage this by adding a new method to get the current weather given the latitude and longitude:
```typescript
...
export type Coordinates = {
  latitude: number,
  longitude: number,
};
...
getCurrentWeatherByCoords(coords: Coordinates): Observable<ICurrentWeather> {
  const uriParams = new HttpParams()
    .set('lat', coords.latitude.toString())
    .set('lon', coords.longitude.toString());

  return this.sendGetCurrentWeatherRequest(uriParams);
}
...
```

Since we changed the interface of our service, we need to reflect these changes to the `IWeatherService` interface definition:
```typescript
...
export interface IWeatherService {
  getCurrentWeather(search: string|number, country?: string): Observable<ICurrentWeather>;
  getCurrentWeatherByCoords(coords: Coordinates): Observable<ICurrentWeather>;
}
...
```

At this point we can use the Weather Service inside the City Search component, just to log results to the console for now:
```typescript
...
import { WeatherService } from '../weather/weather.service';
...
export class CitySearchComponent implements OnInit {
  search = new FormControl();

  constructor (private weatherService: WeatherService) { }
  ...
  ngOnInit(): void {
    this.search.valueChanges().subscribe(
      (searchValue: string) => {
        searchValue = searchValue.trim();
        if (!searchValue) {
          return;
        }

        const userInput = searchValue.split(',').map(s => s.trim());
        this.weatherService.getCurrentWeather(
          userInput[0],
          userInput.length > 1 ? userInput[1] : undefined
        ).subscribe(data => console.log(data));
      }
    );
  }
}
...
```

By subscribing to `valueChanges()` we make an API call every time the user types a character in the search input. This is obviously not ideal, so we'll have to limit the number of calls we make. To do this we'll employ throttling and debouncing, provided by the RxJS library, in the City Search component:
```typescript
...
import { debounceTime } from 'rxjs/operators';
...
this.search.valueChanges
  .pipe(debounceTime(1000))
  .subscribe(...)
...
```

This means that from the values that come from the `valueChanges` stream we only take one every second, and in addition to these, we also take the value that remains after the user stopped typing. By contrast, had we used the `throttleTime` pipe, we wouldn't have kept the last value.

Now we need to update the form control to prevent single character inputs, and to display error messages; let's update the component class:
```typescript
...
import { FormControl, Validators } from '@angular/forms';
...
search = new FormControl('', [Validators.minLength(2)]);
...
this.search.valueChanges
  .pipe(debounceTime(1000))
  .subscribe((search Value: string) => {
    if (!this.search.invalid) {
      ...
    }
  })
...
```

and the template:
```html
...
<form style="margin-bottom: 32px">
  <mat-form-field appearance="outline">
    ...
    <mat-error *ngIf="search.invalid">
      Type more than on character to search
    </mat-error>
  </mat-form-field>
</form>
...
```

The form control provides an `invalid` property that is automatically updated as soon as the validation fails, that we can use to both display the validation message, and prevent calling the API.

Currently we're only printing the API output on the console, but to update the actual widget we need the City Search component to interact with the Current Weather component.

## References

- "Angular for Enterprise-Ready Web Applications Second Edition" by Douguhan Uluca, Packt 2020
