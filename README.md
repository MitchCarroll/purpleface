# PurpleFace
The PurpleFace programming language

_Proposed Features:_
1. Function Inference
2. Boolean datatype and operations replaced with optional data type
  * `true` | `false` | `maybe`
  * every 'concrete' datatype is derived from `true`
  * the `maybe` keyword casts expressions to a `maybe`
3. Lambdas cannot be bound to a variable
  * will result in a compiler error of "Nice Try"
4. There is no 'let' for binding variables; only `maybe`
  * `maybe 2?` returns a value that is either 2 (and `true`) or not 2 (and therefore, `maybe`)
  * `maybe a is 1 + (2?)?` will bind `a` to a value that may or may not be the sum of 1 and a number that may or may not be 2, so the possible values are (1?), 3, maybe, and false
    * Note: `1 + maybe` is `maybe 1`, which has the shorthand `1?` 
5. Optional values are all quantum-spooky. They do not resolve to concrete values until a concrete value is required.
  * The whole program is generally lazy evaluated. 
  * At runtime, `maybe` bindings create multiple continuations.
  * Only continuations which result in valid result are retained.
6. All statements return a value.
7. The return value of a continuation is the last returned value inside of it.
8. Imperitive statements end with a `!`
9. 'Quantum' values and statements end with a `?`
10. Parameters to functions do not need to be surrounded in `()`, but still need to be separated by `,`

_Possible Example Code:_
```
maybe a = 2?
a < 3? 
if true, print a!
```

_Example Execution:_
1. `a` is allocated, and bound to a value which may be `2`, `maybe`, or `false`
  * A new continuation is created for each.
2. In each of the 3 continuations, evaluate `a < 3`, creating another new continuation for each possible result
  * All conditional expressions return an optional value, so a new continuation is created for all possible results:
    * `true`, or `maybe` in the `a = 2` continuation
    * `maybe` in the `a = maybe` continuation
    * `false`, or `maybe` in the `a = false` continuation
3. Only the "`a < 3` is `true`" continuation will reach the `print` statement.
  * The only continuation of the `print` statement prints `2`, since the only continuation which reaches the `print` statement is the one in which `a < 3` is true, which in turn was spawned in the continuation in which `a = 2`, and if `print` is given a valid (`true`) argument it will return `true`.
  * actually, `2` is not printed yet; it is -buffered- to be printed. All side effects are buffered until their execution is guaranteed.
    * side effects are only possible via buffered streams, or by opening and closing of those streams
4. The end of the program has been reached. The remaining continuations are the one that prints `2`, and two which printed nothing, but in which `a` is either `true` or `maybe`.
5. Since multiple continuation threads remain, they must be eliminated until only one remains.
  * The overall return value of each continuation is basically the last returned value.
  * `maybe` is not valid as a terminal return value, since it is not a definite resolution. Therefore that continuation is removed.
    * actually, this happened as soon as there were no remaining execution paths on that continuation, which happened after the `if` statement, not after the `print` statement
  * as a rule, `false` continuations are only selected if no `true` continuations remain. The `false` continuation was retained after when the `maybe` continuation was discarded, but now that all threads have completed, and a `true` continuation exists, the `false` continuation is removed.
6. only the `true` continuation remains, so its buffers are flushed, actuating its side effects
  * in other words, the `2` is actually printed now.

*Execution Steps:*

1. Trace execution paths

```
maybe a = 2?	     a
                   / | \
		  /  /  \ 
		 /  |    \
a < 3?		2   maybe -----false
               / \      \        |   \
if true,   true   maybe  maybe false  maybe
             |
print a!     [queue 2(value of a) on output buffer]
                     |
	            true
END OF PROGRAM
```

2. Prune ambiguous (`maybe`) results from the tree
  * pruning is recursive until no ambiguous ends remain

```
maybe a = 2?	  a
		  /\ 
		 /  \
a < 3?		2  false
                |    |
if true,      true  false
                |
print a! [2 (value of a) queued on output buffer]
                |
	      true
```

3. Successful (`true`) results exist, so remove failures (`false` results)

```
maybe a = 2     a
		|
a < 3?		2
                |
if true,      true
                |
print a!     [2 (value of a) queued on output buffer]
                |
	      true
```

4. Collapse pure (non-effectful) nodes

```
print a!     [2 (value of a) queued on output buffer]
```

5. Flush buffered effects

_prints_ `2`
