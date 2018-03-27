# purpleface
The purpleface programming language

_Proposed Features:_
- Function Inference
- Boolean datatype and operations replaced with optional data type
.- true | false | maybe
.- every datatype is derived from 'true'
.- the 'maybe' keyword casts expressions to a 'maybe'
.- 'maybe 2?' returns a value that is either 2 (and 'true') or not 2 (and therefore, 'maybe')
.- 'maybe a =? 1 + 2?' may or may not bind 'a' to a value that is the sum of 1 and a number that may or may not be 2, so the possible values are 1, 3, maybe, and false
- Lambdas cannot be bound to a variable
.- will result in a compiler error of "Nice Try"
- 
