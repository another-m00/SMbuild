# SMbuild

***__This is a draft, and the language is in Work in progress state. When the language specification is finished it will get a Julia lexer and parser script.__***

A state machine builder language
---
SMbuild is (or at least will be) a simple language to make state machines.

It can be used to create various tools, such as "AI player scripts", event systems, or simple workflow automations, that use a state machine.


## The language
It's simply a text file that defines states and their effects.
Each state is a line with this syntax:
```
state => effect \n
```
Where the `\n` is a newline.

**Whitespace** is ignored completely, which means it can be in identifiers, values, and effects, but they are not replaced with other characters, nor appear in the data as if they didn't exist. (Except for strings of course.)

Basically every character is parsed as part of a name or `ID`, unless they are expected as tokens. So for example you can easily use a `!` in a name, everywhere except in the beginning of a state, because that would be parsed as a NOT token.

The **states are stored within a Dictionary or HashMap** depending on the implementing language, provided by the running environment, where the keys are `String`s and the values are `Number`s, or `Boolean`s. **The `ID`s point into the Dictionary and are parsed as Booleans by default.

## The `state`
A `state` can be an `ID`, a `condition` or a `group`.

### group
A `group` is simply one or more states grouped together with boolean operators. Since the operators use `state`s, `group`s can operate with other groups too.

*The group operators are the following:
| Operator syntax | Description |
|---|---|
| ! state | Boolean NOT operation on the state |
| state && state | Boolean AND between the states |
| state ^ state | Boolean XOR between the states |

**Note that there is no OR operator, since that can easily be done by adding a new line with the same effect.**

### condition
A `condition` is a check made by the executing script, in the global environment. It's syntax is `?( <check> )` where the check is a simple comparsion of values.

A `check` is simply 
```
<value> <operator> <value>
```

The `value`s can only be
* a *Name of a variable*,
* a *Number*,
* a *Boolean*,
* `null` or
* a *String literal*.

When you specify the **variable by name**, the running script will try to access the global object searching for that variable name, and then compare it with the other value. This comparsion will always be a boolean value.

You can use **dot** `.` to access indicies of the found variables, so for example `game.scenes.1` will look for `game["scenes"][1]`. The number will be tried as a number first, then as a string. **The decimal separator is a comma (`,`) in numbers.**

The operators are the following:
| Operator | Description |
|--|--|
| == | **Not strictly equals**, without checking the type to be equal too.
| === | **Strictly equals**, both the value and the types must match to be true. |
| != | **Not strictly not equals**, when the values match when they are converted, this is false. |
| !== | **Strictly not equals**, if either the type, or the value mismatches, becomes true. |
| > | **Bigger than**. *This cannot be applied to strings.* |
| < | **Smaller than**. *This cannot be applied to strings.* |
| >= | **Bigger than or equals to**. *This cannot be applied to strings.* |
| <= | **Smaller than or equals to**. *This cannot be applied to strings.* |
| & | **Bitwise and**. True when not 0. *This cannot be applied to strings.* |
| ^ | **Bitwise XOR**. True when not 0. *This cannot be applied to strings.* |

There are no groupings inside the conditionals, to keep the parser simple. You can use multiple single conditionals to form a group state.

You can also use `$` in front of the variable name to access a **state** that your state machine uses. These however cannot be indexed, and dots are considered to be part of the ID.

### IDs
IDs in states are simple boolean values that are read from the state machine's store.

By default these values are false, unless they are set by your state machine.

IDs can be anything as long as it don't conflict with any other notation, and the name and numbers are parsed as IDs by default and you have to denote when they are not IDs.

## Effects
`effect`s can set states, execute functions, and set variables.

When you give a simple IDs as an effect, the state with that ID becomes true and set , when it's state becomes true.

The state machine function needs to be executed repeatedly, which means the function calls you define will execute multiple times. You might want a second variable to keep track of wether or not you have already executed the function.
