# Data Austerity Notation

A data notation with the bare essentials.

## Design Goals

- Textual data format
- Support API communication a la JSON
- Support config files and document embedding a la YAML
- Support symbols and keywords in lisps a la EDN
- Support incremental parsing better
- Support typed languages better
- Support better schema checking. It doesn't support it better than JSON,
  but it does encourage it more.
## Syntax

## Comments

Comments must be on their own line. The line begins with an arbitrary
number of blankspace characters. The comment itself starts with a semi-colon
(`;`) character and continues to the end of the line. The entire line
is removed as if it never existed.

## Language Types

### Null

Null, nil or nothing is written `#n`.

### Boolean

These are written `#t` and `#f` for false.

### Number

These are written just as JSON numbers, with the addition of the `T` or `t`
characters as stand-ins for the exponent:

`[-+](0|[1-9][0-9]*)(.[0-9]+)?([eE]([0-9]+|[tT]))?`

The `"t"` means `"top"` and means that all the exponent bits in the number
are set to 1. This allows the language to represent IEEE 754
infinity and NaN values. NaN has its exponent set to `t` and is non-zero,
while infinities have all zeroes set for the integer and radix and the exponent
set to `t`. Thus, the canonical way to write the following in DAN is:

| Value             | Canonical Representation |
|-------------------|--------------------------|
| Positive Infinity | `0et`                    |
| Negative Infinity | `-0et`                   |
| Not a Number      | `1et`                    |

Note the positive sign is implied (though specifying it is still valid).

The above representations of the values should be used when generating DAN.

All "Not a Number" values are considered to be equivalent.

### Strings

#### Quoted strings

Quoted strings are quoted with double quotes (`"`).

Scheme r7rs-small strings:

- Can escape:
  - The pipe `\|`
  - The quote `\"`
  - Blank space up to and including the next line delimiter (escape literal
    newlines)

- Can write:
  - The alarm character using `\a`
  - The backspace character using `\b`
  - The tab character using `\t`
  - The newline character using `\n`
  - The carriage return using `\r`
  - Any unicode character using `\xXX+;`; that is, using an arbitrary
    number of hexadecimal digits delimited by an ending semi-colon
    (may be checked to be within the range of `0-10FFFF` by the implementation)

#### Verbatim Strings

Verbatim strings start with a verbatim mark, `#]`. Then blankspace and a line
delimiter follow. Ignoring comment lines, subsequent verbatim marks have
the contents of the line after the mark appended to the multi-line string,
including the line delimiter. The last line delimiter is not
included in the final product.

Thus:

```
    #]
```

Represents the empty string.

This:

```
    #]
    #]
```

Represents the quoted string `"\n"`

This:

```
    #]
    ; This line is very interesting.
    #]12
    ; So is this one.
    #]34
```

Represents the string `"\n12\n34"`.

#### Prose strings

Prose strings are the same as verbatim strings, except that unbroken substrings
of blankspace and whitespace characters that are are found between non-space
characters are replaced with a single space (` `).

This:

```
    #>
```

Still represents the empty string.

This:

```
    #>    As I walked
    #>    slowly by
    #>
```

Represents the quoted string `"    As I walked slowly by  \n"`

### Symbols

Symbols are named entities that are unique if their names are unique. They are
named by the characters in their name. They are semantically something like
enum values with names attached to them.

To quote the R7RS standard:

> An identifier is any sequence of letters, digits, and “ex- tended identifier
> characters” provided that it does not have a prefix which is a valid number.
> However, the . token (a single period) used in the list syntax is not an
> identifier. All implementations of Scheme must support the following extended
> identifier characters:
>
> ```
> ! $ % & * + - . / : < = > ? @ ^ _ ~
> ```
>
> Alternatively, an
> identifier can be represented by a se- quence of zero or more characters
> enclosed within vertical lines (`|`), analogous to string literals.

They can, as in the above quote, appear bear or quoted. If bear, the following
rules apply:

- They can't start with a numerical character. They must start with
  one of the alphabetic ASCII characters or one of the symbols above.
- If they start with a period, a plus, or a minus, the second character
  can't be a number.
- Symbols cannot be a single period (`.`).

Quoted symbols are quoted using pipe characters `|` and escaping follows
the same rules as for strings.

### Lists

Lists begin with a `(` character and end with a `)` character.

## Small Example

```
(
    property-type park
    name "Goblin Valley"
    is-desert #t
    current-month-avg-high-temp 92.0
    yearly-visitors 500000
    jurisdiction state
    same-state-parks (
        "Arches"
        ;; Very Nice and underrated.
        "Coral Pink Sand Dunes"
        "Grand Canyon"
    )

    ascii-face
    #].-------.
    #]| *   * |
    #]|   U   |
    #]| =   = |
    #]|  ---  |
    #]\-------/

    quote
    #>but
    #> why
)
```

