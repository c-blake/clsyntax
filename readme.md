# 0.Summary
Syntax convenience & safety trade-offs can only be decided in calling contexts.
Many programming languages & parsing libraries aid creation & enforcement of
command-line syntax with diverse internal logic and layering.  Any checking is
better than none.  Therefore a modular specification of what libs can & actually
do is needed.  Its run-time use can satisfy more contexts than any static
compromise.  [This proposal](#3proposal)[^1] aims to help interoperability among
diverse parsing libraries trying to address this problem.

<details><summary><strong>1.Interactive Ergonomics Vs. Durable Clarity</strong>
</summary>
Commands are popular, precise instructions to the computer interactively entered
at a command shell prompt (aka "command-line interface" aka "CLI") usually via
keyboard.  These can be auto-logged into history or copy-pasted into files as
"scripts" in many scripting languages.  Most have parameters, both with named
key-value syntax (aka options) & positional orientation (non-options or "args").
This situation results in command syntax itself having essential tension.

In interactive use, terseness has outsized value because:
 - Keystrokes are expensive
 - Invocation is iterative
 - Editing is incremental, rapid
 - User is solving an *immediate* problem
 - Entry cost is *personal*
 - Feedback is immediate
 - Errors are short-lived

In durable scripts, clarity/verbosity has outsized value because:
 - Keystrokes are amortized over *years*
 - Reading/use by others/later selves dominates writing
 - Errors persist
 - Fail fast/early feedback matters more
 - Defaults & option-key sets can drift
 - Clarity compounds over people/use cases

E.g., at entry a user may prefer `foo -abc` which is only 8 keydowns, created
first with `foo -a<ENTER>`, up-arrow, `foo -ab<ENTER>`, `foo -abc<ENTER>`.  The
final totals 8 keydowns.  This is ***at least 3X*** less than *an optimistic*
24+ keydowns for `foo -a<TAB>=on -b<TAB>=on -c<TAB>=on`.[^2]  Measured in "entry
frustration", 3X can get much worse quickly with typist speed/quality variation,
verbosity of what `a<TAB>` does, how commands are interactively edited, etc.

When made durable, calls become part of a larger program and imprecise syntax
leads to bugs and/or vulnerabilities.  So, pressure piles on for unambiguous
syntax, descriptive keys (what `a<TAB>` completes to) and so on.  Most pithily,
interactive="do what I mean, entered & expressed impatiently" while durable="do
*exactly* what I have always meant on an ongoing basis".

These two contexts simply have very different cost functions *yet* are strongly
coupled operationally - both contexts run **the very same programs** which do
not know their calling contexts.  The hope of this document is informing parsing
what **only run-time calling contexts know** can better satisfy both use cases.
</details>
<details><summary><strong>2.Modular, Strict-ward Parsing Library</strong></summary>

Syntax diversity of files holding commands compounds the core issue.  POSIX
shell syntax is an easy target, but the root cause of many undesired outcomes is
multi-source string synthesis, aka templating.  Templating goes hand-in-hand
with abstracting / parameterizing interactive prototypes, but **only callers
know the _lexical_ context** of "some scripting/templating assemblage".

Libraries to provide command syntax also have surprisingly many little choices
to make which *interact* with templating and its mistake category risks.
Options and positionals need distinguishing as well as keys and values and both
long & short syntax boolean "flags" have available ergonomic optimizations, if
short flags are even in play.

Any modular parameterization is effectively a syntax choice taxonomy.  This is
more sensible if begun with the most lax, layering on strictness checks because
A) this is a usual time-order of entry and B) it also fits with "block/check one
more thing" thinking & coding of parser libs/computer programming itself.

This taxonomy must allow fine-grained expression of syntax constraints on end
users because A) *some* strictness or checking along various independent axes is
almost always better than none at all, B) many parsing libraries with diverse
internal structure & logic [already exist or are yet to be written](
https://neurocline.github.io/dev/2015/11/04/command-line-argument-parsing.html)
-- some constraints will be easy-to-automatic/implicit while others will be
hard-to-impossible, and finally C) user-tolerance even when targeting durability
will vary (e.g. "never template like XYZ" may be a higher-level rule).  Being
*too* fine-grained may make it harder to use than it is worth, though.</details>

