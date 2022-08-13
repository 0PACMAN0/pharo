# The Pharo Refactoring Engine 

## About
The Refactoring Engine was originally developed by Don Roberts and John Brant for VisualWorks. It was ported to Squeak and Pharo and saw multiple evolution by multiple contributors.

One goal of this engine was to easily include the code refactoring into the standard development workflow. The refactoring operations help to transform and restructure source code, without to much manual intervention. And without the need to retest every single change.
In addition, some basic primitive refactoring are provided, in a way that more complex operations can be constructed by compositing the primitive ones.

The Tool, or a browser with refactoring support or the whole framework is often just known as the 'Refactoring Browser'. 
That's why all of the classes of this framework start with the prefix 'RB'.

## Overview
The following sections give an overview of the refactoring engine, the collaborating classes and components.
We will present an explanation with some examples, how to manually construct and execute refactoring operations, usable for those operation that aren't supported as actions in the default code browser.

A more in-deep description of the components will show that the refactoring engines is not only useful for code refactoring, but it also provides a powerful general purpose search and rewrite engine. This search capability - search for code "patterns" - is used to detect common program errors or just bad code style by the Pharo code critics. 

## Engine architecture

The architecture of the engine is the following: It builds a representation of the program to be refactored (instances of RBClass, RBMethod...), some refactorings complement it with Abstract Syntax Trees.
The refactorings checks their preconditions (for applicability validation or behavior preservation) against such program representation.
When the preconditions expressed as RBConditions are true, the refactoring is executed. It does not directly modify Pharo code, but produces a list changes - Such changes can be presented to the developer for validation or modification. On approval the changes actually performs the program changes.

## Core Components

Here is a brief overview of the core components.

- `RBScanner` and `RBParser`. The `RBScanner` and `RBParser` are used by Pharo to create an abstract syntax tree (AST) from the methods source code.
- `RBProgramNode` and subclasses. These are the base and concrete subclasses for all RB-Nodes representing a syntax node class, like `RBMethodNode`, `RBAssignmentNode`, et cetera.
- `RBParseTreeSearcher` and `RBParseTreeRewriter`. Some refactoring operations use the tree searcher and rewriter for applying a transformation on the abstract syntax tree. They implement a program node visitor.
- `RBClass`, `RBMetaclass`, `RBMethod`. Class and Method meta-models representing a class or method created, removed or modified during a refactoring
operation.
- `RBNamespace`. A namespace is an environment for resolving class and method entities by name and collects all changes resp. changed entities.
- `RBRefactoring` and subclasses. Abstract base classes and its concrete subclasses for refactoring operations. Every basic refactoring operation is implemented as a subclass of the `RBRefactoring` class. A refactoring operation checks the precondition that must be fulfilled and implements the actual code transform method.
- `RBCondition`. Instead of implementing conditions and condition checking code into every single refactoring operation, the RBCondition class implements a set of common tests and can be created and combined to realize a composition of conditions.
- `RBRefactoryChange`. Applying a refactoring within a namespace collects changes without applying the actual change to the system. These changes are represented by `RBRefactoryChange` subinstances and a composition of refactory changes. 


## Default refactorings exposed to the user

Most refactoring operations fall into one of three categories: class, method and method source level refactorings. Depending on the kind of Refactoring (Class/Method/Source) you can find and apply the refactoring from the "Refactoring" Menu on the class-, method- or code-pane.

Executing a refactoring will open a changes browser that lets you see and (depending on the current operation) select which change to apply.
If the refactoring can not be applied, because one of its preconditions aren't met (for example you try to rename a class with a name that is already used), a warning message appears. The refactoring changes will be applied to all classes and methods in the current "namespace". For the default system browser, this is the whole system. If you want to restrict the operation to some set of classes or packages, you can open a system browser on a refactoring environmet - a kind of "scoped view".

Example:

```
(RBBrowserEnvironment default forPackageNames: {'Kernel'}) browse. 
```

will open a system browser with only the classes from package 'Kernel'. And all refactoring operations will only find and change classes in this selection.

## Class Refactorings
This is the chapter of the refactoring help book about the class refactoring available in the System Browser.
#### Rename
I am a refactoring for renaming a class.

My preconditions verify, that the old class exists (in  the current namespace) and that the new class name is valid and not yet used as a global variable name 

The refactoring transformation will replace the current class and its definition with the new class name. And update all references in all methods in this namespace, to use the new name. Even the definition for subclasses of the old class will be changed.

Example

```
(RBRenameClassRefactoring 
	rename: 'RBRenameClassRefactoring' 
	to: 'RBRenameClassRefactoring2') execute
```

#### Remove
I am a refactoring for removing classes. 

My precondition verifies that the class name exists in this namespace and the class has no references, resp. users, if this is used to remove a trait.

If this class is "empty" (has no methods and no variables), any subclass is reparented to the superclass of this class. It is not allowed to remove non-empty classes when it has subclasses.

#### Remove keeping subclasses
I am a refactoring for removing classes but keeping subclasses in a safe way. 

My precondition verifies that the class name exists in this namespace and the class has no references, resp. users, if this is used to remove a trait.

If this class is "not empty" (has methods and variables), any subclass is reparented to the superclass of this class, and all its methods and variables (instance and class) are push down in its subclasses.

Example
```
(RBRemoveClassKeepingSubclassesRefactoring classNames: { #RBTransformationRuleTestData1 }) execute. 
```

#### Generate Accessors

Generates setter and getter methods for every instance variable defined in this class. The user is provided with a changes browser dialog allowing to select and unselected the creation of single methods. The name for the accessors is auto-generated by the instance variable name. If a method with this name already exists, the new name will have a incremented counter.

#### Generate Superclass

Adds a new superclass between a class and its previous superclass. It offers a check-box list of the subclasses. Every checked element will be moved to be a subclass of the new superclass. That is, subclasses are moved to siblings of its prior superclass. Every instance variable shared by the new siblings will be moved to the new superclass.
The name of the new class needs to be a valid class name, not yet used as any global identifier.

#### Insert Superclass

Similar to "Generate Superclass", but just generates a new class between a class and its previous superclass. With no change on the subclass hierarchy of the class.
The name of the new class needs to be a valid class name, not yet used as any global identifier.

#### Insert Subclass

Just generate a new class between this class and its subclasses.
The name of the new class needs to be a valid class name, not yet used as any global identifier.

#### Generate Subclass

Adds a new class as a subclass of the selected class. It offers a check box list of the subclasses. Every checked element will be moved to be a sub-subclass of the new subclass. The hierarchy of the not checked elements is unchanged and they become siblings of the new class.
The name of the new class needs to be a valid class name, not yet used as any global identifier.

#### Realize
Complete the set of defined methods of this class, by generating a "self shouldBeImplemented" method for all abstract methods defined in its superclass hierarchy. Where an abstract method is a method sending "self subclassResponsibilty.
Shows a warning if this class has abstract methods on its own.

#### Split
I am a refactoring for extracting a set of instance variables to a new class.

You can choose which instance variables should be moved into the new class. The new class becomes an instvar of the original class and every reference to the moved variables is replaced by a accessor call.

My precondition verifies that the new instance variable is a valid variable name and not yet used in this class or its hierarchy
 the name of the new class representing the set of instance variables is a valid class name

Example:
In the following class the variables color/font/style should be moved to a new `TextAttributesClass`.

```
Object subclass: #TextKlass
	instanceVariableNames: 'text color font style'
	classVariableNames: ''
	package: 'TestKlasses'
```	

We apply the Split Refactoring with this three variables and select a new class name TextAttributes used as variable new "textAttributes".
The class definition will be changed to:

```
Object subclass: #TextKlass
	instanceVariableNames: 'text textAttributes'
	classVariableNames: ''
	package: 'TestKlasses'
```
	
and every reference to the old vars color / font / style will be replaced by textAttributes color / textAttributes style / textAttributesFont

#### Class and Instance Variable Refactorings

##### Add Variable
I am a refactoring for adding new instance variables.

My precondition verifies that the variable name is valid, not yet used in the whole hierarchy and not a global name.

##### Rename

Shows a list of variables from the class or instance side. The selected variable is renamed in the class definition and in all methods referring to this var. The name of the accessor methods are unchanged.

##### Remove

Shows a list of variables from the class or instance side. The selected variable is removed. If the variable is referred by a method, it asks for opening a browser window, showing only those classes and its methods accessing this variable (a scoped browser view).

##### Abstract

Shows a list of variables from the class or instance side, creates an accessors for the variable and replaces all direct access to this variable by this accessors method.
(For this class and all of its subclasses.)
There is no special handling for already existing accessors methods, their direct access is replaced too. And if an accessors method with the name of this variable already exists, the newly created method will get the same name with a counter suffix.

##### Accessor

Accessor
Choose one instance / class variable to create accessors for. 
A getter and setter methods is generated, with the name of the chosen instance variable. If a method with this name already exists, it will create a new method with the same name and a counter suffix.

##### Accessor with lazy initialization
I am a refactoring for creating accessors with lazy initialization for variables.

I am used by a couple of other refactorings creating new variables and accessors.

My precondition is that the variable name is defined for this class.

Example

```
(RBCreateAccessorsWithLazyInitializationForVariableRefactoring 
	variable: 'foo1' 
	class: RBLintRuleTestData 
	classVariable: false 
	defaultValue: '123') execute
```

After refactoring we get:
```
RBLintRuleTestData >> foo1 
	^ foo1 ifNil: [foo1 := 123]
	
RBLintRuleTestData >> foo1: anObject
	foo1 := anObject
```
##### Move to class

Only for instance variables. A class search dialog lets you choose the target class to move the instance variable to.
Another dialog for choosing the instance variable to move. If there are any methods referring to this variable, a message list opens, showing all broken methods.

##### Pull up

Moves an instance/class variable up to the superclass. The variable is added to the superclass and removed from this and all other sibling classes, defining this variable. A warning message appears if not all direct subclasses defined this variable.

##### Push down

Moves an instance/class variable down to the subclasses. The variable is added to every direct subclass.
A warning dialog appears if there are methods referring to this class (accessors methods for example), and offers a choice to open a (scoped) browser for this messages.
No accessors method will be changed or generated.

##### Merge variable
I am a refactoring for merge an instance variable into another.

I replace an instance variable by other, in all methods refering to this variable and rename the old accessors, then if the instance variable renamed is directly defined in class it is removed.

My precondition verifies that the new variable is a defined instance variable in class.

Example

```
(RBMergeInstanceVariableIntoAnother rename: 'x' to: 'y' in: Foo) execute.
```

Before refactoring:
```
Class Foo -> inst vars: x, y 

Foo >> foobar
	^ x 

Foo >> foo
	^ x + y 
```
After refactoring merging X into Y
```
Class Foo -> inst vars: y 

Foo >> foobar
	^ y

Foo >> foo 
	^ y + y
```

## Method Refactorings

#### Add parameter
I am a refactoring operations for adding method arguments.

You can modify the method name and add an additional keyword argument and the default value used by senders of the original method. Only one new argument can be added. But you can change the whole method name, as long as the number of argument matches.

For example, for `r:g:b:`  add another parameter "a" the new method is `r:g:b:a:`
or change the whole method to `setRed:green:blue:alpha:`

This refactoring will 
- add a new method with the new argument, 
- remove the old method (for all implementors) and 
- replace every sender of the prior method with the new one, using the specified default argument.

#### Deprecate
I am a refactoring for deprecate a method.

My preconditions verify, that the old selector exists (in  the current namespace) and that the new selector is a valid selector

The refactoring transformation will add the call to the #deprecated:on:in: method 

Example

```
(RBDeprecateMethodRefactoring 
	deprecateMethod: #called:on: 
	in: RBRefactoryTestDataApp 
	using: #callFoo) execute
```

Before refactoring:
```
RBRefactoryTestDataApp >> called: anObject on: aBlock 
	Transcript
		show: anObject printString;
		cr.
	aBlock value
```

After refactoring:
```
RBRefactoryTestDataApp >> called: anObject on: aBlock 
	self
		deprecated: 'Use #callFoo instead'
		on: '16 April 2021'
		in: 'Pharo-9.0.0+build.1327.sha.a1d951343f221372d949a21fc1e86d5fc2d2be81 (64 Bit)'.
	Transcript
		show: anObject printString;
		cr.
	aBlock value
```
#### Inline parameter
I am a refactoring for removing and inlining method arguments.

If all callers of a method with arguments, call that method with the same literal argument expression, you can remove that argument and inline the literal into that method.

My precondition verifies that the method name without that argument isn't already used and that all callers supplied the same literal expression.

