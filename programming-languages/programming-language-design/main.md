# Programming language design

Programs written in a specific language execute according to a certain model of computation, which can be implemented in many different ways according to the targeted architecture.

The most simple programming model is the imperative model, where memory is divided into several areas, like the stored program, the stack, the heap, and possibly static data and global data. Thanks to the stack, the model can keep track of the execution of nested functions or procedures: each stack frame contains the arguments passed to the current function, its local variables, the return address, and other information. The purpose of the imperative model is to get data as input, transform it by updating memory, and thus produce output. This is the computation model that more closely resembles the underlying von Neumann architecture.

In the functional model input data is transformed by executing functions, passing arguments to them. Data is not changed, rather new data is created from old data, and there is no difference between program and data, since functions are data themselves. There is still a runtime-stack to allow function calls, and the heap is used in a much more controlled way, in order to prevent data from being changed. The functional model is very far from the underlying architecture, and it's the purpose of the language implementation to fill the gap.

In the logic computational model there is no program to begin with, but a database of facts and rules, and the language implementation tries to answer questions with yes or no. What questions are asked, and in what order, determines how the program will work.

## References

"Programming Languages" by Kent D. Lee, Springer 2008
