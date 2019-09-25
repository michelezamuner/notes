# Software deploy


## Build, deploy and distribution

The actual software systems that are used themselves by users, or other software systems, consist of so-called *artifacts*, which need to be installed on some target environment, and then be executed on it. Some examples of artifacts and environments can be:
- a JavaScript file executed on a Web browser
- a command line binary executed on a server
- a GUI application running on a desktop computer, or on a mobile device
- a Java servlet application running on a server over a servlet container
- a set of JavaScript sources running on a server over the Node virtual machine

In these examples, artifacts can be single text or binary files, archive files like JAR or WAR, or set of files with a selected entry point (main or index file).

These software systems, however, need to be created, or updated, at a certain point: development work happens on the local workstations of software developers, which then have to create new artifacts from their work, and distribute them to the target environments.

The general term used to describe this process of distribution of artifacts to target environments is *deploy*. However, here I'll refer to a slightly different terminology, for which I'm using the [Maven tool](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html) as a reference:
- *validate*: validate that the project setup and meta-information is correct (if required by the specific project)
- *compile*: compile source code into binary code (if required by the programming language)
- *test*: run unit tests on the (compiled) code; these tests must not require a properly installed instance of the system to be provided to run
- *package*: package the code in a distributable format, or artifact (if required by the target environment)
- *verify*: run integration tests against the artifact; these tests must run on a different process than the system under test itself, and thus no mock is used in the tested system
- *install*: install the artifact on the local environment for development purposes (if required by the specific project)
- *deploy*: upload the artifact to the remote artifact repository

Here we can see that *deploy* refers just to uploading the artifact to the remote artifact repository: no mention is made about how that artifact will actually be distributed to the actual target environments. This is important because the same artifact might need to be distributed to several different environments, for example a testing server to run acceptance end-to-end tests, or a demo server to show the customer: in these cases we don't want to go through the testing and packaging phases again, because it's the same artifact (i.e. the same version of the software) that we want to release, and so it's useful to have the final product already available. For this reason, the step of copying the artifact to the actual target environment will be here referred to as *distribution*.

Additionally, all steps listed above can be referred to as the *build* steps, where the purpose of the build is to provide an artifact to the artifact repository. Build steps are usually run from a development environment, for example the developer's workstation, for example by a tool like [Maven](https://maven.apache.org), while the final distribution step is performed by the artifact repository, or repository manager, like [Nexus](https://www.sonatype.com/nexus-repository-oss).


## Components and deploy units

A typical issue with the build process is that for bigger systems running the compilation, packaging and testing for a whole system can take quite a bit of time. Even if we're using a programming language that doesn't require compilation (a typical long process), still running unit and integration tests for a whole project can take a lot. To mitigate this problem, we need to manage to split the system into multiple smaller parts that can be deployed separately, thus maintaining the "weight" of every deploy as small as possible.

We already strive to design our systems so that they're based on decoupled *components*. When components are as independent as possible, and talk to each other only through limited interfaces, it's much easier to understand the scope and design of each component, to write tests for each components, etc.

Decoupled components, additionally, are also very useful when it comes to deployability. If a component is independent enough, we can work on it, test it, package it and deploy it as a single *deploy unit*, which can then be picked up by the repository manager and injected into systems that are already distributed. Of course systems need to be architected so that this kind of injection is allowed: if a system artifact is a single text or binary file, or a single archive file, this won't be possible, unless something like dynamic libraries at the level of the operating system are used.

A lot of commonly used programming languages, though, produce systems that are just sets of files organized in directories, that are then compiled into some bytecode and run by a virtual machine, with some caching going on perhaps: in this situation, it'd be very easy to replace just a single sub-directory, as a specific deploy unit. This could be done using native package managers too, to stay within the standards. For statically compiled languages, dynamic libraries would probably be the way to go instead. On yet another hand, if the deploy model is based on microservices, it's possible that each deploy unit can fit a single microservice, and then each unit will be distributed on its own process, avoiding these kinds of issues.


## End-to-end acceptance tests

While each component comes with its own unit and integration tests, we usually still need to be able to test the whole integrated system with all the components in place, from the perspective of an outside user. These tests are by their nature specific of the actual application they're targeting: this means that different components are usually independent enough to be reusable on different applications, and a specific application will be a special arrangement of reusable components, and specific ones, thus needing its own set of external tests to verify the correctness of the arrangement.


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