For example, a method foo: anArg

```
foo: anArg
	anArg doSomething.
```

and all senders supply the same argument: 	     

```
method1
	anObject foo: 'text'.

method2
	anObject foo: 'text'.
```	
the method argument can be inlined:

```
foo
 | anArg |
 anArg := 'text'.
	anArg doSomething.
```

and the callers just call the method without any arguments:

```
method1
	anObject foo.
```
#### Inline target sends
I am a refactoring for inlining code of this method.

The call to this method in all other methods of this class is replaced by its implementation. The method itself will be removed.

For example, a method 

```
foo
	^ 'text'
```	
is called in

```
baz
	| a |
	a := self foo.
	^ self foo.
```	
inlining in all senders replaces the call to method foo, with its code:

```
baz
	| a |
	a := 'text'.
	^ 'text'.
```

#### Move
I am a refactoring for moving a method from the class to one of its instance variable objects.

Moving a method moves it implementation to one or more classes and replaces the implementation in the original method by a delegation to one of the classes instance variable. 

I expect an option for selecting the type (classes) to which this method should be added.
A role typer RBRefactoryTyper is used to guess the possible classes used for this instance variables.
And an option for requesting the new method selector.

For all selected classes a method implementing the original method is created, and if the original code uses some references to self, a parameter needs to be added to provided the former implementor.

For example, moving the method #isBlack from class Color to its instvar #rgb for the type "Integer" creates a method 


```
Integer >> isBlack
	 ^ self = 0
```
and changes Colors implementation from: 

```
Color >> isBlack
	^ rgb = 0
```

to:

``` 
Color >> isBlack
	^ rgb isBlack
```

#### Move to class side / instance side
Move a method from the class to the instance side, or vice versa. Normally this is not considered to be a refactoring.

Only instance methods with no instance variable access or class methods with no class instance variable access can be moved.

#### Move to class side
I'm a refactoring to move a method to class side.

My preconditions verify that the method exists and belongs to instance side.

I catch broken references (method senders and direct access to instVar) and fix them.

Example

```
(RBMoveMethodToClassSideRefactoring 
	method: (RBTransformationRuleTestData >> #rewriteUsing:) 
	class: RBTransformationRuleTestData) execute.
```
Before refactoring:
```
RBTransformationRuleTestData >> rewriteUsing: searchReplacer 
     rewriteRule := searchReplacer.
     self resetResult.
```
After refactoring:
```
RBTransformationRuleTestData >> rewriteUsing: searchReplacer
     ^ self class rewriteUsing: searchReplace.

RBTransformationRuleTestData class >> rewriteUsing: searchReplacer
    | aRBTransformationRuleTestData |
    aRBTransformationRuleTestData := self new.
    aRBTransformationRuleTestData rewriteRule: searchReplacer.
    aRBTransformationRuleTestData resetResult.
```

#### Push up
I am a refactoring for moving a method up to the superclass. 

My precondition verify that this method does not refere to instance variables not accessible in the superclass. And this method does not sends a super message that is defined in the superclass.
If the method already exists and the superclass is abstract or not referenced anywhere, replace that implementation and push down the old method to all other existing subclasses.

#### Push down
I am a refactoring for moving a method down to all direct subclasses.

My preconditions verify that this method isn't refered  as a super send in the subclass. And the class defining this method is abstract or not referenced anywhere.

#### Remove Method
I am a refactoring for removing a method.

My preconditions verify that this method is not referenced anywhere.
#### Remove parameter
I am a refactoring for removing (unused) arguments.

My preconditions verify that the argument to be removed is not referenced by the methods and that the new method name isn't alread used.
Any sender of the prior selector will be changed to the new.

If the method contains more than one argument, I request the user to choose one of the arguments.

#### Remove all senders
I am a refactoring to remove all possible senders from a method (you cannot remove those calls where the result of the method call is used or when the method name symbol is referenced).

Example
```
| refactoring options |
refactoring := RBRemoveSenderRefactoring 
			remove: (90 to: 105) "node position to be removed "
			inMethod: #caller1
			forClass: RBRefactoryTestDataApp.
options := refactoring options copy.
options at: #inlineExpression put: [:ref :string | false].
refactoring options: options.
refactoring execute.
```

Before refactoring:
```
RBRefactoryTestDataApp >> caller1
	| anObject |
	anObject := 5.
	self called: anObject + 1
		on1: 
			[:each | 
			each printString.
			^anObject]
```

After refactoring (notice that the call to printstring was removed):
```
RBRefactoryTestDataApp >> caller1
	| anObject |
	anObject := 5.
	self called: anObject + 1
		on1: 
			[:each | 
			^anObject]
```
#### Rename method (all)
I am a refactoring operation for renaming methods.

The new method name has to have the same number of arguments, but the order of arguments can be changed.

My preconditions verify that the number of arguments is the same and that the new method name isn't already used.

All references in senders of the old method are changed, either the method name only or the order of the supplied arguments.

Example

There are two ways to rename a method, one of them is rename all senders of method:
```
(RBRenameMethodRefactoring 
		renameMethod: ('check', 'Class:') asSymbol
		in: RBBasicLintRuleTestData
		to: #checkClass1:
		permutation: (1 to: 1)) execute.
```
And the other is rename the method only in specific packages:
```
|refactoring|
refactoring :=RBRenameMethodRefactoring 
		renameMethod: ('check', 'Class:') asSymbol
		in: RBBasicLintRuleTestData
		to: #checkClass1:
		permutation: (1 to: 1).
refactoring searchInPackages:  #(#'Refactoring-Tests-Core').
refactoring execute
```
#### Replace by another
I'm a refactoring operation for replace one method call by another one.

The new method's name can have a different number of arguments than the original method, if it has more arguments a list of initializers will be needed for them.

All senders of this method are changed by the other.

Example

```
(RBReplaceMethodRefactoring  
	model: model
	replaceMethod: #anInstVar:
	in: RBBasicLintRuleTestData
	to: #newResultClass: 
	permutation: (1 to: 1)
	inAllClasses: true) execute
```
## Source Refactorings

#### Create cascade
I am  a refactoring used to generate cascades in source code.

Two or more message sends to the same object are replaced by a cascaded message send. It expects a selection of the messages and the receiver variable.

#### Extract method
I am a refactoring for creating a method from a code fragment.

You can select an interval of some code in a method and call this refactoring to create a new method implementing that code and replace the code by calling this method instead. 
The new method needs to have as many arguments as the number of (temp)variables, the code refers to.

The preconditions are quite complex. The code needs to be parseable valid code. 
#### Extract method to component
I am a refactoring for extracting code fragments to a new method. 

Similar to `RBExtractMethodRefactoring`, but you can choose to which component (instance or agument variable) the new method is added. 
As such, the new method arguments will include an additional argument for the sender.
Based on the instance variable you chosed for this method I will guess the class where to add this method, but you can change this class or add more classes.

#### Extract to temporary
Add a new temporary variable for the value of the selected code. Every place in this method using the same piece of code is replaced by accessing this new temporary variable instead.
As the code is now only evaluated once for initializing the variable value, this refactoring may modify the behavior if the code statements didn't evaluate to the same value on every call.

My preconditions verify that the new temporary name is a valid name and isn't already used (neither a temporary, an instance variable or a class variable).

#### Inline method
I am a refactoring for replacing method calls by the method implementation.

You can select a message send in a method and refactoring this message send to inline its code.
Any temporary variable used in the original message send is added  into this method and renamed if there are already variables with this name.

My preconditions verify that the inlined method is not a primitive call, the method does not have multiple returns. I'll show a warning if the method is overriden in subclasses.


#### Inline method from component
I am a refactoring for replacing method calls by the method implementation.

Just like `RBInlineMethodRefactoring`,  I replace a message send by the implementation of that  message , but you can provide the component
where this implementation is taken from or choose one if there are move than one implementors.
If the method implementation has some direct variable references, accessor for this variable are created (just as by the generate accessor refactoring).
#### Inline temporary
I am a refactoring to replace a temporary variable by code.

All references to the temporary variable in this method are replaced by the value used to initialize the temporary variable. 
The initialization and declaration of this variable will be removed. You need to select the variable and its initial assignment code to apply this refactoring.
#### Move variable definition
I am a refactoring for moving the definition of a variable to the block/scope where it is used.

For a method temporary variable declared but not initialized in the method scope and only used within a block, the definition can be moved to the block using this variable.
#### Rename temporary/parameter
I am a refactoring for renaming temporary variables.
This can be applied to method arguments as well.

The variable declaration and all references in this method are renamed.

My precondition verifies that the new name is a valid variable name and not an existing instance or a class variable name
#### Split cascade
I am a refactoring splitting a cascade message send to multiple messages.

You can select an interval containing a cascade expression. The refactoring will split this expression to two message sends to the receiver. 

My preconditions verify that the selector containing the cascaded message send is defined in this class, and a cascade message can be found.

If the receiver of the cascade expression is a literal or the return value of another message send, I will add another temporary variable for the interim result.
#### Move temporary to instvar
I am a refactoring for changing a temporary variable to an instance variable.

My preconditions verify that this variable is not yet used as an instance variable in this class.

The temporary variable is added to the class definition and removed from the temporary declaration in this method .

If this instance variable is already used in a subclass it will be removed from that class, because subclasses already inherit this attribute.

The temporary variables with the same name in hierarchy will be removed, and replaced with the new instance variable.

Example
--------------------

Script refactoring:
```
(RBTemporaryToInstanceVariableRefactoring 
    class: MyClassA
    selector: #someMethod
    variable: 'log') execute
```
Before refactoring:
```
Object subclass: #MyClassA
	instanceVariableNames: ''
	classVariableNames: ''
	package: 'example'

MyClassA >> someMethod 
    |log aNumber|
    log := self newLog.
    log isNil.
    aNumber := 5.

MyClassA >> anotherMethod
    #(4 5 6 7) do: [:e | | log |
        log := e ]

MyClassA subclass: #MyClassB
	instanceVariableNames: 'log'
	classVariableNames: ''
	package: 'example'
```
After refactoring:
```
Object subclass: #MyClassA
	instanceVariableNames: 'log'
	classVariableNames: ''
	package: 'example'

MyClassA >> someMethod 
    | aNumber |
    log := self newLog.
    log isNil.
    aNumber := 5.

MyClassA >> anotherMethod
    #(4 5 6 7) do: [:e | 
        log := e ]

MyClassA subclass: #MyClassB
	instanceVariableNames: ''
	classVariableNames: ''
	package: 'example'
```
## Package Refactorings


#### Rename

I'm a refactoring to rename a package. My preconditions verify that the new name is different from the current package name and is a valid name.

I change all the references of the classes that are defined within the package, and if there is a manifest, it is updated with the new name of the package. 

Example

```
(RBRenamePackageRefactoring 
				rename: (self getPackageNamed: #'Refactoring-Tests-Core')
				to: #'Refactoring-Tests-Core1') execute.
```


## Refactoring Examples
This section we show how to manually use the refactoring operations.

### Overview

This chapter describes the steps needed to set up an environment and fill in the data
to execute refactoring operations.

This enables you to execute refactorings that aren't provided by the System Browser, or combining a set of operations for a more complex refactoring. It also gives a hint on how to add refactoring support for your own tools.


These are the steps used for all of the examples:
- create a RBNamespace
- instantiate the refactoring operation
- execute it (primitiveExecute)
- open a changes browser to view and apply the changes
  applying the changes will actually execute the code transformations 'for real'.

### A first example

We want to add a new class with the RBAddClassRefactoring. (This is just a simple example, most of the time you won't use a refactoring operation for adding new classes. But it is used by tools generating classes or by other refactoring operations (RBSplitClassRefactoring)).

First we need a namespace, a RBNamespace, it collects the changes generated by this operation and provides an environment for finding other classes / methods affected by the operation.
We create a 'default' RBNamespace that represents an environment of all system classes. The RBAddClassRefactoring needs all the information needed for the class hierarchy (name/superclass/subclasses/category) and our namespace as the 'model'. The ChangesBrowser lists all the refactoring changes in a check box list. The reason for calling 'changes changes' not the model is, because the first 'changes' does not give a list of all changes but a RBCompositeRefactoryChange that actually holds the list of all changes.

  | model addClassRB browser |
    model := RBNamespace new.
    addClassRB := RBAddClassRefactoring
        model: model
        addClass: #SomeClass
        superclass: #Object
        subclasses: {}
        category: #Category.
    addClassRB primitiveExecute.
    browser := ChangesBrowser changes: (model changes changes ).
    browser open

In the ChangesBrowser list of changes you can select which one to apply. Keep in mind that some compound refactorings may not show all intermediate changes.

The primitiveExecute method will check all preconditions for this Refactoring and either shows a warning or a refactoring error, if this operation can not be performed.
We can execute the above refactoring 'twice' and will see the second time it shows an error about SomeClass already exists.

There is a global change manager - RBRefactoryChangeManager, we can use it to undo the last operation.

RBRefactoryChangeManager instance undoOperation.
and again redo
RBRefactoryChangeManager instance redoOperation.
and undo, and .... :)


