# Writing integration tests for components with ports

When a component provides ports, it means it's designed to work with any implementation (adapter) of the ports that it defines. When we're writing integration tests for this component, then, we need to provide stubs for these adapters, in order to be able to exercise its functionality.

It might be the case that in the application where this component is defined, we're also providing default implementations for these ports: for example, we might be providing an application that is fully working out of the box, but which also allows to be extended or modified by replacing one or some of its default adapters with custom ones.

In this case we might be tempted to use the default adapters as test stubs in our integration tests, in order to avoid duplicating the effort of writing these stubs, since we're going to have those default adapters there in the first place anyway.

There are, however, reasons for it to be a better choice to write custom stubs, dedicated specifically to these integration tests:
- default implementations are bound to change overtime, either with evolving design and implementation, or with bug fixes: the integration tests will then be run against an always changing set of "stubs", where the nature of these changes is not under the control of the component itsel, and it might be the case that a future version of these stubs will introduce some bugs or weird behavior, producing false positives during the tests execution
- integration tests must be setup according to the specific way the default implementations are designed: if their design or implementation changes with time, all tests using them must be updated as well
- when a port is designed, but only one concrete adapter is ever written and used with it, we run the risk that the port is not providing the correct abstractions, or that it's actually bound in some non obvious wais to that specific implementation, so that the first time we need to write a different adapter, we discover that it's very hard to do so, because the port is not properly representing the concept we needed to model, or because it's making some assumption that only makes sense for that other implementation instead; writing dedicated test stubs will provide a first example of a different implementation of the port, thus giving us the chance of immediately testing the quality of its abstraction

For these reasons, then, it's better to provide adapter stubs that are dedicated to the integration tests, instead of reusing already existing (or bound to be created anyway) actual default implementations.
