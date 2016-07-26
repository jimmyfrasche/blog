# On the Use and Abuse of internal/ Packages

Recently, I've run across people who are unaware of the internal/ package mechanism and people who object to it. I am a fan of internal/ packages but they can be abused, so I'm going to describe the mechanism, its uses, and how to use it fruitfully.


## What are internal/ packages?

The internal/ package mechanism was added to the go tool in Go 1.4, but only for the standard library. The feature was opened to all in Go 1.5.

I say go tool specifically as it is not a feature of the language, but of the build system, as with packages in vendor/. The Go compiler knows nothing of this. You won't find it in the language spec. The go tool implements the feature in its entirety.

As of 1.5, the go tool does not allow importing an internal/ package except by packages that share the same root. That is, given a/b/internal/c, c may be imported from a/b/d and a/b/e, since they share the root a/b, but not by a/f or foo/bar, as they are not contained in a/b.

See https://golang.org/s/go14internal for the full (accepted and implemented) proposal.

As a convenience, I will refer to packages that are not under internal/ as "public". This term isn't canonical or meaningful outside this document. It is intended only as a shorthand for this discussion.


## What goes in internal/ packages?

There are three use cases.
- Code that only exists to support the other packages in a tree and is not meant for public consumption.
- Miscellaneous code that doesn't really deserve a public API but just doesn't fit anywhere else.
- Code that may or may not deserve a public API.

Let's investigate them in turn.


### Shared code that should not have a public API

Before internal/ if you had shared code there were three options:
- have multiple copies of the code in various packages
- have a public package that exists for no reason other than to support your other packages
- only publish a single package that contained semi-related but disparate functionality

None of those are great options. Here, internal/ is a clear win.

Just make an internal/ package that your other packages can use. No duplicating code, no leaking implementation details, no poorly factored uberpackages.

This is the most important use case in that there was no real option before the introduction of the internal/ mechanism. The others are nice bonuses of the general mechanism.


### Miscellaneous code

This is the last case where the number of packages that need access to the code is 1.

Junk drawer files are a common practice across programming languages. You have a util.go and you throw random code in it that doesn't fit anywhere else. These introduce irrelevant details into your package.

With internal/ you have the option of throwing this random code in internal/ packages. If you have a few random routines for integer math, you can put them in internal/intmath instead of adding to an ever growing util.go (or a separate mathutil.go, which is the best practice if you want to stick with junk drawer files).

The benefit here may not be immediately clear, but it avoids cluttering your package with irrelevant details and defines a clear boundary that let's you reason about the separate packages as separate units.

This is not always possible. Sometimes those random utilities need deep knowledge of the inner workings of your package or would require too many contortions to avoid introducing a cyclic dependency. If you push those you can into internal/ packages, the fact that some generic looking utility code hasn't been signals to the reader that this utility code is somehow indivisible from the current package.


### Code that may not deserve a public API

Sometimes you write a package that you need but you're not sure if anyone else ever will or that, while you need it now, you're not sure if it's ready for prime time. If you put such a package under internal/, you can always gomvpkg it out later. If you start with it public, you're stuck maintaining that package now and have to worry about breaking its API.

Incubating packages in internal/ costs nothing. Best case, you never need to move it. Worst case, you move it outside internal/ with a single invocation of gomvpkg.


## Dos and Don'ts.

I don't claim to have ever followed this advice perfectly or that it universally applies to every situation, but as I've worked with internal/ packages I've learned from my mistakes and found the following useful:

Use internal/ packages.

Use internal/ packages even if you only have one public package or command in your repository.

Never put code in internal/ itself. Always, always, always create a well named package under internal/. If you have a set of utilities for working with regular expressions, put that in internal/re or internal/regexputil.

Consider junk drawer files a code smell and prefer internal/ packages.

Don't move from junk drawer files to junk drawer packages. Even if a package is internal/ it should still do one thing, no matter how large or small that one thing is. Remember that there is no minimum size for a package. Exporting a single function is fine.

Don't put code in an internal/ package just because it would otherwise be unexported. They are for code that another package needs but that does not fall within its scope.

Avoid leaking types, functions, etc. out of internal/ packages. Your public packages should look and act like the internal/ packages are not there.

Treat your internal/ packages as you would any other package because they are just packages:
- Name the package well.
- Don't export irrelevant functionality.
- Document the package and everything exported in it well.
- Write tests if the logic is complicated.
If you end up moving the package outside of internal/ you're already done. If you never do at least it plays well with godoc for when you forget what something was supposed to do.

Refactor internal/ packages mercilessly. If you stop using a function or type, delete it. If you stop using the package entirely, delete it. Split it in half. Merge it with another internal/ package. It doesn't matter if you break the API since you own all the code that can access it.


## Things you can do

I don't really have enough data to make a determination and commit to these, but I think these are probably good ideas that largely follow from the above and I will try them when presented with an opportunity. At any rate, the best way to find the limits of a technique is to go overboard with it and then, in hindsight, figure out where you crossed the line.

If you have multiple packages, keep internal/ as close to the packages using that functionality as possible. If you have
```
gihub.com/user/myrepo/
	|
	+- a/
	|
	+- b/
```
and you want to add a package internal/c, that only applies to a and not b, do
```
gihub.com/user/myrepo/
	|
	+- a/
	|  |
	|  +- internal/c
	|
	+- b/
```
not
```
gihub.com/user/myrepo/
	|
	+- a/
	|
	+- b/
	|
	+- internal/c
```

Have more than one internal/ directory in your repository if it makes sense.

If you really need it, internal/ nests with the same rules: you can have mypkg/internal/regexputil/internal/indexutil.


## Conclusion

The internal/ package mechanism is a handy way to organize code. It lets you create packages without having to expose them outside of your codebase. It lets you nicely factor a set of public packages without having to expose internal extension points or shared functionality. It lets you to provide clear boundaries around utility code. It lets you incubate packages you need now and may want to expose tomorrow.

I think you should go nuts with internal/ packages.

The important thing to remember is that internal/ packages are still just packages. If you don't treat them with the same discipline you would a public package, the readability of your code will suffer just as much as if you were using a third-party package with a bad API.