## What is a plist?

It is a list with an even number of values in it. This can be thought of as the
"dictionary" of DAN, where the values at the even-numbered spots are the "keys"
and the values at the odd-numbered spots are the "values. Keep in mind, it is
still a list. Multiple keys can be present in the plist, since it's just a
list. What you do with that is up to you. Since order matters, you might
implement a "first wins" parser, for example, where objects later in the plist
take precedence, or you might simply implement a multimap and encode its
contents with a plist.

Plists can still be treated like lists in APIs if that is what is desired.
They're just another type of list. They are present in the language because
that allows untyped languages to suck in plists as dictionaries if they
want to, which provides a lot of flexibility.

There are several advantages to plists over unordered maps (in a serialization
format, at least):

  - They have order. Imagine being able to guarantee in JSON RPC that the
    `jsonrpc` version key is the first key to appear and that the
    `method` key was right after it, followed by the `params` key.
    All of a sudden, JSON RPC would be much easier to support in typed
    languages, because incremental parsing is possible, making polymorphism
    much easier without requiring reflection or untyped/type-erasure parsing.
  - They don't require their "keys" to be hashable or even orderable,
    since it is just a list. Keys can be anything.
  - They don't require their key/value pairs to appear only once. When
    converting a plist to a map internally, it is advisable that "first
    wins", but this is not a requirement.

## What Might APIs look like

### Typed languages

An incremental parser something like:

```
listin(&stream) // Enter a list under cursor
listout(&stream) // Exit current list, skipping all values while doing so
find(&stream, vset) // Move the cursor until it points to one of the values
                    // matching one in the given vset.
skip(&stream, num) // Skip <num> number of values.
expnul(&stream) // Expect a null and consume it if it's there.
expint(&stream) // Expect a int and consume it if it's there.
explng(&stream) // Expect a long and consume it if it's there.
expflt(&stream) // Expect a float and consume it if it's there.
expdbl(&stream) // Expect a double and consume it if it's there.
expstr(&stream) // Expect a string and consume it if it's there.
expsym(&stream) // Expect a symbol and consume it if it's there.
scan(&stream, scanstr, ...) // Scanf-like experience to extract values from the
                            // list.
```

I could add these arguments to the scan function:

"%d_1 %d_1 %d_1 %d_2" or whatever
The number at the end would signify that that thing is part of a struct
I pass the struct in as another argument between args for 1 2 and 3 but before
4
I pass in the struct, followed by its size
so

`scan(stream,scan,&bufstruct)` could add these arguments to the scan function:

`"_ [ %d %d %d ] [ %d %d ] ` or whatever
The brackets would signify that that everything inside is part of a struct
I pass the struct in as another argument between args for 1 2 and 3 but before
4
I pass in the struct, followed by its size
so

```
scan(stream,scan,&bufstruct.a,&bufstruct.b,&bufstruct.c,&bufstruct,sizeof(bufstruct_t), &buf2.x, &buf2, sizeof(buf2_t), &bufstruct_list, &buf2_list)
```
When the list runs out, the loop is jumped out of that is running the scan.

Scan-f could be super good here.

The above function could consume two lists of structs using `memcpy()`.

Notably, in golang, nothing will change. DAN will be supported just as well
as YAML or JSON is because of the nature and shape of their
`Unmarshall` and `Marshall` functions. A similar API to those would work
beautifully.

### Untyped Languages

An experience similar to Golang's might be used in Python. The important thing
to note is that, while it is more difficult to parse in typed languages,
it may be advantageous. You have to at least know what you want to extract.
I submit you had to know that anyway. An API should be able to find a way
to make this easy to declare up-front.

```
# TODO
```

Notice, in languages that don't support symbols, they may be safely handled as
strings, or not, depending on the schema provided.

### Clojure, Common Lisp

These lisps don't share the same symbol definition as scheme, and they have
keywords which scheme does not have. The good news is, all valid keywords and
symbols in Clojure and Common Lisp are also scheme symbols. This was chosen
on purpose. Now all lisps are well supported, as well as any other language
using keywords. Just use a colon in the symbol name as normal.

## How is this better?

Since parsers have no way to distinguish between lists, sets, maps, or
vectors -- everything is just a list -- the program must know the schema
before hand and declare it. This is safer anyway. The right API will
make this easy, regardless of the language. The upside is that since
everything has order, typed languages have a much easier time since
incremental parsing is possible.


