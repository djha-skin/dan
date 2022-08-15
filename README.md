# Data Austerity Notation


I wanted to make a data language. I saw how all the greats did it: they took
a programming language and removed features.

But I wanted YAML-like multiline strings, except waaaay simpler.

So I made a *DEAD SIMPLE* data language.

It is mostly a subset of r7rs-small scheme, with additions for those multi-line
strings.

It has null (`#0`),  booleans (`#t` for true and `#f` for false),
normal JSON numbers, strings, scheme-like symbols, lists, and plists.

# What is a plist?

It is a list that starts and ends with braces (`{}`) with an even number of
values in it. This can be thought of as the "dictionary" of DAN, where the
values at the even-numbered spots are the "keys" and the values at the
odd-numbered spots are the "values. Keep in mind, it is still a list. Multiple
keys can be present in the plist, since it's just a list. What you do with that
is up to you. Since order matters, you might implement a "last wins" parser,
for example, where objects later in the plist take precedence, or you might
simply implement a multimap and encode its contents with a plist.

Plists can still be treated like lists in APIs if that is what is desired.
They're just another type of list. They are present in the language because
that allows untyped languages to suck in plists as dictionaries if they
want to, which provides a lot of flexibility.

# What is a symbol?

It is a value with a name that is unique. Since it is intended to be a name
of something, it can't be empty. If you are not interested in symbol types,
Just think of them as another way to type a string. If you are, use the symbol
type instead.

# Examples

More to follow.
