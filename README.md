# Data Asceticist Notation (DAN). Also, Dan's Ascetecist Notation.

(Also my name is Dan.)

I wanted to make a data language. I saw how all the greats did it: they took
a programming language and removed features.

But I wanted YAML-like multiline strings, except waaaay simpler.

So I made a *DEAD SIMPLE* data language.

It is mostly a subset of r7rs scheme, with additions for those multi-line
strings.

It has booleans (`#t` for true and `#f` for false), normal JSON numbers,
strings, scheme-like symbols, and lists.

Notably, it's missing a null value and a key-value set-up. If you want
a dictionary, just use a plist (a list where even-numbered indexed things
are keys and odd-numbered things are values). Just as JSON numbers don't have
a type associated with them, but the type is up to the reader, so the type of
datastructure that the list represents is up to the reader. It could be a dict,
list, vector, whatever. it's just a sequence of stuff, you do what you want
with it.

ABNF included.