# 3.Proposal
The choice for a run-time switch in this proposal is the highly portable one of
environmental variables, like [`NO_COLOR`](https://no-color.org/).  This is easy
enough on both users & programmers.[^3] While being inheritable/globally scoped
and low visibility can be problems, here that is *actively wanted* since the
idea is specifically to run-time activate syntactic strictness across a whole
assembly of durably expressed and perhaps dynamically nested programs, written
against various parsing libs in various programming languages

The name `CLSYNTAX` seems brief & to the point.  Each feature removes a distinct
class of ambiguity common to permissive CLI syntax.  The taxonomy/"type system
for syntax itself" is a set of feature substrings:
| Feature  | Description                                                     |
|----------|-----------------------------------------------------------------|
| `kvSep`  | **means:**  key-value separator required (`--key=value`)        |
|          | **motive:** Avoid flag/value splitting & bool-value ambiguity   |
| `noMix`  | **means:**  first non-option ends option parsing                |
|          | **motive:** Avoid positionals interpreted as options            |
| `endOpt` | **means:** `--` *must* precede positionals                      |
|          | **motive:** Avoid positionals interpreted as options            |
| `typed`  | **means:**  declared types enforced                             |
|          | **motive:** Avoid malformed values (`--int=0.5 -p="443 extra"`) |
| `known`  | **means:**  unknown option keys rejected                        |
|          | **motive:** Avoid typos (`--mistkae`) "succeeding" silently     |
| `noFold` | **means:**  no short-flag folding (`-a -b`, not `-ab`)          |
|          | **motive:** Avoid folded-value ambiguity (`-a$VALUE`)           |
| `valued` | **means:**  `bool` flags require explicit values (`--flag=on`)  |
|          | **motive:** Avoid toggle/default ambiguity                      |
| `full`   | **means:**  no key abbreviations (`--verbose`, not `--ver`)     |
|          | **motive:** Avoid ambiguity from future option keys             |
| `exact`  | **means:**  case-(&style)-sensitive option keys                 |
|          | **motive:** Avoid `isLand` vs `island` ambiguity                |
| `long`   | **means:**  long-form `--` syntax only (`--key`, not `-k`)      |
|          | **motive:** Avoid multiple-spellings of option keys             |
| `strict` | **means:**  all of the above flags are activated (like -Wall)   |
|          | **motive:** Avoid long-form entry of the most strict set        |
<details><summary><strong>4.Details & Clarifications</strong></summary>

Feature presence tests are case-sensitive substring search on `CLSYNTAX` - no
tokenization to specify, but sensible spelling is suggested![^4]  This is
intended to be set in non-interactive ways. **Partial** support or various
features being always-on **is expected** for some libs, but some extra checking
is better than none.  Incremental adoption is both valuable and expected.  To
future-proof this idea itself, unknown feature names should be simply ignored.
The chosen lack of syntax for the `CLSYNTAX` value means new feature names will
never be substrings of any existing ones.

Some clarifications.  `CLSYNTAX` should override library defaults, but config
files can override `CLSYNTAX` and programs themselves can opt-out completely (by
hand-rolling option parsing anyway!).  Command-lines self-specifying their own
syntax is out of scope.  `noMix` & `endOpt` are distinct - `noMix` *implicitly*
ends options before required `--` ends them *explicitly*.

Also, this is not intended to be exhaustive.  E.g., late rather than early
optionals better tracks prevailing trends of how programming language argument
lists are handled, but for good or ill most commands in most OSes that do not
allow options anywhere force them early.  On another front, options headers can
be `[+/]` or like `dd` be "" instead of "-" and "--".  In truth, free parameters
of such (namely two substrings), even if enabled by parsing libs, fit poorly
into the current bag of flags `CLSYNTAX`.  Extending for args within `CLSYNTAX`
itself means agreeing upon yet more syntax which is out of the present scope.
</details>
<details><summary><strong>5.Example Activation</strong></summary>

Activating maximum strictness is simply `export CLSYNTAX=strict`, but any subset
also fits in a 1-line block of POSIX shell:
```sh
export CLSYNTAX=kvSep,noMix,endOpt,typed,known,noFold,valued,full,exact,long
```
or a line of Python (or an easier lib call) after `os.environ` is imported but
before subprocess module work:
```python
environ["CLSYNTAX"]="kvSep noMix endOpt typed known noFold valuedfullexactlong"
```
You could of course set `strict=kvSep,endOpt,..` a custom set of strictness,
`lax=''`, and then toggle `CLSYNTAX=$strict`,  `CLSYNTAX=$lax`.
</details>
<details><summary><strong>6.Example Implementation</strong></summary>

Presence testing is as direct as:
```C
char *clsyntax = getenv("CLSYNTAX");
int noFold = strstr("noFold", clsyntax); /* ... */
```

```Python
clsyntax = os.environ.get("CLSYNTAX", "")
noFold = "noFold" in clsyntax) # ...
```
and so on.  There are only 11 substrings and `CLSYNTAX` itself is also short.
Any increased time relative to a non-allocating tokenization + hash lookups is
in the noise compared to the implicit new program image creation in play.

A full flag-set-conditional working implementation is the Nim
[`cligen`](https://github.com/c-blake/cligen) programs [implement](
https://github.com/c-blake/cligen/blob/master/cligen/parseopt3.nim).  That can
provide *almost* any combination of these rules by default.[^5]  There is no way
to disable `known` (known options only) which is just always on.  `typed` (type
validation enforced) is always in effect for all parameters.  Almost any system
that allows string parameters is vulnerable to stringly-typing issues, though.

As mentioned, partial support is fine, but more is better.  Here is a probably
mostly correct [survey](survey.md) of prominent command-line toolkits.
</details>
<details><summary><strong>7.Limitations & Counters</strong></summary>

This kind of run-time multi-syntax flexibility itself introduces a new tension -
that of generating good enough vs. optimal error messages or at least portable
error messages.  E.g., in the very strictest mode `-ab` may be incorrect for
many reasons (the leading `--` is `-`, the separator '=/:/etc' is missing, there
is no key `a` or no key `ab`, etc.)  The logical path from lax to strict may be
very different in different parsers and we want that to be ok for adoption.  So,
while `cligen` may emit the message `"Short options are run-time disallowed at
`a`"`, other ways to parse are easily imagined where the library author might
decide the "more primary" mistake is that `'a'` (or `ab`) is not a key at all.
Similar error messages for the same erroneous string in the same mode basically
has to be out of scope.

</details>
<details><summary> <strong>8.Related Work</strong></summary>

Recognition of the essential tension goes back to early Bell Labs shell
discussion. _The Unix Programming Environment_ by Kernighan & Pike is quoted
[here](https://stackoverflow.com/questions/24983901/command-line-argument-program-option-parsing-styles-and-specification)
Other quotes within the same book call the syntax "capricious" and of a "taste
for anarchy".  Early IBM, MIT, BBN, etc. HCI work surely also discussed it.

I thought of this solution independently and could find no idea more similar
than `getopt_long` interpretation of `POSIXLY_CORRECT` (which I didn't even know
about until I started searching for features like this).  That's a broader idea,
though, adjusting other interoperability knobs like 1024-byte vs 512-byte block
block units.  The equivalent here, `CLSYNTAX=noMix` is more specific/targeted.

That said, I'm happy to cite things if told.  It is also a work-in-progress and
I'm happy to credit useful contributions if github's PR system doesn't work for
you.  There is, of course, much discussion of CL syntax & semantic problems
(like repeats replacing or collecting, or the key-name auto-duplications like
`--no-` for bool flags), but such semantics is out-of-scope here.
</details>

[^1]: This document began life as
https://github.com/nim-lang/Nim/pull/25499#issuecomment-3884473108 based on a
[May 2020/v0.9.46](https://github.com/c-blake/cligen/releases/tag/v0.9.46)
`cligen` feature promoting a 2016 compile-time settable `requireSeparator` (here
`kvSep`) to environment-based run-time.

[^2]: `echo "foo -at=on -bt=on -ct=on" |`
[`keydowns`](https://github.com/c-blake/bu/blob/main/doc/keydowns.md)` using 't'
as a stand-in for the TAB key even though, at least on my keyboard layout, both
TAB and '=' are *much* more "strained finger reach" than index-finger 't'.  This
also assumes *unique completions*.

[^3]: It might be nicer to automatically identify if command strings "come from
files", but "files are strings" equivalences work against this and terminal
tests also fail since programs often inherit those. 

[^4]: Terse `CLSYNTAX=ketwsnvfxl` ('s' for option-positional {s}egregation) may
be ok, but is maybe off-style using `CLSYNTAX` at all (once doing lax-\>strict).

[^5]: If they really want, a program author can pass their own `ClCfg` object
which overrides the default interpretation of `$CLSYNTAX`.
Usual approaches
hard-wire such choices, trying to balance aforementioned incompatible concerns.
Given the mentioned dynamic, lexical, and even mental context diversity,
deferring such choice to run-time parameters seems a better way forward.
