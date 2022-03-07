# Welcome to The Clause

[Join our gitter chat room!](https://gitter.im/TheClause/community)

The Clause is an initiative to bring Prolog into the modern web development world. Although Prolog is already quite capable, there is work to be done in writing more documentation, resources, and tutorials that are catered towards the web development industry.

This repo is an accumulation of these efforts. Below you will find resources for learning and using Prolog.

### Support

If you want to support The Clause and help pave the way forward, you can:

- [Join the discussion](https://gitter.im/TheClause/community) on gitter.
- Write articles that are relevant to mainstream web developers.
- Follow The Clause [on Twitter!](https://twitter.com/ThePrologClause)

Feel free to [ping me on gitter](https://gitter.im/fogfish) if you have any questions.

<!-- ## Table of Contents -->

<!-- - [Articles](#Articles) -->
<!-- - [Fundamentals](#Fundamentals) -->
<!-- - [Resources](#Resources) -->

## Introductions to Prolog

- [Prologue to Prolog](https://twitter.com/ThePrologClause/status/1500642533940023302) (36m) ([slides](assets/prologue-to-prolog.pdf))
- [Programmation en Logique (Programming in Logic)](https://www.youtube.com/watch?v=VjJQQTfxuP0) (42m)

## Articles

- Building a user permissions system ([part 1](https://dev.to/gilbert/write-a-user-permissions-system-in-5-lines-of-prolog-mof), [part 2](https://dev.to/theclause/write-a-role-permissions-system-in-14-lines-of-prolog-part-2-371n))
- Setting Up Unit Testing In SWI-Prolog ([link](http://www.paulbrownmagic.com/blog/swi_prolog_unit_testing_env.html))
- Functional Prolog: Map, Filter, and Reduce ([link](https://pbrown.me/blog/functional-prolog-map-filter-and-reduce/))

## Fundamentals

- [Defining custom operators](http://www.amzi.com/AdventureInProlog/a12oper.php)
- [Modules](https://www.swi-prolog.org/pldoc/man?section=modules)
- [Autogenerated Tests](https://www.swi-prolog.org/pldoc/man?section=wizard)

## In Production

- [Production Prolog](https://youtu.be/G_eYTctGZw8) (39:57)
  - [0:00] Intro: Prolog has many solid implementations, logic programming is ridiculously simple
  - [1:54] Demo: Quick intro to Prolog
  - [12:17] Demo of SWI Prolog's incredible debugger
  - [16:36] Code is data, which makes writing tools for Prolog very easy
  - [18:19] Some libraries the speaker uses in production (library(func), library(mavis), library(bencode))
  - [23:01] Constraint logic programming - library(julian) for dates and times. A beauty.
  - [26:34] Concurrency - library(spawn). Wow.
  - [30:04] Deploying saved states, his favorite production trick. `swipl -o foo -c foo.pl`
  - [32:18] Not all sunshine and rainbows

## Resources

- [SWI Prolog forums](https://swi-prolog.discourse.group) - for deeper discussions and troubleshooting
- [Awesome Prolog](https://github.com/klaussinani/awesome-prolog#resources)
- [Full-featured online environment](https://swish.swi-prolog.org) (SWI)
- [JavaScript-based browser environment](http://tau-prolog.org/sandbox/)
- [Prolog cheet sheat](https://github.com/alhassy/PrologCheatSheet)
- The [Association for Logic Programming](http://logicprogramming.org) for those who are interested in the theoretical side of Prolog. If you're looking to do research and/or a PhD, definitely visit that link!
