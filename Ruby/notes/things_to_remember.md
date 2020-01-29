# Things to Remember

A collection of things to remember that I've found through reading documentation, articles, and general researching.

___

## Table of Contents

| Name | Description | Name | Description |
| ---| --- | --- | --- |
| [Operators](#operators) |  | | |

___

### Operators

#### Double Pipe / Or Equals

**Symbol:** `||=`

In `a = a || b`, `a` is set to something by the statement on every run, whereas with `a || a = b`, `a` is only set if `a` is logically false. 

Essentially this statement evaluates if the variable on the left sign of the operator is true. If it is true, then the statement on the right side of the operator is not carried out.

##### ex

```ruby
a = nil
b = 20
a ||= b
a       # => 20
```

> _The above example demonstrates that if a is falsy, then the value of a is set to b_

___