### Combining operations - Add class with instance variables

As we saw, the RBAddClassRefactoring does not allow us to define any instance variables. Instead we can add a new class and then apply another refactoring, RBAddInstanceVariableRefactoring.
We just need to call them in the appropriate order and make sure that both operations operate on the same model - otherwise the instance variable refactoring would not know about the class it
should operate on.

  | model addClassRB addInstVarsRB browser |
    model := RBNamespace new.
    addClassRB := RBAddClassRefactoring
        model: model
        addClass: #SomeClass
        superclass: #Object
        subclasses: {}
        category: #Category.
    addClassRB primitiveExecute.
    addInstVarsRB := RBAddInstanceVariableRefactoring
        model: model
        variable: 'x'
        class: #SomeClass.
    addInstVarsRB primitiveExecute.
    browser := ChangesBrowser new.
    browser := ChangesBrowser changes: (model changes changes ).
    browser open

It is important to actually execute the first operation before creating the second one. The instantiation of the RBAddInstanceVariableRefactoring will query the environment the class #SomeClass and
init the reference to nil if it doesn't yet exists.

The changes browser now includes two refactorings, you can select only the second one but this won't work.
If you applied both, and want to undo that changes, you'll need to call two times:
RBRefactoryChangeManager instance undoOperation.

### Scoping Refactoring

A refactoring operation often changes existing code by first, search for a pattern, and then transform the matching code into the new form. But in some situations you don't want to apply the change to all found matches.
For example, you want to rename a method in all implementors and all callers of your package. If this method name is a common message in other (system) classes as well, you don't want to rename all places and for sure you don't want to go through the ChangesBrowser and unselect all those matches by hand.

We can restrict the search space by creating our namespace from a restricted browser environment.
(More about restricted environments in the chapter RBBrowserEnvironments)

In this example we will apply the RBPrettyPrintCodeRefactoring to all classes in the Package 'Tests', by first creating a RBBrowserEnvironment for packages and then create the RBNamespace with this environment:

 | env model prettyPrintRB browser |
    env := RBBrowserEnvironment new forPackageNames:{'Tests'}.
    model := RBNamespace onEnvironment: env.
    prettyPrintRB := RBPrettyPrintCodeRefactoring new model: model; yourself.
    prettyPrintRB primitiveExecute.
    browser := ChangesBrowser new.
    browser := ChangesBrowser changes: (model changes changes ).
    browser open

After applying this refactoring, all methods in all classes of the package 'Tests' will be reformatted (pretty print).

### Refactoring Options

Some refactoring operations may require additional informations for performing the transformation. For example a 'move method ' refactoring, moving a method from one class to another may add an additional argument if the prior method had some 'self sends'. Some of the information are given by instantiating the refactoring and some information can be computed by the
operation itself. For other cases the refactoring may actually break code or create broken code. To make this operation still work the programmer or user of the refactoring engine
could provide the needed information. 

For this, the engine contains a set of 'options' that can be set by the tool using the framework, to register callback functions used to aquire the information from the user.

The options that can be used are:

 #implementorToInline - select one of a list of method names

 #methodName - ask for a method name

 #selfArgumentName - argument name to use for replacing self sends

 #selectVariableToMoveTo - select one of a list of variable names

 #variableTypes - select or provide a class

 #extractAssignment - should the code extraction include the variable assignment

 #inlineExpression - I don't know

 #alreadyDefined - Should it override  methods defined in the hierarchy.

 #useExistingMethod - Should it use existing (equivalent) method

 #openBrowser - call to open system browser

A tool now can register a callback like

refactoring setOption:#name_of_an_option toUse:[:a :b: ... a block with needed arguments]

for example, Calypso sets the option
 #implementorToInline to a method showing a dialog with a list to choose one of the provided selector names.

In the following example we show how to set the needed options manually. The RBMoveMethodRefactoring will ask us three questions
- selfArgumentName
- variableTypes
- methodName

