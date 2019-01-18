# partial #

![Not actually curry.](doc/curry.jpg)

### Overview ###

This is an Erlang parse transform for partial function application, in the
spirit of [Scheme's
SRFI-26](https://srfi.schemers.org/srfi-26/srfi-26.md).

It enable the use of cuts to create anonymous functions with _some_
arguments applied. For instance:

```

Hex = partial:cut(integer_to_list(_, 16)).
```

Creates a single argument function which will convert integers to hexadecimal
strings. It is equivalent to:

```

Hex = fun (Integer) -> integer_to_list(Integer, 16) end.
```

A cute variant is also supplied, which is short for "cut evaluated". This
version evaluates the arguments supplied when it is declared, instead of when
it's called. For instance:

```

FormatNow = partial:cute(io:format("~w~n", [erlang:now()])).
```

Will always print the same timestamp when called, while:

```

FormatNow = partial:cut(io:format("~w~n", [erlang:now()])).
```

Will print the current timestamp.

To use this transform, add the following to the top of any module after adding
this library to your application:

```

-compile({parse_transform, partial}).
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
<tr><td><a href="http://github.com/jkrukoff/partial/blob/master/doc/partial.md" class="module">partial</a></td></tr></table>

