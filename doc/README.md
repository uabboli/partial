

# partial #

Copyright (c) 2018 John Krukoff

__Version:__ 1.2.0

__Authors:__ John Krukoff ([`github@cultist.org`](mailto:github@cultist.org)).


### Overview ###

![Not actually curry.](curry.jpg)

This is an Erlang parse transform for partial function application, in the
spirit of [Scheme's
SRFI-26](https://srfi.schemers.org/srfi-26/srfi-26.md).

It enables the use of cuts, as represented by the special variable `_`,  to
create anonymous functions with _some_ arguments applied.

For example, to create a function which takes a single integer argument to
convert to a hexadecimal string:

```

> Hex = partial:cut(integer_to_list(_, 16)),
> Hex(255).
"FF"
```


### Getting Started ###

This library is published to [hex.pm](https://hex.pm) as [partial](https://hex.pm/packages/partial). If you're using [rebar3](https://www.rebar3.org/) as your build tool, it can be added
as a dependency to your rebar.config as follows:

```

{deps, [{partial}]}.
```

To use this parse transform, add the following to the top of any module after adding
this library to your application:

```

-compile({parse_transform, partial}).
```

Parse transforms run as part of module compilation. As such, this can not be
used in the shell.


#### Marker Functions ####

Two special functions are provided, which are used by the parse transform to
identify where to apply the transformation. The special cut syntax is only
allowed as an argument to these two functions:

* partial:cut

* partial:cute


The two functions are differentiated by when arguments are evaluated. Cut
evaluates arguments when the function is called. Cute evaluates arguments only
once, when the partial function is constructed.


#### Cuts ####

Cuts are represented by the special variable `_`. As this variable is usually
only legal on the left hand side of a match expression, this use should not
conflict with any existing Erlang syntax.

Cuts may be used for function arguments or for the two forms of function
names. For instance, all three of these usages have the identical result.

```

Rev1 = partial:cut(lists:reverse(_)),
Rev1([1, 2, 3]),
Rev2 = partial:cut(lists:_([1, 2, 3])),
Rev2(reverse),
Rev3 = partial:cut(_([1, 2, 3])),
Rev3(fun lists:reverse/1).
```
All, some or none of the arguments to a function may be cuts.
<h4>Examples</h4>
Partial evaluation provides an easy way to work with higher order functions.
For instance, to double the items in a list:
```
> lists:map(partial:cut(erlang:'*'(_, 2)), [1, 2, 3]).
[2, 4, 6]
```

The difference between `partial:cut/1` and `partial:cute/1` can be seen when
dealing with functions with side effects. For instance, when we try and read a
value from the process dictionary with `get/1`.

```

> Identity = fun (X) -> X end,
> put(example, 1),
> Cut = partial:cut(Identity(get(example))),
> Cute = partial:cute(Identity(get(example))),
> Cut().
1
> Cute().
1
> put(example, 2),
> Cut().
2
> Cute().
1
```

This also makes `partial:cute/1` an easy way to cache expensive computation and
reuse it in later calls.

Finally, partial evaluation can make creating pipelines across multiple
functions easier:

```

> lists:foldl(fun (Step, Acc) -> Step(Acc) end,
              "1,2,3,4,5,6",
              [partial:cut(string:split(_, ",", all)),
               partial:cut(lists:map(fun erlang:list_to_integer/1, _)),
               partial:cut(lists:filter(fun (I) -> I rem 2 == 0 end, _))]).
[2, 4, 6]
```


#### Options ####

The parse transform supports a single option: `partial_allow_local`. This
option allows for a bare `cut/1` or `cute/1` call to be treated the same as
the fully qualified `partial:cut/1` or `partial:cute/1` call.

To enable, either pass as a compiler flag or specify as a compile attribute:

```

-compile(partial_allow_local).
```


### Implementation ###

The transformations are implemented as a replacement of the marker functions
with fun expressions. Only direct literal calls to the marker functions are
found by the parse transform. Indirect calls, as Module:Function(...) or via
erlang:apply/3 are not detected or rewritten and will result in a run time
exception. The result of either transform is _always_ a fun
expression, even when no unevaluated arguments are found.

The underscore variable `_` is used as a placeholder for unevaluated
arguments. It is only legal as a standalone expression, as either the function
name to call or as an argument. Unevaluated arguments are converted to
arguments of the created fun in strict left to right order. There is no
support for reordering or duplicating unevaluated arguments.

partial:cut/1 is implemented as a transformation from:

```

Fun = partial:cut(some_fun(X, Y, _)).
```

To:

```

Fun = fun (Arg1) ->
	some_fun(X, Y, Arg1)
end.
```

`partial:cute/1` is implemented as a transformation from:

```

Fun = partial:cute(some_fun(X, Y, _)).
```

To:

```

Fun = (fun () ->
	Arg1 = X,
	Arg2 = Y,
	fun (Arg3) ->
		some_fun(Arg1, Arg2, Arg3)
	end
end)().
```


### Contributing ###

Please fork the repo and submit a PR. Tests are run via:

```

rebar3 eunit
```

Documentation is autogenerated using edown and edoc via:

```

rebar3 as markdown edoc
```

Some effort has been put into using only the erl_syntax (sometimes via merl)
interfaces to traverse and modify the parse tree. This was done in the hope of
those interfaces being more stable than the raw format, which is explicitely
not guaranteed by the documentation. Any PR which depends on erl_parse forms
will be asked to rewrite using erl_syntax forms.

The application has only been tested with Erlang/OTP 21 on Windows 10. Reports
of success (or failure!) on other versions and operating systems are
appreciated.


### Lineage ###

The most common syntactical element I miss in Erlang is the ability to easily
do partial function application. While researching existing solutions in
Erlang I came across this [erlando issue](https://github.com/rabbitmq/erlando/issues/2) from
2011, which I thought had some pretty good ideas. Notably, I wanted the
ability to use a marker function to make use of the transform explicit and I
wanted to severly limit the scope of where cuts could be used in order to
simplify reasoning about them. As such, it's reasonable to think of this
module as a less powerful version of erlando's cut parse_transform.

The [datum
library](https://github.com/fogfish/datum/blob/master/src/partial.erl) also contains a similar parse transform, which was used for
reference.


### Attribution ###

Image by Cuklev

CC BY-SA 4.0 [`https://creativecommons.org/licenses/by-sa/4.0`](https://creativecommons.org/licenses/by-sa/4.0)


## Modules ##


<table width="100%" border="0" summary="list of modules">
<tr><td><a href="partial.md" class="module">partial</a></td></tr></table>

