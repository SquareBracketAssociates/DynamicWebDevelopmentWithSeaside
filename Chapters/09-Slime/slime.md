## Writing good Seaside Code
@cha:slime

This short chapter explains how you can improve the quality of your code and your programming skills in general by running _Slime_, a Seaside-specific code critics tool. For example, Slime can detect if you do not follow the canonical forms for brush usage that we presented in Chapter XXX. It will also help you identify other potential bugs early on, and help you produce better code. Furthermore, if you work on a Seaside library, it is able to point out portability issues between the different Smalltalk dialects.

### A Seaside Program Checker


Slime analyzes your Seaside code and reveals potential problems. Slime is an extension of _Code Critics_ that is shipped with the _Refactoring Browser_. Code Critics, also called _SmallLint_, is available with the Refactoring Browser originally developed by John Brant and Don Roberts. Lukas Renggli and the Seaside community extended Code Critics to check common errors or bad style in Seaside code. The refactoring tools and Slime are available in the One-Click Image and we encourage you to use them to improve your code quality. 

Pay attention that the rules are not bulletproof and by no means complete. It could well be that you encounter false positives or places where it misses some serious problems, but it should give you an idea where your code might need some further investigation. 

Here are some of the problems that Slime detects:

**Possible Bugs.** This group of rules detects severe problems that are most certainly serious bugs in the source code:

- The message `with:` is not last in the cascade,
- Instantiates new component while generating HTML, 
- Manually invokes `renderContentOn:`,
- Uses the wrong output stream, 
- Misses call to super implementation,
- Calls functionality not available while generating output, and 
- Calls functionality not available within a framework callback.


**Bad style.** These rules detect some less severe problems that might pose maintainability problems in the future but that do not cause immediate bugs.

- Extract callback code to separate method,
- Use of deprecated API, and
- Non-standard object initialization.


**Suboptimal Code.** This set of rules suggests optimization that can be applied to code without changing its behavior. 

- Unnecessary block passed to brush.


**Non-Portable Code.** While this set of rules is less important for application code, it is central to the Seaside code base itself. The framework runs without modification on many different Smalltalk platforms, which differ in the syntax and the libraries they support. To avoid that contributors from a specific platform accidentally submit code that only works with their platform we've added some rules that check for compatibility. The rules in this category include:

- Invalid object initialization,
- Uses curly brace arrays,
- Uses literal byte arrays,
- Uses method annotations,
- Uses non-portable class,
- Uses non-portable message,
- ANSI booleans,
- ANSI collections,
- ANSI conditionals,
- ANSI convertor,
- ANSI exceptions, and
- ANSI streams.



### Summary


In this chapter, we presented tasks, subclasses of Task. Tasks are components that do not render themselves but are used to build application flow based on the composition of other components. We saw that the composition is expressed in plain Pharo.