for all of this options we set a simple block that just returns the information needed for this example task. In a real world tool, we
would need some interactive tool to let the user make a choice.
This RBMoveMethodRefactoring will move the implementation from TestResult class>>#historyFor: to its argument of type TestCase

    | model rbMoveMethod browser |
    model := RBNamespace onEnvironment: RBBrowserEnvironment new.
    rbMoveMethod := RBMoveMethodRefactoring
        model: model
        selector: #historyFor:
        class: TestResult class
        variable: 'aTestCaseClass'.
    rbMoveMethod setOption: #selfArgumentName toUse: [ :ref | 'aResultClass' ].
    rbMoveMethod
        setOption: #variableTypes
        toUse: [ :ref :types :selected | {(model classNamed: #TestCase)} ].
    rbMoveMethod
        setOption: #methodName
        toUse: [ :ref :name | RBMethodName selector: 'asHistoryFor:' arguments: {'aTestResult'} ].
    rbMoveMethod primitiveExecute.
    browser := ChangesBrowser changes: model changes changes.
    browser open

The result of this operation is, the method #historyFor: is moved to the class TestCase and the former implementation is replaced by
  aTestCase asHistoryFor: self
as the former implementation had a call to self (self newTestDictionary) we need to add self as an argument for the new method.
The refactoring operation queries this argument name by calling the registered block for the option 'selfArgumentName', as the refactoring can not guess the type
of the class we want to move the method, it will ask us by calling 'variableTypes' and finally the new method name and arguments are provided by calling the block for option
'methodName'.

## RB Refactoring Engine
A chapter with a more in-depth description of the core components of the refactoring engine.
### Overview

This book contains some chapter about the core components
the Abstract Syntax Tree (AST)
the parser (RBParser)
the extended pattern parser (RBPatternParser)
the tree searcher / rewriter (RBParseTreeSearcher/RBParseTreeRewriter)

### AST Nodes

The AST representing the code by a tree of nodes. A node may represent 
a single element
- RBVariableNode 
- RBLiteralValueNode 
an expression
- RBAssignmentNode
- RBMessageNode
- RBReturnNode
- RBCascadeNode
a sequence of expressions
- RBSequenceNode
or a block or Method
- RBBlockNode
- RBMethodNode

This nodes are part of a class hierarchy starting with RBProgramNode an abstract class defining the common operations needed for all nodes. Every node knows about its child nodes, the source code location, any comment attached (comment prior to this node in the source code, or for RBMethodNodes the "method comment" line), and the type (by its subclass) - see the is-Methods in "testing"-protocol.

Keep in mind that the syntax tree is created from the source code only and may not distinguish all possible type information without actually analyzing the semantic context. For example, a global variable is represented as RBGlobalNode, but just from parsing an expression, the AST only knows that this is a RBVariableNode. You need to call doSemanticAnalysis on the parse tree to convert variable nodes into the  type they represent in the code.


### AST Vistor

With this hierarchy of classes, the operations and programs working with the AST are often implemented with the visitor pattern.

AST node visitors are subclasses of a ProgramNodeVisitor, or a just any other class implementing the appropriate visitNode: / visitXXX: methods.

Some examples of ProgramNodeVisitors operating on the RBParsers AST:

Opal Compiler
Opals translator visits the AST tree to create a intermediate representation that is finally used to generated method byte code. Another step in the compiler work flow, the ClosureAnalyzer, is implemented as
a ProgramNodeVisitor too.

Reflectivity Compiler
For reflectivity support, can add MetaLinks to the nodes of the compiled method and generate new methods with code injections augmenting or modifying the executed code.

Code formatter (BIConfigurableFormatter/BISimpleFormatter)
A code formatter walks over the AST tree and reformats the code (node positions) based on a simple format rule or a configurable formatting style.

TextStyler
SHRBTextStyler builds a attributed text representation of the source code, augmented with text font, color or emphasis attributes based on the current style settings. 

And of course
RBParseTreeSearcher and RBParseTreeRewriter
The original users of this AST structure for searching and rewriting code, more on this in its own chapter.

### RBParser

The Refactoring Framework contains its own parser.

Defining or implementing refactoring operations on the raw source code level is difficult. For example, we would have to distinguish whether a word is an instance variable name, an argument or a reserved word.
Therefor a parser first translates the source code into an abstract syntax tree (AST).

The tree consists of nodes for every source code element, tagged it with some "type" information (the node subclass), source code location, and optional properties. And it represents the whole source code structure. 

For example, the AST for the source code of a method has a RBMethodNode with child nodes RBArgument for the arguments (if any) and a RBSequenceNode for the code body. The RBSequenceNode has child nodes for any
defined temporaries and the actual code, RBAssignmentNode for variable assignments, RBMessageNode for message sends.

This is how the structure  for Numbers #sgn method AST looks:
RBParser parseMethod:'sign
	self > 0 ifTrue: [^1].
	self < 0 ifTrue: [^-1].
	^0'

|->RBMethodNode sign
  |->RBSequenceNode self > 0 ifTrue: [ ^ 1 ]. self < 0 ifTrue: [ ^ -1 ]. ^ 0
    |->RBMessageNode ifTrue:
      |->RBMessageNode >
        |->RBSelfNode self
        |->RBLiteralValueNode 0
      |->RBBlockNode [ ^ 1 ]
        |->RBSequenceNode ^ 1
          |->RBReturnNode ^ 1
            |->RBLiteralValueNode 1
    |->RBMessageNode ifTrue:
      |->RBMessageNode <
        |->RBSelfNode self
        |->RBLiteralValueNode 0
      |->RBBlockNode [ ^ -1 ]
        |->RBSequenceNode ^ -1
          |->RBReturnNode ^ -1
            |->RBLiteralValueNode -1
    |->RBReturnNode ^ 0
      |->RBLiteralValueNode 0

Although many Smalltalk implementations already include a parser as a part of its compiler tool chain, they don't fulfill the requirements needed for the code transformations with the refactoring framework.
The AST for the compiler, is often only needed to create the byte code and therefore can ignore any code comments or the code formatting. If we use the AST in the refactoring for search and replace code, for example renaming a variable, we don't want to reformat the whole code or remove any code comments. 

The RBParser therefore stores the original code locations and code comments, and only replaces those elements defined by the refactoring transformation and preserves the method comments.

In recent pharo versions, the RBParser actual replaces the original parser used to compile code. It is as powerful as the prior parser, maybe a little bit slower, but easier to maintain. And in the mean time other tools, despite the compiler and the refactoring framework are using this tools as well. 
(For instance, the syntax highlighting and the code formatter are based on the RBParsers AST nodes).

But the real strength of the refactoring framework comes from another (RBParser sub-) class, the 
RBPatternParser, described in its own chapter.

### RBPatternParser and metavariables

Generating an AST of Smalltalk source code and implementing a program node visitor gives already great and powerful capabilities. The refactoring framework extends this expressiveness by including so called "metavariables".

As this expressions are using an extended syntax - metavariables aren't known to the RBParser - a special parser is needed to parse this expression, the RBPatternParser.
The following pages describe the added syntax elements. Examples on how to use or tests these expressions
can be found in the chapter "RBPatternParser examples".

metavariables are a part of a parser expression, just like any other Smalltalk code, but instead of representing an expression with the exact name, they form a variable that can be unify with any real code expression with the same *structure*.

An example:
Parsing an expression like:
a := a + 1 
creates a parse tree with an assignment node assigning to 'a', the value of sending the message '+' with argument 1 to the object 'a'.

We could implement a refactoring operation (or directly use the RBParseTreeSearcher/Rewriter) to create a refactoring  for this kind of code. But of course, it would only work for code using this variable name.

We can define the expression with the meaning of 'increment a variable by one' by using a metavariable. All metavariables start with a ´ (backquote).
`a := `a + 1

This is the simplest metavariable, a name with a backquote. It will match a single variable. And for matching the whole expression, all variables with the same name must match the same variables. 
The above expression only matches 
'x:=x+1' 
but not 
'x:=y+1'.

If we want to match more than a single variable, we can prefix the name with a '@':

`a matches a single variable
`@a matches multiple items in this position

For example, 
`@a add: `@b
will match any expression with the message send #add: regardless whether the receiver or arguments are single variables
'coll add: item'
or the return of another expression
'self data add: self'

Furthermore we can restrict the expression to be matched to be a literal instead of variable by using the prefix '#':

`@exp add: `#item

This will match any code calling #add: on an object or expression with a literal as argument:
'coll add: 3'
'self foo add: 'text' '
'coll add: #symbol'

But again, #lit is a named variable and matches only the same literal in every part of the expression:

`self add: `#lit; add: `#lit

will match
'self add: #a; add: #a'
but not 
'self add: #a; add: #b'

Similar to a statement ending with a dot, the metavariable prefix '.' defines a variable matching a statement, resp. '.@' a (possible empty) list of statements.

Example, match ifTrue:ifFalse: with first statement in true and false block being the same

`@exp ifTrue:[`.stm. 
				  `.@trueCase]
      ifFalse:[`.stm. 
				  `.@falseCase]

This will match

someValue ifTrue:[ self doA.
	                self doFoo]
          ifFalse:[ self doA.
	                self doBaz]


Important especially for the rewriter, we may not only want to know the first node matching an expression but every other and for example any possible subexpression matching the metavariable. For this, we can
use a double backquote to indicate that the search should recurse into the found node expression to search for more matches.

This expression will find all senders of add:
`@exp add:`@value
but if we would use this expression to rewrite add: by addItem:
an expression like

var add: (self add: aValue).

would be replaced by

var addItem: (self add: aValue).

If we want to find the same call in the argument, we need to recurse into it by using a double backquote

`@exp add:``@value

### Examples and usage of RBPatternParser expressions

The chapter "RBPatternParser and metavariables" describes the added syntax elements for the RBPatternParser used in the refactoring engine (RBParseTreeSearcher/RBParseTreeRewriter).

In this chapter we show some example expressions and how to test and use them.

Calypso has a search function that is the simples way to use and see the result of searching expressions with pattern syntax. Open the the class menu / Refactoring / Code Rewrite / Search code or Rewrite code entry.

Search code
The search code menu will put a search pattern template in the code pane:

RBParseTreeSearcher new
	matches: '`@object' do: [ :node :answer | node ];
	matchesMethod: '`@method: `@args | `@temps | `@.statements' do: [ :node :answer | node ];
	yourself
	

This template defines two match rules, one for the code search 'matches:' and one for the named method search 'matchesMethod', the former looks for expression in any method while the latter one matches whole methods.

You can replace the example pattern '`@object' or '`@method: `@args | `@temps | `@.statements' by
the search pattern you want to use. And most of the time you only want to use one, the code expression search or the method search.

A first example, replace the code pane content by:
RBParseTreeSearcher new
	matchesMethod: 'drawOn: `@args | `@temps | `@.statements' do: [ :node :answer | node ];
	yourself

You can now accept this code, instead of saving this method it will just spawn a code searcher trying all defined methods to match against this pattern and opens a MessageBrowser for all found results.
The result is actually the same as if we had searched for all implementors of #drawOn:

Next example, replace the code pane content by:
RBParseTreeSearcher new
	matches: '`@object drawOn: `@args' do: [ :node :answer | node ];
	yourself

The result is similar to looking for senders of #drawOn: (not the same actually, as sendersOf also looks for methods containing the symbol #drawOn: )	
	
The #do: block can be used to further test or filter the found matches. The node is the current matched node and the answer is not needed here. It is important that for every entry you want to include in the result to return "the node" and for everything else return "nil"

Example, search for all methods with at least one argument where the method name starts with 'permform':

RBParseTreeSearcher new
		matchesMethod: '`@method: `@args | `@temps | `@.statements'
			do: [ :node :answer | 
			((node selector beginsWith: 'perform') and: [ node arguments isEmpty not ])
				ifTrue: [ node ]
				ifFalse: [ nil ] ];
		yourself

Another way to use extended pattern syntax is to directly instantiate a RBParseTreeSearcher and execute it on a parse tree.
First we define the pattern, instantiate a tree searcher and tell him what to do when matching this pattern (just return the matched node) and execute it on the AST of Numbers method #asPoint.

| searcher pattern parseTree |
pattern := '^ self'.
searcher := RBParseTreeSearcher new.
searcher matches: pattern do:[:node :answer |node].
searcher executeTree: (Number>>#asPoint) ast initialAnswer: nil.

it will return nil, since no node in that method returns 'self'. If we execute the searcher instead on the method
for class Point, it will return the found node, a RBReturnNode

searcher executeTree: (Point>>#asPoint) ast initialAnswer: nil.

If we don't just want to match an expression but collecting all matching nodes, we can collect all nodes within the #do: block:

| searcher pattern parseTree  selfMessages |
selfMessages := Set new.
pattern := 'self `@message: ``@args'.
searcher := RBParseTreeSearcher new.
searcher matches: pattern do:[:node :answer |  selfMessages add: node selector].
searcher executeTree: (Morph>>#fullDrawOn:) ast initialAnswer: nil.
selfMessages inspect.

This will collect all messages send to self in method Morph>>#fullDrawOn:



### RBBrowserEnvironment

The first and main use for browser environments are to restrict the namespace in which a refactoring operation is applied. For example, if you want to rename a method and and update all senders of this method, but only in a certain package, you can create a RBNamespace from a scoped 'view' of the classes from the whole system. Only the classes in this restricted environment are affected by the transformation.

In the mean time other tools are using this environment classes as well. Finder, MessageBrowser or the SystemBrowser can work with a scoped environment to show and operate only on classes and methods in this environment.

There are different subclasses of RBBrowserEnvironment for the different kind of 'scopes'. 

RBClassEnvironment - only show classes/methods from a set of classes.
RBPackageEnvironment - only show classes / packages / methods from a set of packages.
RBSelectorEnvironment - only show classes / methods from a set of selector names.
(see the list of RBEnvironment(subclasses) pages in this book).

Instead of directly using the different subclasses for a scoped view, the base class RBBrowserEnvironment can act as a factory for creating restricted environments. See the methods in its 'environments'-protocol, on how to create the different environments.

You start with a default environment containing all classes from the system and create a new scoped environment by calling the appropriate method.

For example, creating an environment for all classes in package 'Kernel':

RBBrowserEnvironment new forPackageNames:{'Kernel'}.

You can query the environment just like you for Smalltalk globals
|env|
env := RBBrowserEnvironment new forPackageNames:{'Kernel'}.
env allClasses "-> a list of all classes in package Kernel"

or open a browser
env browse "-> starts Calypso showing only this package"

and you can further restrict this package environment by calling one of the other factory methods:

env class "-> a RBPackageEnvironment"
(env implementorsOf:#collect:) class "->  RBSelectorEnvironment"

Another way to combine or further restrict environments is to use boolean operations and, not or or.

|implDrawOn callsDrawOn implAndCalls |
callsDrawOn := RBBrowserEnvironment new referencesTo: #drawOn:.
implDrawOn :=  RBBrowserEnvironment new implementorsOf: #drawOn:.
"create an 'anded'-environment"
implAndCalls := callsDrawOn & implDrawOn.
"collect all message and open a MessageBrowser"
MessageBrowser browse: implAndCalls methods.

This opens a MessageBrowser on all methods in the system that implement #drawOn: and calls drawOn:.

|implPrintOn notImplPrintOn |
implPrintOn := RBBrowserEnvironment new implementorsOf: #printOn:.
"create a 'not'-environment"
notImplPrintOn := implPrintOn not.
implPrintOn includesClass: Object. "-> true"
notImplPrintOn includesClass: Object. "-> false"

classes implementing #printOn: are not in the 'not'-environment.

A more generic way to create an environment by giving an explicit 'test'-block to select methods for this environment:

|implementedByMe|
implementedByMe := RBBrowserEnvironment new selectMethods:[:m | m author = Author fullName ].
implementedByMe browse.
 
This opens (may be slow) a browser with all classes with methods having my (current Author) name for its current methods version author stamp.

## Core classes
A book chapter describing  important core classes from the class comments.
### RBProgramNode
RBProgramNode is an abstract class that represents an abstract syntax tree node in a Smalltalk program.

Subclasses must implement the following messages:
	accessing
		start
		stop
	visitor
		acceptVisitor:
	testing
		isFaulty

The #start and #stop methods are used to find the source that corresponds to this node. "source copyFrom: self start to: self stop" should return the source for this node.

The #acceptVisitor: method is used by RBProgramNodeVisitors (the visitor pattern). This will also require updating all the RBProgramNodeVisitors so that they know of the new node.

The #isFaulty method is used to distinguish between valid nodes and nodes created from an invalid source Smalltalk code. For example, code parsed with RBParsers #parseFaultyExpression: or #parseFaultyMethod:.

Subclasses might also want to redefine match:inContext: and copyInContext: to do parse tree searching and replacing.

Subclasses that contain other nodes should override equalTo:withMapping: to compare nodes while ignoring renaming temporary variables, and children that returns a collection of our children nodes.

Instance Variables:
	properties	<Dictionary of: Symbol -> Object>	A set of properties set to this node, for example every node can have the Property #comment to attach the method comment or the comment of the code line this node represents. Other classes or tools may add more type of properties; for example, the reflectivity support adds properties for managing Metalinks. 
	parent	<RBProgramNode>	the node we're contained in

Class Variables:
	FormatterClass	<Behavior>	the formatter class that is used when we are formatted

#### RBSequenceNode
RBSequenceNode is an AST node that represents a sequence of statements. Both RBBlockNodes and RBMethodNodes contain these.

Instance Variables:
	leftBar	<Integer | nil>	the position of the left | in the temporaries definition
	rightBar	<Integer | nil>	the position of the right | in the temporaries definition
	statements	<SequenceableCollection of: RBReturnNode or RBValueNode> the statement nodes
	periods	<SequenceableCollection of: Integer>	the positions of all the periods that separate the statements
	temporaries	<SequenceableCollection of: RBVariableNode>	the temporaries defined


#### RBReturnNode
RBReturnNode is an AST node that represents a return expression.

Instance Variables:
	return	<Integer>	the position of the ^ character
	value	<RBValueNode>	the value that is being returned


#### RBComment
An RBComment represents a text comment associated with an AST node.

An RBComment is not an AST-Node (not a subclass of program node). But its instances are just wrapping the comment text and (start-) position.

Due to the way the parser handles comments, the RBComment is assigned to its preceding (real) AST node, although we often write the comment prior to a statement.

For example:

foo
"method comment"

self firstStatement.

"comment about the return"
^ self

The "method comment" is assigned to the method node, the "comment about the return" is assigned
to the "self firstStatement" node!

instance variables
	contents 	<String> the comment text
	start	<Number> (start-) position within the method source

#### RBPragmaNode
RBPragmaNode is an AST node that represents a method pragma.

We have a fixed set of allowed "primitive" pragma keywords. Every method implemented as a primitive call uses one of this pragmas.
And as we need some special treatment for methods implemented as primitive, the RBPragmaNode adds the #isPrimitive testing method.

Instance Variables:
	arguments <SequenceableCollection of: RBLiteralNode> our argument nodes
	left <Integer | nil> position of <
	right <Integer | nil> position of >
	selector	<Symbol>	the selector we're sending
	keywordsPositions	<IntegerArray | nil>	the positions of the selector keywords
##### RBPatternPragmaNode
RBPatternPragmaNode is an RBPragmaNode that is used by the tree searcher to
match pragma statements. Just like RBPatternMethodNode for method nodes.

Instance Variables:
	isList	<Boolean>	are we matching each keyword or matching all keywords together (e.g., `keyword1: would match a one-argument method whereas `@keywords: would match 0 or more arguments)

#### RBValueNode
RBValueNode is an abstract class that represents a node that returns some value.

Subclasses must implement the following messages:
	accessing
		startWithoutParentheses
		stopWithoutParentheses
	testing
		needsParenthesis

Instance Variables:
	parentheses	<SequenceableCollection of: Inteval>	the positions of the parentheses around this node. We need a collection of intervals for stupid code such as "((3 + 4))" that has multiple parentheses around the same expression.


##### RBAssignmentNode
RBAssignmentNode is an AST node for assignment statements.

Instance Variables:
	assignment	<Integer>	position of the :=
	value	<RBValueNode>	the value that we're assigning
	variable	<RBVariableNode>	the variable being assigned


##### RBSelectorNode
RBSelectorNode is an AST node that represents a selector (unary, binary, keyword).

Instance Variables:
	value	<String>	the selector's name I represent or the ensemble of keywords I'm made of
	start <Integer>	the position where I was found at the source code

##### RBMessageNode
RBMessageNode is an AST node that represents a message send.

Instance Variables:
	arguments	<SequenceableCollection of: RBValueNode>	 our argument nodes
	receiver	<RBValueNode>	the receiver's node
	selector	<Symbol>	the selector we're sending
	keywordsPositions	<IntegerArray | nil>	the positions of the selector keywords


###### RBPatternMessageNode
RBPatternMessageNode is an RBMessageNode that will match other message nodes without their selectors being equal. 

Instance Variables:
	isCascadeList	<Boolean>	are we matching a list of message nodes in a cascaded message
	isList	<Boolean>	are we matching each keyword or matching all keywords together (e.g., `keyword1: would match a one-argument method whereas `@keywords: would match 0 or more arguments)
###### RFMessageNode
A message node
##### RBCascadeNode
RBCascadeNode is an AST node for cascaded messages (e.g., "self print1 ; print2").

Instance Variables:
	messages	<SequenceableCollection of: RBMessageNode>	the messages 
	semicolons	<SequenceableCollection of: Integer>	positions of the ; between messages


##### RBBlockNode
RBBlockNode is an AST node that represents a block "[...]".

Like RBMethodNode, the scope attribute is only valid after doing a semantic analyzing step.

Instance Variables:
	arguments	<SequenceableCollection of: RBVariableNode>	the arguments for the block
	bar	<Integer | nil>	position of the | after the arguments
	body	<RBSequenceNode>	the code inside the block
	colons	<SequenceableCollection of: Integer>	positions of each : before each argument
	left	<Integer>	position of [
	right	<Integer>	position of ]
	scope	<OCBlockScope | OCOptimizedBlockScope | nil> the scope associated with this code of this block


###### RBPatternBlockNode
RBPatternBlockNode is the node in matching parse trees (it never occurs in normal Smalltalk code) that executes a block to determine if a match occurs. valueBlock takes two arguments, the first is the actual node that we are trying to match against, and the second node is the dictionary that contains all the metavariable bindings that the matcher has made thus far.

Instance Variables:
	valueBlock	<BlockClosure>	The block to execute when attempting to match this to a node.


####### RBPatternWrapperBlockNode
RBPatternWrapperBlockNode allows further matching using a block after a node has been matched by a pattern node.

Instance Variables:
	wrappedNode	<RBProgramNode>	The original pattern node to match
##### RBArrayNode
An RBArrayNode is an AST node for runtime arrays.

Instance Variables
	left:	 <Integer | nil> position of {
	periods: <SequenceableCollection of: Integer> the positions of all the periods that separate the statements
	right: <Integer | nil> position of }
	statements: <SequenceableCollection of: RBValueNode> the statement nodes
##### RBLiteralNode
RBLiteralNode is an AST node that represents literals.

Instance Variables
	start: <Integer | nil> source position for the literal's beginning
	stop: <Integer | nil> source position for the literal's end
###### RBLiteralValueNode
RBLiteralNode is an AST node that represents literal values (e.g., #foo, true, 1, etc.), but not literal arrays.

The sourceText field is needed for the formatter for the correct printing of strings vs symbols. If we just call
value asString, both a string and a symbol print itself as a string.

Instance Variables
	value	<Numeric | Symbol | String  | Character>	the literal value I represent
	sourceText <String> the original source text of this literal
###### RBLiteralArrayNode
An RBLiteralArrayNode is an AST node that represents literal arrays #(1 2 3) and literal byte arrays #[ 1 2 3 ].

Instance Variables
	contents: <Array of: RBLiteralNode> literal nodes of the array
	isByteArray: <Boolean> if the receiver is a literal byte array

##### RBParseErrorNode
I am a node representing a source code segment that could not be parsed. I am mainly used for source-code coloring where we should parse as far as possible and mark the rest as a failure.

Parsing faulty code without raising a syntax error is done by 
RBParser parseFaultyExpression:
or
RBParser parseFaultyMethod: 

The return value is either valid nodes representing the AST, or nodes representing the valid portion and an RBParseErrorNode for the remaining invalid code.


###### RBEnglobingErrorNode
I am a node representing a source code segment that parsed but never used in a node because of an unexpected error at the end. I am mainly used for source-code coloring and icon styling where all the code parsed should be colored normaly but underlined as part of the error.
This node also propose a reparation research.

Parsing faulty code without raising a syntax error is done by 
RBParser parseFaultyExpression:
or
RBParser parseFaultyMethod: 

Accessing to the parsed nodes contained inside the node is the method 'content'.


####### RBBlockErrorNode
This is a particular englobing node that is a Block.
Exemple : [ block node ]
Can be created by forgetting 
either the opening : block node ]
or the closure : [ block node .

####### RBLiteralByteArrayErrorNode
This is a particular englobing node that is a literal byte array.
Exemple : #[ literal byte array node ]
Can be created by forgetting 
the closure : #[ array node .
Forgetting the opening will be assumed to be a block node.
####### RBParenthesesErrorNode
This is a particular englobing node that is a parentheses.
Exemple : ( parentheses node )
Can be created by forgetting 
either the opening : parentheses node )
or the closure : ( parentheses node .

####### RBTemporariesErrorNode
This is a particular englobing node that is a temporaries.
Exemple : | temporaries node |
Can be created by forgetting 
the closure : | temporaries node .
Forgetting the opening will be assumed to be a binary selector.
####### RBInvalidCascadeErrorNode
Please comment me using the following template inspired by Class Responsibility Collaborator (CRC) design:

For the Class part:  State a one line summary. For example, "I represent a paragraph of text".

For the Responsibility part: Three sentences about my main responsibilities - what I do, what I know.

For the Collaborators Part: State my main collaborators and one line about how I interact with them. 

Public API and Key Messages

- message one   
- message two 
- (for bonus points) how to create instances.

   One simple example is simply gorgeous.
 
Internal Representation and Key Implementation Points.


    Implementation Points
####### RBUnreachableStatementErrorNode
Please comment me using the following template inspired by Class Responsibility Collaborator (CRC) design:

For the Class part:  State a one line summary. For example, "I represent a paragraph of text".

For the Responsibility part: Three sentences about my main responsibilities - what I do, what I know.

For the Collaborators Part: State my main collaborators and one line about how I interact with them. 

Public API and Key Messages

- message one   
- message two 
- (for bonus points) how to create instances.

   One simple example is simply gorgeous.
 
Internal Representation and Key Implementation Points.


    Implementation Points
####### RBLiteralArrayErrorNode
This is a particular englobing node that is a literal array.
Exemple : #( literal array node )
Can be created by forgetting 
the closure : #( array node .
Forgetting the opening will be assumed to be a parentheses node.
####### RBPragmaErrorNode
This is a particular englobing node that is a pragma.
Exemple : < pragma node >
Can be created by forgetting 
the closure : < pragma node .
Forgetting the opening will be assumed to be a binary selector.
####### RBUnfinishedStatementErrorNode
Please comment me using the following template inspired by Class Responsibility Collaborator (CRC) design:

For the Class part:  State a one line summary. For example, "I represent a paragraph of text".

For the Responsibility part: Three sentences about my main responsibilities - what I do, what I know.

For the Collaborators Part: State my main collaborators and one line about how I interact with them. 

Public API and Key Messages

- message one   
- message two 
- (for bonus points) how to create instances.

   One simple example is simply gorgeous.
 
Internal Representation and Key Implementation Points.


    Implementation Points
####### RBArrayErrorNode
This is a particular englobing node that is an array.
Exemple : { array node }
Can be created by forgetting 
either the opening : array node }
or the closure : { array node .

####### RBMissingOpenerErrorNode
Please comment me using the following template inspired by Class Responsibility Collaborator (CRC) design:

For the Class part:  State a one line summary. For example, "I represent a paragraph of text".

For the Responsibility part: Three sentences about my main responsibilities - what I do, what I know.

For the Collaborators Part: State my main collaborators and one line about how I interact with them. 

Public API and Key Messages

- message one   
- message two 
- (for bonus points) how to create instances.

   One simple example is simply gorgeous.
 
Internal Representation and Key Implementation Points.


    Implementation Points
##### RBVariableNode
RBVariableNode is an AST node that represents a variable (global, inst var, temp, etc.).

Although this is the basic class for the concrete variable types, this is not an abstract class and is actually used
by the parser for all variables that aren't special builtin types like self/super/thisContext. All other variables are
just RBVariableNodes until the semantic analyser can deduce the type.

Instance Variables:
	name	<RBValueToken>	the variable's name I represent
	nameStart <Integer>	the position where I was found at the source code

###### RBPatternVariableNode
RBPatternVariableNode is an AST node that is used to match several other types of nodes (literals, variables, value nodes, statement nodes, and sequences of statement nodes).

The different types of matches are determined by the name of the node. If the name contains a # character, then it will match a literal. If it contains, a . then it matches statements. If it contains no extra characters, then it matches only variables. These options are mutually exclusive.

The @ character can be combined with the name to match lists of items. If combined with the . character, then it will match a list of statement nodes (0 or more). If used without the . or # character, then it matches anything except for a list of statements. Combining the @ with the # is not supported.

Adding another ` in the name will cause the search/replace to look for more matches inside the node that this node matched. This option should not be used for top-level expressions since that would cause infinite recursion (e.g., searching only for "``@anything").

Instance Variables:
	isAnything	<Boolean>	can we match any type of node
	isList	<Boolean>	can we match a list of items (@)
	isLiteral	<Boolean>	only match a literal node (#)
	isStatement	<Boolean>	only match statements (.)
	recurseInto	<Boolean>	search for more matches in the node we match (`)


###### RFStoreIntoTempNode
I define a temp that I can store into
###### RFStorePopIntoTempNode
I define a temp that I can store into
#### RBMethodNode
RBMethodNode is the node that represents AST of a Smalltalk method.

Some properties aren't known to the parser creating this Object. For example, the scope value isn't known by parsing the code but only after doing a
semantic analysis. Likewise, the compilation context isn't needed until we try to do the semantic analysis. 

Instance Variables:
	arguments	<SequenceableCollection of: RBVariableNode>	the arguments to the method
	body	<BRSequenceNode>	the body/statements of the method
	nodeReplacements	<Dictionary>	a dictionary of oldNode -> newNode replacements
	replacements	<Collection of: RBStringReplacement>	the collection of string replacements for each node replacement in the parse tree
	selector	<Symbol>	the method name
	keywordsPositions	<IntegerArray | nil>	the positions of the selector keywords
	source	<String>	the source we compiled
	scope	<OCMethodScope | nil> the scope associated with this code of this method
	pragmas	< SequenceableCollection of: RBPragmaNodes > Nodes representing the pragma statements in this method
	compilationContext	<CCompilationContext | CompilationContext>

##### RBPatternMethodNode
RBPatternMethodNode is an RBMethodNode that will match other method nodes without their selectors being equal. 

Instance Variables:
	isList	<Boolean>	are we matching each keyword or matching all keywords together (e.g., `keyword1: would match a one-argument method whereas `@keywords: would match 0 or more arguments)


### RBRefactoring
I am the abstract base class for refactoring operations. 

I define the common workflow for a refactoring:
check precondition, 
primitive execute - a dry run collecting the changes without applying them,
and execute - run and apply changes.

I provide many utility methods used by my subclasses. 
Every  concrete subclass implements a single refactoring. They have to implement the methods
preconditions and transform.


Instance Variables

options:
Some refactorings may need user interactions or some extra data for performing
the operation, the code for requesting this data is stored in a block associated with a "refacotring option"
(see RBRefactoring>>#setOption:toUse:  and RBRefactoring class>>#initializeRefactoringOptions).

model:
My model - a RBNamespace - defines the environment in which my refactoring is applied and collects all changes (RBRefactoryChange).

A RBRefactoringManager  is used to collect the executed refactorings and provides an undo and redo facility.

#### RBAbstractVariablesRefactoring
I am a refactoring used by other refactoring operations for extracting direct inst var and pool var 
access to accessor methods.

For example RBMoveMethodRefactoring uses me.
#### RBClassRefactoring
I am an abstract base class for class refactorings.

All that I provide is the class name, my subclass refactorings are operating on, and a instance creation method
for setting the class name and an initial namespace model.

Check method `RBClassRefactoring class>>#model:className:` 


##### RBAccessorClassRefactoring
I am a refactoring operation for creating accessors for all variables.

Example:
Create accessors for all instance variables:

```
RBAccessorClassRefactoring 
	model: RBNamespace new className: 'Morph' .
```
Create accessors for all class instance variables:

```
RBAccessorClassRefactoring 
	model: RBNamespace new className: 'Morph class' .
```
If the class already contains that accessor, I will create another one with a numbered suffix.

##### RBAddClassRefactoring
I am a refactoring for creating new classes. 

You can define the name, superclass, category and subclasses.

I am used by other refactorings that may create new classes, for example, RBSplitClassRefactoring.

My preconditions verify that I use a valid class name, that does not yet exists as a global variable, 
and the subclasses (if any) were direct subclasses of the superclass.
##### RBChildrenToSiblingsRefactoring
I am a refactoring operation for moving a class and its subclasses to a new super class.

You can choose which of the original childclasses should become now siblings.

For example,  we can generate a new Superclass for ClassS in
Object >> ClassP >> ClassS
Object >> ClassP >> ClassS >> ClassC1
Object >> ClassP >> ClassS >> ClassC2
Object >> ClassP >> ClassS >> ClassC3

and choose to move ClassC2 and ClassC3 to the new superclass - ClassNewP.

Object >> ClassP >> ClassNewP >> ClassS
Object >> ClassP >> ClassNewP >> ClassS >> ClassC1
Object >> ClassP >> ClassNewP >> ClassC2
Object >> ClassP >> ClassNewP >> ClassC3

Any method and instance variables,  defined in ClassS and used by the new siblings of ClassS are pushed up to the new superclass.


##### RBCopyClassRefactoring
I am a refactoring for copy a class.

My preconditions verify, that the copied class exists (in  the current namespace) and that the new copy class name is valid and not yet used as a global variable name 

The refactoring transformation create a new class and copy all instance and class methods of copied class.

Example
---------------
```
	(RBCopyClassRefactoring 
		copyClass: #RBFooLintRuleTestData 
		withName: #RBFooLintRuleTestData1 in: #Example1) execute. 
```
##### RBDeprecateClassRefactoring
I am a refactoring operation for removing of usages of a deprecated class, that was renamed to another name.

 I'm doing following operations:
 - all subclasses of the deprecated class will use the new class as superclass (optional)
 - convert new class to superclass of deprecatedclass, remove methods of deprecated class and add class method #isDeprecated (optional)
 - rename all references in the code
 - move extensions of the deprecated class owned by other packages to the new class (optional)
 - remove the extensions (optional)

##### RBGenerateEqualHashRefactoring
I am a refactoring for generating `#hash` and `#=` comparing methods.

For example, a Class with three instance methods inst1-inst3

```
RBGenerateEqualHashRefactoring 
	model: RBNamespace new 
	className: ClassS 
	variables: { #inst1 . #inst2 . #inst3 }.
```
will create:
a `#hash` method 

```
hash
	"Answer an integer value that is related to the identity of the receiver."

	^ inst1 hash bitXor: (inst2 hash bitXor: inst3 hash)
```
	
and a `#=` method
```

= anObject
	"Answer whether the receiver and anObject represent the same object."

	self == anObject
		ifTrue: [ ^ true ].
	self class = anObject class
		ifFalse: [ ^ false ].
	^ inst1 = anObject inst1
		and: [ inst2 = anObject inst2 and: [ inst3 = anObject inst3 ] ]
```

and any instvar accessor for the  instance variables used by method `#=`.

##### RBGeneratePrintStringRefactoring
I am a refactoring for generating a printString (printOn: aStream) method. 

You can specify which of my instance variables should be used for generating the printString.

For example: 

```
RBGeneratePrintStringRefactoring 
	model: RBNamespace new 
	className: ClassS 
	variables: { #inst1 . #inst2 . #inst3 }
```

##### RBRealizeClassRefactoring
Complete the set of defined methods of this class, by generating a "self shouldBeImplemented" method for all abstract methods defined in its superclass hierarchy. Where an abstract method is a method sending "self subclassResponsibilty.
Shows a warning if this class has abstract methods on its own.
##### RBRenameClassRefactoring
I am a refactoring for renaming a class.

My preconditions verify, that the old class exists (in  the current namespace) and that the new class name is valid and not yet used as a global variable name 

The refactoring transformation will replace the current class and its definition with the new class name. And update all references in all methods in this namespace, to use the new name. Even the definition for subclasses of the old class will be changed.

Example
---------------

	(RBRenameClassRefactoring rename: 'RBRenameClassRefactoring' to: 'RBRenameClassRefactoring2') execute
#### RBExpandReferencedPoolsRefactoring
I am a refactoring operations for finding direct pool variables  references.

I am used by other refactorings, for example to push down/ pull up a method.
Moving a method from class A to class B, that referes to some pool variables of class A, 
this refactoring will add the pool definition to class B.


#### RBMethodRefactoring
I am an abstract base class for method refactorings.

I only provide a helper method for generating  selector names.
##### RBAddMethodRefactoring
I am a refactoring for adding methods to a class.

My operation will compile a new method to a class in the specified protocol.

You can create an instance with: 

```
RBAddMethodRefactoring 
	model: RBNamespace new 
	addMethod:'foo ^ self' 
	toClass:Morph inProtocols:{'test'}.
```

The method to compile is the full method source (selector, arguments and code).

My precondition verifies that the methods source can be parsed and that the class does not already understands this methods selectors. That means, you can not use this refactoring to add methods for overwriting superclass methods.

##### RBChangeMethodNameRefactoring
I am an abstract base class for refactorings changing a method name.

Doing a method rename involves:
- rename implementors
- rename message sends and
- remove renamed implementors.

I implement the above precedures and provide helper functions for finding and renaming references.
Every concrete subclass has to add its own precondition (see `myPrecondition`).

###### RBAddParameterRefactoring
I am a refactoring operations for adding method arguments.

You can modify the method name and add an additional keyword argument and the default value used by senders of the original method. Only one new argument can be added. But you can change the whole method name, as long as the number of argument matches.

For example, for `r:g:b:`  add another parameter "a" the new method is `r:g:b:a:`
or change the whole method to `setRed:green:blue:alpha:`

This refactoring will 
- add a new method with the new argument, 
- remove the old method (for all implementors) and 
- replace every sender of the prior method with the new one, using the specified default argument.
###### RBRemoveParameterRefactoring
I am a refactoring for removing (unused) arguments.

My preconditions verify that the argument to be removed is not referenced by the methods and that the new method name isn't alread used.
Any sender of the prior selector will be changed to the new.

If the method contains more than one argument, I request the user to choose one of the arguments.
####### RBInlineParameterRefactoring
I am a refactoring for removing and inlining method arguments.

If all callers of a method with arguments, call that method with the same literal argument expression, you can remove that argument and inline the literal into that method.

My precondition verifies that the method name without that argument isn't already used and that all callers supplied the same literal expression.

For example, a method foo: anArg

```
foo: anArg
	anArg doSomething.
```

and all senders supply the same argument: 	     

```
method1
	anObject foo: 'text'.

method2
	anObject foo: 'text'.
```	
the method argument can be inlined:

```
foo
 | anArg |
 anArg := 'text'.
	anArg doSomething.
```

and the callers just call the method without any arguments:

```
method1
	anObject foo.
```
###### RBRenameMethodRefactoring
I am a refactoring operation for renaming methods.

The new method name has to have the same number of arguments, but the order of arguments can be changed.

My preconditions verify that the number of arguments is the same and that the new method name isn't already used.

All references in senders of the old method are changed, either the method name only or the order of the supplied arguments.

Example
--------
There are two ways to rename a method, one of them is rename all senders of method:
```
(RBRenameMethodRefactoring 
		renameMethod: ('check', 'Class:') asSymbol
		in: RBBasicLintRuleTestData
		to: #checkClass1:
		permutation: (1 to: 1)) execute.
```
And the other is rename the method only in specific packages:
```
|refactoring|
refactoring :=RBRenameMethodRefactoring 
		renameMethod: ('check', 'Class:') asSymbol
		in: RBBasicLintRuleTestData
		to: #checkClass1:
		permutation: (1 to: 1).
refactoring searchInPackages:  #(#'Refactoring-Tests-Core').
refactoring execute
```
###### RBReplaceMethodRefactoring
I'm a refactoring operation for replace one method call by another one.

The new method's name can have a different number of arguments than the original method, if it has more arguments a list of initializers will be needed for them.

All senders of this method are changed by the other.

Example
-------
Script:
```
(RBReplaceMethodRefactoring  
	model: model
	replaceMethod: #anInstVar:
	in: RBBasicLintRuleTestData
	to: #newResultClass: 
	permutation: (1 to: 1)
	inAllClasses: true) execute
```
##### RBCreateCascadeRefactoring
I am  a refactoring used to generate cascades in source code.

Two or more message sends to the same object are replaced by a cascaded message send. It expects a selection of the messages and the receiver variable.

##### RBDeprecateMethodRefactoring
I am a refactoring for deprecate a method.

My preconditions verify, that the old selector exists (in  the current namespace) and that the new selector is a valid selector

The refactoring transformation will add the call to the #deprecated:on:in: method 

Example
---------------

Script:
```
	(RBDeprecateMethodRefactoring 
		deprecateMethod: #called:on: 
		in: RBRefactoryTestDataApp 
		using: #callFoo) execute
```

Before refactoring:
```
RBRefactoryTestDataApp >> called: anObject on: aBlock 
	Transcript
		show: anObject printString;
		cr.
	aBlock value
```

After refactoring:
```
RBRefactoryTestDataApp >> called: anObject on: aBlock 
	self
		deprecated: 'Use #callFoo instead'
		on: '16 April 2021'
		in: 'Pharo-9.0.0+build.1327.sha.a1d951343f221372d949a21fc1e86d5fc2d2be81 (64 Bit)'.
	Transcript
		show: anObject printString;
		cr.
	aBlock value
```
##### RBExtractMethodAndOccurrences
I am a refactoring for creating a method from a code fragment and then find all occurrences of code fragment in its class and its hierarchy if apply.

You can select an interval of some code in a method and call this refactoring to create a new method implementing that code and replace the code by calling this method instead. Then find occurrences of extracted method in all class methods and replace the duplicated code by calling extracted method istead.
###### RBExtractSetUpMethodAndOccurrences
I'm a refactoring to extract a setUp method and then searchs occurrences of it and replaces them.

Example script
--------------
```
(RBExtractSetUpMethodAndOccurrences
	extract: (17 to: 56)
	from: #testExample1
	in: RBTest) execute.
```
Before refactoring:
```
RBDataTest >> testExample1 	
	self someClasses.
	aString := 'Example'.
	self assert: 4 > 5 equals: false.
RBDataTest >> testExample2
	"Example"
	self someClasses.
	aString := 'Example'.
	self assert: true.
	
RBDataTest >> testExample3
	"Example"
	self someClasses.
	"Comment"
	aString := 'Example'.
	self deny: false.
	
RBDataTest >> testExample4
	self assert: true.
	self deny: false.
```
After refactoring:
```
RBDataTest >> setUp "Added setUp"
	super setUp.
	self someClasses.
	aString := 'Example'.

RBDataTest >> testExample1 "removes setUp occurrence"
	self assert: 4 > 5 equals: false
	
RBDataTest >> testExample2
	self assert: true.

RBDataTest >> testExample3
	self deny: false.

RBDataTest >> testExample4 "this method did not change"
	self assert: true.
	self deny: false
```
##### RBExtractMethodRefactoring
I am a refactoring for creating a method from a code fragment.

You can select an interval of some code in a method and call this refactoring to create a new method implementing that code and replace the code by calling this method instead. 
The new method needs to have as many arguments as the number of (temp)variables, the code refers to.

The preconditions are quite complex. The code needs to be parseable valid code. 
###### RBExtractSetUpMethodRefactoring
I am a refactoring for creating a setUp method from a code fragment.

You can select an interval of some code in a test method and call this refactoring to create a setUp method implementing that code and replace the code by nothing. 
The selected class need to be a subclass of TestCase.

The preconditions are quite complex.
	- The code needs to be parseable valid code. 
	- The class must not implement setUp method.
	- Class must inherit from testCase class 
	
Example script
---------------
```
(RBExtractSetUpMethodRefactoring
	extract: (14 to: 29)
	from: #testExample
	in: RBDataTest) execute.
```

Before refactoring:
```
TestCase subclass: #RBDataTest
	instanceVariableNames: ''
	classVariableNames: ''
	package: 'Example'.

RBDataTest >> someMethod
	#someMethod.

RBDataTest >> testExample
	self someMethod.
	self assert: true.
```
After refactoring:
```
RBDataTest >> setUp
	super setUp.
	self someMethod.

RBDataTest >> testExample
	self assert: true.
``` 




##### RBExtractMethodToComponentRefactoring
I am a refactoring for extracting code fragments to a new method. 

Similar to `RBExtractMethodRefactoring`, but you can choose to which component (instance or agument variable) the new method is added. 
As such, the new method arguments will include an additional argument for the sender.
Based on the instance variable you chosed for this method I will guess the class where to add this method, but you can change this class or add more classes.

##### RBExtractToTemporaryRefactoring
Add a new temporary variable for the value of the selected code. Every place in this method using the same piece of code is replaced by accessing this new temporary variable instead.
As the code is now only evaluated once for initializing the variable value, this refactoring may modify the behavior if the code statements didn't evaluate to the same value on every call.

My preconditions verify that the new temporary name is a valid name and isn't already used (neither a temporary, an instance variable or a class variable).
##### RBFindAndReplaceRefactoring
I am a refactoring for find occurrences of a method in owner class and in the whole hierarchy if apply.

My precondition verifies that the method exists in specified class, and if occurrences are found in hierarchy this method should not overwritten in hierarchy.

Example script
----------------

```
(RBFindAndReplaceRefactoring 
find: #methodWithArg:andArg: 
of: MyClassA 
inWholeHierarchy: true) execute.
```
Before refactoring:
```
Object subclass: #MyClassA
	instanceVariableNames: ''
	classVariableNames: ''
	category: 'Testing'

MyClassA >> methodWithArg: anArg1 andArg: anArg2
	^ (anArg1 > anArg2) not	

MyClassA subclass: #MyClassB
	instanceVariableNames: ''
	classVariableNames: ''
	category: 'Testing'
	
MyClassB >> someMethod
	^  3
	
MyClassB >> dummyMethod
	(3 > self someMethod) not
```

After refactoring:

```
MyClassB >> dummyMethod 
	self methodWithArg: 3 andArg: self someMethod
```
###### RBFindAndReplaceSetUpRefactoring
I am a refactoring for find occurrences of setUp method in owner class and in the whole hierarchy if apply.

My precondition verifies that setUp method exists in specified class, and if occurrences are found in hierarchy this method should not overwritten in hierarchy.

Example script
----------------
```
(RBFindAndReplaceSetUpRefactoring 
	of: RBTest 
	inWholeHierarchy: false) execute.
```
Before refactoring:
```
TestCase subclass: #RBTest
	instanceVariableNames: 'aString'
	classVariableNames: ''
	package: 'Example'
	
RBTest >> setUp
	self someClasses.
	aString := 'Example'.

RBTest >> someClasses.
	"initialize some classes"
	
RBTest >> testExample1 	
	self someClasses.
	aString := 'Example'.
	self assert: 4 > 5 equals: false.
	
RBTest >> testExample2
	"Example"
	self someClasses.
	aString := 'Example'.
	self assert: true.

RBTest >> testExample4
	self assert: true.
	self deny: false
```

After refactoring: 
```
RBTest >> testExample1 
	self assert: 4 > 5 equals: false.

RBTest >> testExample2
	self assert: true

RBTest >> testExample4
	self assert: true.
	self deny: false
```
##### RBInlineAllSendersRefactoring
I am a refactoring for inlining code of this method.

The call to this method in all other methods of this class is replaced by its implementation. The method itself will be removed.

For example, a method 

```
foo
	^ 'text'.
```	
is called in

```
baz
	| a |
	a := self foo.
	^ self foo.
```	
inlining in all senders replaces the call to method foo, with its code:

```
baz
	| a |
	a := 'text'.
	^ 'text'.
```

##### RBInlineMethodRefactoring
I am a refactoring for replacing method calls by the method implementation.

You can select a message send in a method and refactoring this message send to inline its code.
Any temporary variable used in the original message send is added  into this method and renamed if there are already variables with this name.

My preconditions verify that the inlined method is not a primitive call, the method does not have multiple returns. I'll show a warning if the method is overriden in subclasses.


###### RBInlineMethodFromComponentRefactoring
I am a refactoring for replacing method calls by the method implementation.

Just like `RBInlineMethodRefactoring`,  I replace a message send by the implementation of that  message , but you can provide the component
where this implementation is taken from or choose one if there are move than one implementors.
If the method implementation has some direct variable references, accessor for this variable are created (just as by the generate accessor refactoring).
####### RBRemoveSenderRefactoring
I'm a refactoring for remove a method call.

Example
-------
```
| refactoring options |
refactoring := RBRemoveSenderRefactoring 
			remove: (	90 to: 105)
			inMethod: #caller1
			forClass: RBRefactoryTestDataApp.
options := refactoring options copy.
options at: #inlineExpression put: [:ref :string | false].
refactoring options: options.
refactoring execute.
```

Before refactoring:

```
RBRefactoryTestDataApp >> caller1
	| anObject |
	anObject := 5.
	self called: anObject + 1
		on1: 
			[:each | 
			each printString.
			^anObject]
```

After refactoring:
```
RBRefactoryTestDataApp >> caller1
	| anObject |
	anObject := 5.
	self called: anObject + 1
		on1: 
			[:each | 
			^anObject]
```

##### RBInlineTemporaryRefactoring
I am a refactoring to replace a temporary variable by code.

All references to the temporary variable in this method are replaced by the value used to initialize the temporary variable. 
The initialization and declaration of this variable will be removed. You need to select the variable and its initial assignment code to apply this refactoring.
##### RBMoveMethodRefactoring
I am a refactoring for moving a method from the class to one of its instance variable objects.

Moving a method moves it implementation to one or more classes and replaces the implementation in the original method by a delegation to one of the classes instance variable. 

I expect an option for selecting the type (classes) to which this method should be added.
A role typer RBRefactoryTyper is used to guess the possible classes used for this instance variables.
And an option for requesting the new method selector.

For all selected classes a method implementing the original method is created, and if the original code uses some references to self, a parameter needs to be added to provided the former implementor.

For example, moving the method #isBlack from class Color to its instvar #rgb for the type "Integer" creates a method 


```
Integer>>#isBlack
 ^ self = 0
```
and changes Colors implementation from: 
```
Color>>#isBlack
   ^ rgb = 0
```

to:

``` 
Color>>#isBlack
   ^ rgb isBlack
```
##### RBMoveMethodToClassRefactoring
A RBMoveMethodToClassRefactoring is a class that represents functionality of "Move method to class" refactoring.
User chooses method, and than any of existiong classes.
Refactoring moves chosen method to class.

Instance Variables
	method:		<RBMethod>

method
	- chosen method

###### RBMoveMethodToClassSideRefactoring
I'm a refactoring to move a method to class side.

My preconditions verify that the method exists and belongs to instance side.

I catch broken references (method senders and direct access to instVar) and fix them.

Example
-----------

Script
```
	(RBMoveMethodToClassSideRefactoring 
		method: (RBTransformationRuleTestData >> #rewriteUsing:) 
		class: RBTransformationRuleTestData) execute.
```
Before refactoring:
```
RBTransformationRuleTestData >> rewriteUsing: searchReplacer 
     rewriteRule := searchReplacer.
     self resetResult.
```
After refactoring:
```
RBTransformationRuleTestData >> rewriteUsing: searchReplacer
     ^ self class rewriteUsing: searchReplace.

RBTransformationRuleTestData class >> rewriteUsing: searchReplacer
    | aRBTransformationRuleTestData |
    aRBTransformationRuleTestData := self new.
    aRBTransformationRuleTestData rewriteRule: searchReplacer.
    aRBTransformationRuleTestData resetResult.
```
##### RBMoveVariableDefinitionRefactoring
I am a refactoring for moving the definition of a variable to the block/scope where it is used.

For a method temporary variable declared but not initialized in the method scope and only used within a block, the definition can be moved to the block using this variable.
##### RBPullUpMethodRefactoring
I am a refactoring for moving a method up to the superclass. 

My precondition verify that this method does not refere to instance variables not accessible in the superclass. And this method does not sends a super message that is defined in the superclass.
If the method already exists and the superclass is abstract or not referenced anywhere, replace that implementation and push down the old method to all other existing subclasses.



##### RBPushDownMethodRefactoring
I am a refactoring for moving a method down to all direct subclasses.

My preconditions verify that this method isn't refered  as a super send in the subclass. And the class defining this method is abstract or not referenced anywhere.


##### RBRemoveAllSendersRefactoring
I am a refactoring to remove all possible senders from a method (you cannot remove those calls where the result of the method call is used or when the method name symbol is referenced).

Example Script
----------------
```
| refactoring options |
refactoring := RBRemoveSenderRefactoring 
			remove: (90 to: 105) "node position to be removed "
			inMethod: #caller1
			forClass: RBRefactoryTestDataApp.
options := refactoring options copy.
options at: #inlineExpression put: [:ref :string | false].
refactoring options: options.
refactoring execute.
```
Before refactoring:
```
RBRefactoryTestDataApp >> caller1
	| anObject |
	anObject := 5.
	self called: anObject + 1
		on1: 
			[:each | 
			each printString.
			^anObject]
```
After refactoring (notice that the call to printstring was removed):
```
RBRefactoryTestDataApp >> caller1
	| anObject |
	anObject := 5.
	self called: anObject + 1
		on1: 
			[:each | 
			^anObject]
```
##### RBRemoveHierarchyMethodRefactoring
I am a refactoring for removing a method and those of its subclasses,
 to remove the methods use RBRemoveMethodRefactoring.

Example
-------
Script
```
(RBRemoveHierarchyMethodRefactoring 
		removeMethods: #(#msg4)
		from: RBSharedPoolForTestData) execute
```
##### RBRemoveMethodRefactoring
I am a refactoring for removing a method.

My preconditions verify that this method is not referenced anywhere.
##### RBRenameArgumentOrTemporaryRefactoring
I am a refactoring for renaming temporary variables.
This can be applied to method arguments as well.

The variable declaration and all references in this method are renamed.

My precondition verifies that the new name is a valid variable name and not an existing instance or a class variable name
##### RBSplitCascadeRefactoring
I am a refactoring splitting a cascade message send to multiple messages.

You can select an interval containing a cascade expression. The refactoring will split this expression to two message sends to the receiver. 

My preconditions verify that the selector containing the cascaded message send is defined in this class, and a cascade message can be found.

If the receiver of the cascade expression is a literal or the return value of another message send, I will add another temporary variable for the interim result.
##### RBSwapMethodRefactoring
Move a method from the class to the instance side, or vice versa. Normally this is not considered to be a refactoring.

Only instance methods with no instance variable access or class methods with no class instance variable access can be moved.
##### RBTemporaryToInstanceVariableRefactoring
I am a refactoring for changing a temporary variable to an instance variable.

My preconditions verify that this variable is not yet used as an instance variable in this class.

The temporary variable is added to the class definition and removed from the temporary declaration in this method .

If this instance variable is already used in a subclass it will be removed from that class, because subclasses already inherit this attribute.

The temporary variables with the same name in hierarchy will be removed, and replaced with the new instance variable.

Example
--------------------

Script refactoring:
```
(RBTemporaryToInstanceVariableRefactoring 
    class: MyClassA
    selector: #someMethod
    variable: 'log') execute
```
Before refactoring:
```
Object subclass: #MyClassA
	instanceVariableNames: ''
	classVariableNames: ''
	package: 'example'

MyClassA >> someMethod 
    |log aNumber|
    log := self newLog.
    log isNil.
    aNumber := 5.

MyClassA >> anotherMethod
    #(4 5 6 7) do: [:e | | log |
        log := e ]

MyClassA subclass: #MyClassB
	instanceVariableNames: 'log'
	classVariableNames: ''
	package: 'example'
```
After refactoring:
```
Object subclass: #MyClassA
	instanceVariableNames: 'log'
	classVariableNames: ''
	package: 'example'

MyClassA >> someMethod 
    | aNumber |
    log := self newLog.
    log isNil.
    aNumber := 5.

MyClassA >> anotherMethod
    #(4 5 6 7) do: [:e | 
        log := e ]

MyClassA subclass: #MyClassB
	instanceVariableNames: ''
	classVariableNames: ''
	package: 'example'
```
#### RBPackageRefactoring
I am an abstract base class for package refactorings.

All that I provide is the package name, my subclass refactorings are operating on, and a instance creation method for setting the package name and an initial namespace model.
##### RBCopyPackageRefactoring
I am a refactoring for copy a package.

My preconditions verify, that the copied package exists (in  the current environment) and that the new copy package name is valid and not yet used as a global variable name 

The refactoring transformation create a new package and copy defined classes of origin package (exclude all class' extensions)

Example
---------------
```
	(RBCopyPackageRefactoring 
		copyPackage: #'Refactoring-Help' 
		in: #'Refactoring-Help1') execute. 
```
##### RBRenamePackageRefactoring
I'm a refactoring to rename a package.

My preconditions verify that the new name is different from the current package name and is a valid name.

I change all the references of the classes that are defined within the package, and if there is a manifest, it is updated with the new name of the package. 

Example
---------
```
(RBRenamePackageRefactoring 
				rename: (self getPackageNamed: #'Refactoring-Tests-Core')
				to: #'Refactoring-Tests-Core1') execute.
```
#### RBPrettyPrintCodeRefactoring
I am a refactoring for reformat the source code of all methods in this environment.

I have no precondition.
#### RBRegexRefactoring
I am a abstract base class for a refactoring replacing strings by a regular expression.

My concrete subclasses define on what kind of string the replace regulare expression should be applied to.
They have to implement the RBRefactoring method #transform.

I have no special precondition.

Here is a sample of a script that will change of the YK* classes of package PrefixOfPackageNames* into ZG*.

```
| pkgPrefix newClassPrefix env model |
pkgPrefix := 'PrefixOfPackageNames'.
newClassPrefix := 'ZG'.
env := RBBrowserEnvironment new 
			forPackageNames: (RPackage organizer packageNames select: [ : pkgName | (pkgName beginsWith: pkgPrefix) ]).
model := (RBClassModelFactory rbNamespace onEnvironment: env) name: 'MyModel'; yourself.
RBClassRegexRefactoring new
	model: model;
	renameClasses; 					"Here I just want a rename no copy"
	replace: '^YK(.*)$' with: newClassPrefix,'$1';
  execute.
```

- ==renameClasses== renames the classes in place
- ==copyclasses== copies the classes (pay attention we are not sure that it is fully working)







##### RBCategoryRegexRefactoring
I am a regex refactoring renaming class categories (package names).

See comment of superclass for a nice script to be adapated to package names.
##### RBClassRegexRefactoring
I am a regex refactoring renaming or copying class names.
I offer several models of operation in addition to regex matching. 

Refactored classes can be renamed, copied and kept aside old ones: try renameClasses, copyClasses, or createClasses. 

See also the comment of superclass for a nice script.
##### RBProtocolRegexRefactoring
I am a regex refactoring renaming protocol names.

See comment of superclass for a nice script to be adapated to package names.
##### RBSourceRegexRefactoring
I am a regex refactoring replacing method sources.

See comment of superclass for a nice script to be adapated to method sources.
#### RBRemoveClassRefactoring
I am a refactoring for removing classes. 

My precondition verifies that the class name exists in this namespace and the class has no references, resp. users, if this is used to remove a trait.

If this class is "empty" (has no methods and no variables), any subclass is reparented to the superclass of this class. It is not allowed to remove non-empty classes when it has subclasses.
##### RBRemoveClassKeepingSubclassesRefactoring
I am a refactoring for removing classes but keeping subclasses in a safe way. 

My precondition verifies that the class name exists in this namespace and the class has no references, resp. users, if this is used to remove a trait.

If this class is "not empty" (has methods and variables), any subclass is reparented to the superclass of this class, and all its methods and variables (instance and class) are push down in its subclasses.

Example
--------
```
(RBRemoveClassKeepingSubclassesRefactoring classNames: { #RBTransformationRuleTestData1 }) execute. 
```

#### RBSplitClassRefactoring
I am a refactoring for extracting a set of instance variables to a new class.

You can choose which instance variables should be moved into the new class. The new class becomes an instvar of the original class and every reference to the moved variables is replaced by a accessor call.

My precondition verifies that the new instance variable is a valid variable name and not yet used in this class or its hierarchy
 the name of the new class representing the set of instance variables is a valid class name

Example:
In the following class:

```
Object subclass: #TextKlass
	instanceVariableNames: 'text color font style'
	classVariableNames: ''
	package: 'TestKlasses'
```	
the variables color/font/style should be moved to a new "TextAttributes"-Class.
We apply the Split Refactoring with this three variables and select a new class name TextAttributes used as variable new "textAttributes".
The class definition will be changed to

```
Object subclass: #TextKlass
	instanceVariableNames: 'text textAttributes'
	classVariableNames: ''
	package: 'TestKlasses'
```
	
and every reference to the old vars color / font / style will be replaced by textAttributes color / textAttributes style / textAttributesFont

#### RBVariableRefactoring
I am an abstract base class of refactorings modifying class or instance variables.
##### RBAbstractClassVariableRefactoring
I am a refactoring for replacing every direct access to  class variables with accessor methods.

My precondition verifies that the variable is directly defined in that class.
I create new accessor methods for the variables and replace every read and write to this variable with the new accessors.

##### RBAbstractInstanceVariableRefactoring
I am a refactoring for replacing every direct access to  instance  variables with accessor methods.

My precondition verifies that the variable is directly defined in that class.
I create new accessor methods for the variables and replace every read and write to this variable with the new accessors.

##### RBAddClassVariableRefactoring
I am a refactoring for adding new class variables.

My precondition verifies that the variable name is valid, not yet used in the whole hierarchy and not a global name.
##### RBAddInstanceVariableRefactoring
I am a refactoring for adding new instance variables.

My precondition verifies that the variable name is valid, not yet used in the whole hierarchy and not a global name.
##### RBCreateAccessorsForVariableRefactoring
I am a refactoring for creating accessors for variables.

I am used by a couple of other refactorings  creating new variables and accessors.

My precondition is that the variable name is defined for this class.
###### RBCreateAccessorsWithLazyInitializationForVariableRefactoring
I am a refactoring for creating accessors with lazy initialization for variables.

I am used by a couple of other refactorings creating new variables and accessors.

My precondition is that the variable name is defined for this class.

Example
--------
Script
```
(RBCreateAccessorsWithLazyInitializationForVariableRefactoring 
	variable: 'foo1' 
	class: RBLintRuleTestData 
	classVariable: false 
	defaultValue: '123') execute
```

After refactoring we get:
```
RBLintRuleTestData >> foo1 
	^ foo1 ifNil: [foo1 := 123]
	
RBLintRuleTestData >> foo1: anObject
	foo1 := anObject
```
##### RBMoveInstVarToClassRefactoring
RBMoveInstVarToClassRefactoring knows how to move instance variable from one class to another.

Instance Variables
	newClass:		<RBClass>

newClass
	- class, in which user moves an instance variable
##### RBProtectInstanceVariableRefactoring
I am a refactoring for protecting instance variable access.

If a class defines methods for reading and writing instance variables, they are removed and all calls on this methods.
Omit method that are redefined in subclasses.
##### RBPullUpClassVariableRefactoring
I am a refactoring for moving a class variable up to the superclass.
##### RBPullUpInstanceVariableRefactoring
I am a refactoring for moving a instance  variable up to the superclass.
##### RBPushDownClassVariableRefactoring
I am a refactoring for moving a class variable down to my subclasses.

My precondition verifies that the moved variable is not referenced in the methods of the original class.
##### RBPushDownInstanceVariableRefactoring
I am a refactoring for moving a instance variable down to my subclasses.

My precondition verifies that the moved variable is not referenced in the methods of the original class.
##### RBRemoveClassVariableRefactoring
I am a refactoring for removing class variables.

My precondition verifies that there is no reference to this class variable.
##### RBRemoveInstanceVariableRefactoring
I am a refactoring for removing instance variables.

My precondition verifies that there is no reference to this instance  variable.
##### RBRenameClassVariableRefactoring
I am a refactoring for rename class variables.

I rename the class variable in the class definition and in all methods refering to this variable.

My precondition verifies that the new variable is valid and not yet used in the whole class hierarchy.
##### RBRenameInstanceVariableRefactoring
I am a refactoring for rename instance variables.

I rename the instance variable in the class definition, in all methods refering to this variable and rename the old accessors.

My precondition verifies that the new variable is valid and not yet used in the whole class hierarchy.
###### RBMergeInstanceVariableIntoAnother
I am a refactoring for merge an instance variable into another.

I replace an instance variable by other, in all methods refering to this variable and rename the old accessors, then if the instance variable renamed is directly defined in class it is removed.

My precondition verifies that the new variable is a defined instance variable in class.

Example
----------------------------
Script
```
(RBMergeInstanceVariableIntoAnother rename: 'x' to: 'y' in: Foo) execute.
```

Before refactoring:
```
Class Foo -> inst vars: x, y 

Foo >> foobar
	^ x 

Foo >> foo
	^ x + y 
```
After refactoring merging X into Y
```
Class Foo -> inst vars: y 

Foo >> foobar
	^ y

Foo >> foo 
	^ y + y
```
#### EpRBPropagateRefactoring
I am a RBRefactoring intended for prepagating another refactoring. We call to propagate a refactoring to redo just the secondary effects of such refactoring. 

For example, the propagation of a 'message rename' is to change the senders of the old selector to use the new selector. 


## Environments

The infrastructure of the refactoring engine defines some environments and operations (and, or,...) over such environments. 
An environment is basically a slice over the system: it can contain for example all the classes of a set of packages. 
The key class is `RBBrowserEnvironment`. 
The following shows the class comments of the environments available in Pharo.

### RBBrowserEnvironment
I am the base class for environments of the refactoring framework.

I define the common interface for all environments.
And I act as a factory for various specialized environments. See my 'environment' protocol.

I am used by different tools to create a 'views' of subsets of the whole system environment to browse or act on (searching/validations/refactoring)

#### create instances:
```
RBBrowserEnvironment new forClasses:  Number withAllSubclasses.
RBBrowserEnvironment new forPackageNames: { #Kernel }.
```
#### query:

```
|env|
env := RBBrowserEnvironment new forPackageNames: { #Kernel }.
env referencesTo:#asArray.
-> RBSelectorEnvironment.
```

#### browse:

```
|env|
env := RBBrowserEnvironment new forPackageNames: { #Kernel }.
(Smalltalk tools browser browsedEnvironment: env) open.
```

### RBBrowserEnvironmentWrapper
I am a wrapper around special browser environment subclasses and
the base RBBrowserEnvironment class. I define common methods
for my subclasses to act as a full environment.
no public use.


### RBCategoryEnvironment
I am a RBBrowserEnvironment on a set of category names.
I containt all entities using this category name.
I am more restricted to the exact category name compared
to a package environment.

Example, all Morph subclasses in category Morphic-Base-Menus
```
(RBBrowserEnvironment new forClasses: Morph withAllSubclasses) forCategories: {#'Morphic-Base-Menus'}
```

### RBClassEnvironment
I am a RBBrowserEnvironment on a set of classes.
I containt all entities of this set.

Example:
```
(RBBrowserEnvironment new) forClasses: Number withAllSubclasses.
```
### RBClassHierarchyEnvironment
I am a RBBrowserEnvironment on a set of classes of a class hierarchy.

Example:

```
(RBBrowserEnvironment new) forClass:Morph protocols:{'printing'}.
```

### RBAndEnvironment

I am the combination of two RBEnvironments, a logical AND. That is: 
entity A is in this environment if it is in BOTH environment I am constructed from.

Do not construct instances of me directly, use method #& for two existing environments:
env1 & env2 -> a RBAndEnvironment.

### RBOrEnvironment
I am the combination of two RBEnvironments, a logical OR. That is: 
entity A is in this environment if it is in at least ONE environment I am constructed from.

Do not construct instances of me directly, use method #| for two existing environments:
env1 | env2 -> a RBOrEnvironment.

### RBNotEnvironment
I am the complement of RBEnvironments, a logical NOT. That is: 
entity A is in this environment if it is in NOT in the environment I am constructed from.

Do not construct instances of me directly, use method #not for an existing environment:
env1 not -> a RBNotEnvironment.

### RBPackageEnvironment
I am a RBBrowserEnvironment on a set of packages or package names.
I containt all entities are defined in this packages.
(classes and class that have extensions from this packages)

Example:
```
RBBrowserEnvironment new forPackageNames:{ 'Morphic-Base'}.
```

### RBPragmaEnvironment
I am a RBBrowserEnvironment on a set of Pragmas.
I containt all entities that define methods using this pragmas.
Example:
```
RBBrowserEnvironment new forPragmas:{ #primitive:}.
```
### RBProtocolEnvironment
I am a RBBrowserEnvironment on a set of protocols of a class.

Example:
```
RBBrowserEnvironment new forClass:Morph protocols:{'printing'}.
```
### RBSelectorEnvironment
I am a RBBrowserEnvironment for a set of selectors. 
Usually I am constructed as a result of a query on another environment:

```
env referencesTo:#aselector -> a RBSelectorEnvironments.
```

### RBVariableEnvironment
I am a RBBrowserEnvironment for items referring class or instvars.
Constructed by quering extisting environments with 
refering, reading or writing to the variables of a class.

Example:
```
RBBrowserEnvironment new instVarWritersTo:#color in: Morph.
-> a RBVariableEnvironment
```
