CL app devs mostly just want to get to the meat of their program's logic rather
than roll parsers to route. Consequently, there are [many libs to do just that](
https://neurocline.github.io/dev/2015/11/04/command-line-argument-parsing.html).
This document places some prominent ones within the `CLSYNTAX` taxonomy. ***We
can include new ones via PRs if people are interested.***

`endOpt` (the requirement to put `--` before positionals) is likely to be the
least popular strict feature.  A less strict (& probably less unpopular) variant
still more strict than the most lax case is "`--` only when options and
positionals are ***both*** present in a command-line".  Here we will call this
"when-needed opt-arg delimiting" (if literally any parsing library even supports
it).  This optional vs. mandatory distinction did not seem worth a new syntax
flag vs. just a Partial support designation.

Also, the context here is a meta-author/CLuser run-time activating hardening
features.  So, these tables are primarily about expectations of command-users
upon their tools & whatever coherency libs add to that.  So, when in doubt, a
"what library defaults push command authors to do" bias is expected/intentional.

<details><summary><strong>Unix/BSD getopt(3)</strong></summary>

| Feature  | Support | Notes                                                 |
|----------|---------|-------------------------------------------------------|
| `kvSep`  | No      | Core syntax adjacency-separation with a symbol table  |
| `noMix`  | Partial | Some can toggle off of `POSIXLY_CORRECT` presence     |
| `endOpt` | No      | No when-needed opt-arg delimiting either              |
| `typed`  | Partial | String values may/may not be acted upon               |
| `known`  | Yes     | Always On (Caller is free to ignore errors)           |
| `noFold` | No      | Short options can always fold; -ab always means -a -b |
| `valued` | No      | Syntax does not allow explicit bool values            |
| `full`   | N/A     | There are no long options                             |
| `exact`  | N/A     | There are no long options                             |
| `long`   | N/A     | There are no long options                             |
</details>
<details><summary><strong>GNU getopt_long</strong></summary>

| Feature  | Support | Notes                                                 |
|----------|---------|-------------------------------------------------------|
| `kvSep`  | Partial | -o99 -o=99, -o 99 all allowed if any arg is           |
| `noMix`  | Partial | Toggles off of `POSIXLY_CORRECT` presence             |
| `endOpt` | No      | No when-needed opt-arg delimiting either              |
| `typed`  | Partial | String values may/may not be acted upon               |
| `known`  | Yes     | Always On (Caller is free to ignore errors)           |
| `noFold` | No      | Shorts w/no args can always fold; -ab always => -a -b |
| `valued` | No      | Would need 2 whole tables to toggle arg-required-ness |
| `full`   | No      | Hard-coded to always look for abbreviations           |
| `exact`  | Partial | No option key case/style folding => Always On         |
| `long`   | No      | Would need 2 whole tables - one with no shorts        |

Many who may care about this topic are unaware of `full`.  E.g. `ls --hel` works
like `ls --help` and has since at least 1995 and possibly quite earlier.
`getopt_long_only` is a separate entry point for the same option-key defining
data structure but remains more lax than `long` really suggests (i.e., any short
can still be matched).
</details>
<details><summary><strong>Python argparse</strong></summary>

| Feature  | Support | Notes                                                 |
|----------|---------|-------------------------------------------------------|
| `kvSep`  | Partial | Per option key; Can be done with a custom actions     |
| `noMix`  | Partial | Defaults to On; `parse_intermixed_args` => off        |
| `endOpt` | No      | Not w/o many manual tweaks to deal with `--file --`   |
| `typed`  | Partial | String values may/may not be acted upon               |
| `known`  | Partial | Depends upon if CLauthor uses `parse_uknown_args`     |
| `noFold` | Partial | Yes with non-default `prefix_chars='--'`              |
| `valued` | Partial | Per option key; No easy global syntax switch          |
| `full`   | Partial | Per option key `allow_abbrev`; No easy global switch  |
| `exact`  | Partial | No option key case/style folding => Always On         |
| `long`   | Partial | Yes with non-default `prefix_chars='--'`              |

Python is very flexible (in general!).  Most things labeled as "Partial" can be
systematized with higher level wrapper libraries.  Enough such wrappers exist
that adding tables here for `>1` would be potentially overwhelming.

Many "per option key" things *can* be coordinated among the keys via simple
global vars (but typically *are not* so coordinated).  So, it is possible that
[`cg.py`](https://github.com/c-blake/cligen/blob/master/python/cg.py), an
`argparse` wrapper able to share config files with [`cligen`](
https://github.com/c-blake/cligen), will grow most `CLSYNTAX` support.
</details>
<details><summary><strong>Nim https://github.com/c-blake/cligen</strong></summary>

| Feature  | Support | Notes                                                 |
|----------|---------|-------------------------------------------------------|
| `kvSep`  | Yes     | Run-time `CLSYNTAX`,config file; Compile-time `clCfg` |
| `noMix`  | Yes     | Run-time `CLSYNTAX`,config file; Compile-time `clCfg` |
| `endOpt` | Yes     | Run-time `CLSYNTAX`,config file; Compile-time `clCfg` |
| `typed`  | Partial | Always On                                             |
| `known`  | Partial | Always On                                             |
| `noFold` | Yes     | Run-time `CLSYNTAX`,config file; Compile-time `clCfg` |
| `valued` | Yes     | Run-time `CLSYNTAX`,config file; Compile-time `clCfg` |
| `full`   | Yes     | Run-time `CLSYNTAX`,config file; Compile-time `clCfg` |
| `exact`  | Partial | Defaults to inexact; Compile-time switch `cgNoNorm`   |
| `long`   | Yes     | Run-time `CLSYNTAX`,config file; Compile-time `clCfg` |

Here Support==Yes means "can be flipped separately at run-time independently" {
with some reductions (`kvSep` => `noFold`) as mentioned [here](readme.md) }.
</details>
