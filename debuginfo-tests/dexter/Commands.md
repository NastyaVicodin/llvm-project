# Dexter commands

* [DexExpectProgramState](Commands.md#DexExpectProgramState)
* [DexExpectStepKind](Commands.md#DexExpectStepKind)
* [DexExpectStepOrder](Commands.md#DexExpectStepOrder)
* [DexExpectWatchType](Commands.md#DexExpectWatchType)
* [DexExpectWatchValue](Commands.md#DexExpectWatchValue)
* [DexUnreachable](Commands.md#DexUnreachable)
* [DexLimitSteps](Commands.md#DexLimitSteps)
* [DexLabel](Commands.md#DexLabel)
* [DexWatch](Commands.md#DexWatch)

---
## DexExpectProgramState
    DexExpectProgramState(state [,**times])

    Args:
        state (dict): { 'frames': [
                        {
                          # StackFrame #
                          'function': name (str),
                          'is_inlined': bool,
                          'location': {
                            # SourceLocation #
                            'lineno': int,
                            'path': str,
                            'column': int,
                          },
                          'watches': {
                            expr (str): value (str),
                            expr (str): {
                              'value': str,
                              'type_name': str,
                              'could_evaluate': bool,
                              'is_optimized_away': bool,
                              'is_irretrievable': bool,
			    }
                          },
                        }
                      ]}

    Keyword args:
        times (int): Minimum number of times this state pattern is expected to
             be seen. Defaults to 1. Can be 0.

### Description
Expect to see a given program `state` a certain number of `times`.

For every debugger step the reported state is compared with the expected state.
To consider the states a match:

* The `SourceLocation` must match in both states. Omitted fields in the
`SourceLocation` dictionary are ignored; they always match.
* Each `expr` in `watches` in the expected state can either be a dictionary
with the fields shown above, or a string representing its value. In either
case, the actual value of `expr` in the debugger must match.
* The function name and inline status are not considered.

### Heuristic
[TODO]


---
## DexExpectStepKind
    DexExpectStepKind(kind, times)

    Args:
      kind (str): Expected step kind.
      times (int): Expected number of encounters.

### Description
Expect to see a particular step `kind` a number of `times` while stepping
through the program.

`kind` must be one of:

`FUNC`: The first step into a function which is defined in the test
directory.</br>
`FUNC_EXTERNAL`: A step over a function which is not defined in the test
directory.</br>
`FUNC_UNKNOWN`: The first step over a function an unknown definition
location.</br>
`VERTICAL_FORWARD`: A step onto a line after the previous step line in this
frame.</br>
`VERTICAL_BACKWARD`: A step onto a line before the previous step line in
this frame.</br>
`HORIZONTAL_FORWARD`: A step forward on the same line as the previous step in
this frame.</br>
`HORIZONTAL_BACKWARD`: A step backward on the same line as the previous step
in this frame.</br>
`SAME`: A step onto the same line and column as the previous step in this
frame.</br>

### Heuristic
[TODO]


---
## DexExpectStepOrder
    DexExpectStepOrder(*order)

    Arg list:
      order (int): One or more indices.

### Description
Expect the line every `DexExpectStepOrder` is found on to be stepped on in
`order`. Each instance must have a set of unique ascending indices.

### Heuristic
[TODO]


---
## DexExpectWatchType
    DexExpectWatchType(expr, *types [,**from_line=1][,**to_line=Max]
                        [,**on_line][,**require_in_order=True])

    Args:
        expr (str): expression to evaluate.

    Arg list:
        types (str): At least one expected type. NOTE: string type.

    Keyword args:
        from_line (int): Evaluate the expression from this line. Defaults to 1.
        to_line (int): Evaluate the expression to this line. Defaults to end of
            source.
        on_line (int): Only evaluate the expression on this line. If provided,
            this overrides from_line and to_line.
        require_in_order (bool): If False the values can appear in any order.

### Description
Expect the expression `expr` to evaluate be evaluated and have each evaluation's
type checked against the list of `types`

### Heuristic
[TODO]


---
## DexExpectWatchValue
    DexExpectWatchValue(expr, *values [,**from_line=1][,**to_line=Max]
                        [,**on_line][,**require_in_order=True])

    Args:
        expr (str): expression to evaluate.

    Arg list:
        values (str): At least one expected value. NOTE: string type.

    Keyword args:
        from_line (int): Evaluate the expression from this line. Defaults to 1.
        to_line (int): Evaluate the expression to this line. Defaults to end of
            source.
        on_line (int): Only evaluate the expression on this line. If provided,
            this overrides from_line and to_line.
        require_in_order (bool): If False the values can appear in any order.

### Description
Expect the expression `expr` to evaluate to the list of `values`
sequentially.

### Heuristic
[TODO]


---
## DexUnreachable
    DexUnreachable()

### Description
Expect the source line this is found on will never be stepped on to.

### Heuristic
[TODO]


----
## DexLimitSteps
    DexLimitSteps([expr, *values][, **from_line=1][,**to_line=Max]
                  [,**on_line][,**hit_count])

    Args:
        expr (str): variable or value to compare.

    Arg list:
        values (str): At least one potential value the expr may evaluate to.

    Keyword args:
        from_line (int): Define the start of the limited step range.
        to_line (int): Define the end of the limited step range.
        on_line (int): Define a range with length 1 starting and ending on the
                       same line.
        hit_count (int): If provided, limit the number of times the command
                         triggers.

### Description
Define a limited stepping range that may be predicated on a condition. When the
leading line is stepped on and any condition '(expr) == (values[n])' is true or
there are no conditions, set a range of temporary breakpoints within the test
file defined by the range 'from_line' and 'to_line' or 'on_line'. This only
happens 'hit_count' number of times if the argument is provided.

The condition is only evaluated on the line 'from_line' or 'on_line'. If the
condition is not true at the start of the range, or that line is never stepped
onto, the whole range is ignored.

DexLimitSteps commands are useful for reducing the amount of steps gathered in
large test cases that would normally take much longer to complete.

----
## DexLabel
    DexLabel(name [, **on_line])

    Args:
        name (str): A unique name for this line.

    Keyword args:
        on_line (int): Specify a line number to label.

### Description
Name the line this command is found on or 'on_line' if it is provided. Line
names can be converted to line numbers with the `ref(str)` function. For
example, `DexExpectWatchValues(..., on_line=ref('my_line_name'))`. Use
arithmetic operators to get offsets from labels:

    DexExpectWatchValues(..., on_line=ref('my_line_name') + 3)
    DexExpectWatchValues(..., on_line=ref('my_line_name') - 5)


### Heuristic
This command does not contribute to the heuristic score.


---
## DexWatch
    DexWatch(*expressions)

    Arg list:
        expressions (str): `expression` to evaluate on this line.

### Description
[Deprecated] Evaluate each given `expression` when the debugger steps onto the
line this command is found on.

### Heuristic
[Deprecated]
