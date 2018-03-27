# purpleface
The purpleface programming language

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
  * `maybe a =? 1 + 2?` may or may not bind `a` to a value that is the sum of 1 and a number that may or may not be 2, so the possible values are 1, 3, maybe, and false
5. Optional values are all quantum-spooky. They do not resolve to concrete values until a concrete value is required.
  * The whole program is generally lazy evaluated. 
  * At runtime, `maybe` bindings create multiple continuations.
  * Only continuations which result in valid result are retained.

```
maybe a = 2?
  if (a < 3) 
     print a
```

    * first, binds `a` to a value which may be `2`, `maybe`, or `false`
    * in each of the 3 continuations, evaluates `a < 3`, creating another new continuation for each possible result
      * all conditional expressions return an optional value, so this evaluates to:
        * `true` in the `a = 2` continuation
	* `maybe` in the `a = maybe` continuation
	* `false` in the `a = false` continuation
      * since only the `true` condition is handled, only the "`a < 3` is `true`" continuation will reach the print statement.
    * In the "`a < 3` is `true`" continuation, the `print` statement should print `2`, since `false` or `maybe` cannot be less than 3, and `print` requires a value which is true.
