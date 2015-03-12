# Building from source #

The downloads page only contains stable releases, so if you want bleeding edge features you will need to build your own JAR from source.

First, [checkout](http://code.google.com/p/google-rfc-2445/source/checkout) the source via [SVN](http://svnbook.red-bean.com/en/1.6/svn-book.html#svn.tour).

This project uses [ANT](http://ant.apache.org/manual/install.html) to build.

From the `google-rfc-2445` directory
```
ant -p
```
should display help information for the project.

```
An implementation of RFC2445 recurrence rules
Main targets:

Other targets:

 clean
 default
 docs
 rfc2445
 rfc2445-no-joda
 runtests
 tests
Default target: default
```

Then pick the target you want.
```
ant docs
```
will build the JavaDoc pages for the project and make them available under `google-rfc-2445/docs/`, and
```
ant rfc2445
```
will make JARs available under `google-rfc-2445/jars/` that you can use with your project.