# Software deploy


## Build, deploy and distribution

Software systems consist of artifacts, that need to be installed on target environments in order to be executed. Some examples of artifacts and environments can be:
- a JavaScript file executed on a Web browser
- a command line binary executed on a server
- a GUI application running on a desktop computer, or mobile device
- a Java servlet application running on a servlet container
- a set of JavaScript sources running on a server over the Node virtual machine

In these examples, artifacts can be single text or binary files, archive files like JAR or WAR, or set of files with a selected entry point (main or index file).

These software systems, however, need to be created, or updated, at a certain point: development work happens on the local workstations of software developers, who then have to create new artifacts from their work, and distribute them to the target environments. The definition of a software system resides in a *project*, which is a set of files including sources, assets, configuration, etc. The *build* process is responsible to create a *package* from a project, according to specific build instructions.

Builds can usually take quite a bit of time, including compilation, static analysis, unit and integration tests, etc.: to avoid re-running long builds only to obtain the same result, packages are stored in *artifact repositories* or *repository managers*, like [Nexus](https://www.sonatype.com/nexus-repository-oss), tagged with a version, so that packages created in different moments of time can be identified. A package that is versioned and stored in an artifact repository is called *artifact*. The build process can be performed locally by build tools like [Maven](https://maven.apache.org), or within a CI environment, like [Jenkins](https://jenkins.io/).

A complete software system is usually composed of several different artifacts installed together in the same environment. Systems are split into different artifacts, first because independent components can be kept more simple, and easier to work with, and second because if components are properly decoupled, changes will only impact a limited amount of components, making building a new version of the application much faster, because it'll only require re-building a few components instead of all of them. A set of artifacts representing a software system at a certain point of time is called a *release*: releases are versioned and stored as single artifacts in the artifact repository as well, in order to simplify the installation of the same release in different environments. The installation of a release to a specific environment is called *distribution*.

Applications are usually configurable, in order to allow modifying how the application works without complex changes to the code. We can identify the following types of configuration:
- *package configuration*: describes how the application's units should be laid out at the package level, e.g. components configuration for Dependency Injection; requires a rebuild to change, and is the same for all environments
- *application configuration*: describes of application's features should behave, e.g. business logic configuration; requires a new release to change, and is the same for all environments
- *environment configuration*: contains environment-specific information, e.g. database password; requires re-distribution to change, and varies per environment

Package configuration is useful because it allows to reuse generic components, like Dependency Injection Containers, in multiple different applications. However, package configuration doesn't make the application more flexible, since changing this configuration requires a full rebuild and redistribution. Package configuration is stored with the project, and thus will be the same for different applications built out of the same project, and for all environments where the application will be distributed.

Application configuration is used to select specific behaviors of the application features. By using different application configurations, we can create different software systems, behaving differently, out of the same project. Application configuration is meant to allow to easily tweak the application behavior, and as such it doesn't require a rebuild, only the creation of a new release out of the same artifacts, and a redistribution.

Finally, environment configuration is needed to allow the application to communicate with specific environments, since different environments have different characteristics. Environment configuration can be easily changed, and doesn't require a new build, nor a new release, only a new redistribution, to install the same release again on the changed environment. 


## Development phases and steps

### Development

Development happens in the developers' local environments. In general, development can involve all the steps of the other phases as well, because while developing it's necessary to have a development version of the artifacts running on the local environment; so there will be integration, release and distribution also happening locally, in a special way. Here we list the steps that happen only on the development phase though: 
- write code
- VCS
- documentation

### Integration

Integration has the purpose of producing a single, well-working software artifact. This is usually done during Continuous Integration. It's specific steps are:
- fetch project files
- install dependencies (including development dependencies)
- run static analysis
- compile/transpile
- run unit tests
- create package (removing development dependencies and any non-package related project file)
- run integration tests
- tag package
- store package to artifact repository

### Release

The release phase has the purpose of producing the final, complete application, by gathering all needed artifacts, and wiring them together, adding the specific application configuration. Different applications might share some artifacts, and that's why it's important to distinguish releases from artifacts. The release steps are:
- fetch artifacts from artifact repository
- add application configuration
- install in the test environment
- run end-to-end tests
- create release
- tag release
- store release to artifact repository

### Distribution

The distribution phase is the final one, where we install a specific release version on a specific environment, adding the related environment configuration. The steps are:
- fetch release from artifact repository
- add environment configuration
- install release to environment


## Project files

Each development phase requires some project files that are specific to it.

- development
    - `src/` source files
    - `doc/` documentation
    - `.gitignore` VCS configuration
    - `README.md` documentation
- integration
    - `test/` unit and integration test files
    - `dist/` actual package files
    - `lib/`  dependencies
    - `conf/` configuration
    - `bin/` binaries
    - `res/` assets
    - `deps.conf` dependency management configuration
    - `lint.conf` static analysis configuration
    - `build.conf` build configuration
- release
    - `features/` user stories
    - `acceptance/` end-to-end tests
    - `app.conf` application configuration
- distribution
    - `env.conf` environment configuration

Now, from the same project files different applications can be built, with different configurations. For this reason, application configuration and environment configuration should not be stored along with the project files, which are always the same for all releases and instances. The integration process might copy configuration files from the repositories where they are stored, into the `conf/` directory of the package, if that's what the application expects to work.

Additionally, while each component comes with its own unit and integration tests, we usually still need to be able to test the whole integrated system with all the components in place, from the perspective of an outside user. These tests are by their nature specific to the actual application they're targeting: this means that different components are usually independent enough to be reusable on different applications, and a specific application will be a special arrangement of both reusable components, and specific ones, in addition to special application configuration, thus needing its own set of external tests to verify the correctness of the arrangement.

We thus need at least two projects: one to build the packages, and another to maintain all different applications and environments configurations, in addition to end-to-end tests. Of course more than two projects can be used if needed.

```
project/
    src/
    test/
    dist/
    lib/
    doc/
    conf/
    bin/
    res/
    .gitignore
    README.md
    deps.conf
    lint.conf
    pkg.conf
```
```
app1/
    features/
    acceptance/
    app.conf
    env1.conf
    env2.conf
    ...
app2/
    ...
```


## Components and deploy units

A typical issue with the build process is that for bigger systems running the compilation, packaging and testing for a whole system can take quite a bit of time. Even if we're using a programming language that doesn't require compilation (a typical long process), still running unit, integration and end-to-end tests for a whole project can take a lot. To mitigate this problem, we need to manage to split the system into multiple smaller parts that can be deployed separately, thus maintaining the "weight" of every deploy as small as possible.

We already strive to design our systems so that they're based on decoupled *components*. When components are as independent as possible, and talk to each other only through limited interfaces, it's much easier to understand the scope and design of each component, to write tests for each components, etc.

Decoupled components, additionally, are also very useful when it comes to deployability. If a component is independent enough, we can work on it, test it, package it and deploy it as a single *deploy unit*, which can then be picked up by the repository manager and injected into systems that are already distributed. Of course systems need to be architected so that this kind of injection is allowed: if a system artifact is a single text or binary file, or a single archive file, this won't be possible, unless something like dynamic libraries at the level of the operating system are used.

A lot of commonly used programming languages, though, produce systems that are just sets of files organized in directories, that are then compiled into some bytecode and run by a virtual machine, with some caching going on perhaps: in this situation, it'd be very easy to replace just a single sub-directory, as a specific deploy unit. This could be done using native package managers too, to stay within the standards. For statically compiled languages, dynamic libraries would probably be the way to go instead. On yet another hand, if the deploy model is based on microservices, it's possible that each deploy unit can fit a single microservice, and then each unit will be distributed on its own process, avoiding these kinds of issues.


## Example application

Let's consider a generic application implementing the ports-and-adapters architecture, these could be its deployable components:
```
org/app:0.0.1
    org/app-context:0.0.1
        org/app-context-domain:0.0.1
        org/app-context-usecase:0.0.1
        org/app-context-client:0.0.1
            org/app-context-client-main:0.0.1
            org/app-context-client-adapter:0.0.1
                org/app-context-client-adapter-presenters:0.0.1
                org/app-context-client-adapter-controllers:0.0.1
                org/app-context-client-adapter-views:0.0.1
        org/app-context-driver1:0.0.1
        org/app-context-driver2:0.0.1
```

Here `org` represents the organisation name, used as a namespace for package names, `app` refers to the whole application that is being built. Following DDD, we assume the application is partitioned in different bounded context, each providing a cohesive set of functionality: one of these would be `context`. Each context will bring some `domain`'s, some `usecase`'s, some `client`'s and some `driver`'s. A client typically contains a `main` module, and sets of `presenters`, `controllers` and `views`.

When these packages are designed, the Inversion of Control Principle is stressed, so that more abstract modules never depend on more concrete ones. For example, the `presenters` module defines some `View` interfaces, whose implementations will be contained in `views`; or again, the `usecase` module will define some `Presenter` interface, that will be implemented by the `presenters` module.

This way of decoupling modules is useful for maintainability and testing, but it is indeed fully leveraged the moment we actually deploy these different modules as separate units: for example, if `presenters` is completely independent from `views`, and they also are two distinct deploy units, the moment we need to update a view, we can develop, test and package only the `views` module, and push only that one to the artifact repository, with a new version: the repository manager, when doing the distribution of the application, will then pick up the new available version for `views`, for example according to Semantic Versioning. This way we reduce the build and distribution time down to really small amounts.
