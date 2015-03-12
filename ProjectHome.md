# RFC 2445 describes a scheme for calendar interoperability. #

This project implements core parts of [RFC 2445](http://google-rfc-2445.googlecode.com/svn/trunk/rfc2445.html) including a parser for recurrence rules, and date lists, and a mechanism for evaluating recurrence rules.

See the [javadoc](http://google-rfc-2445.googlecode.com/svn/trunk/snapshot/docs/index.html) for known issues and caveats.

For questions and the occasional answer, join the [user & developer group](http://groups.google.com/group/google-rfc-2445).


## Features: ##
  * evaluates recurrence rules that don't occur more frequently than daily
  * evaluates groups of recurrence rules and handles exceptions
  * Support for Joda-time dates and java.util.Date

## To-Do: ##
  * Direct support for recurrences more frequent than daily
  * Support for user-defined timezones

## Requirements: ##
  * JDK 1.5

## Maturity: ##
  * Stable -- deployed in a large scale calendaring application
  * Efficient for all common recurrences and reasonably efficient for others (Run tests for micro-benchmarks)
  * No known non-halting behaviors