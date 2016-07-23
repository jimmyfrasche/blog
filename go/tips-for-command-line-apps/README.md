# Tips for writing command line apps in Go

Some of these may apply generally, at least in some circumstances. I've probably ignored this advice more than I've followed it: that's because I've developed these techniques and practices after looking back at programs I've written and thinking "oh I should have done that", but then been too lazy to refactor because, eh, it works—maybe next time. These are guidelines, not strictures.

(I am going to discuss logging and flags in terms of the stdlib packages, but if you don't care for them just substitute whatever you use.)


## Avoid recreating the environment

os.Stdin and os.Stdout are good defaults. Unless you have an excellent reason not to use them, use them.

If the shell or OS implements something, that means you don't have to.


## If you use color or cursor addressing:

- Have a way to disable it
- Implicitly toggle that if you are not writing to a tty, see isatty(3).

Bonus: offer an option for users of terminals with a light color scheme. Red on white is not readable.


## Always use log

The stdlib log package defaults to writing os.Stderr. It is threadsafe. It has helper methods for most common things.

For most commands the only annoying thing is that it defaults to a more server friendly output. This can easily be turned off with a call to `log.SetFlags(0)` at the top of main. You can do this in an `init()` function, but do it in `main()` because you shouldn't be logging anything before `main()` runs.

```
func main() {
	log.SetFlags(0)
	//...
}
```


## Use flag even if you don't take arguments

You might later.

It makes it easy to provide a nice usage message that's called appropriately and is easily customized and extended if you do add arguments.

It also catches accidental flags passed to your command. If your program creates a file with its first arg and someone tries to query the version with -v, they may be confused at why they have a file named -v now. If they actually meant to create a file named -v they can use `prog -- -v`.

```
func main() {
	log.SetFlags(0)

	flag.Usage = func() {
		log.Printf("Usage: %s", os.Args[0])
		log.Print("foos all your bars")
		flag.PrintDefaults()
	}
	flag.Parse()
	//...
}
```

(Note that flag.PrintDefaults() does nothing if no flags are defined and it's easy to forget to put it in if you later add a flag.)


## If you do take arguments, define them in main

This forces you to do two things.

Actually use the flags you define.

Pass the state through the system instead of relying on the presence of globals, making it easier to test your code.

```
func main() {
	log.SetFlags(0)

	flag.Usage = func() {
		log.Printf("Usage: %s [options]", os.Args[0])
		log.Print("foos all your bars")
		flag.PrintDefaults()
	}
	var (
		foo = flag.String("foo", "foo", "alternate `foo`")
		bar = flag.String("bar", "bar", "alternate `bar`")
	)
	flag.Parse()
	//...
}
```

(I don't normally use `var ()` in functions but always do with flags. No idea why.)


## Validate your flags

If any of your flags can't be used together reasonably or their domain is a subset of their type's, check and inform the user. It's better they find out now then in a stack trace.

```
func main() {
	log.SetFlags(0)

	flag.Usage = func() {
		log.Printf("Usage: %s [options]", os.Args[0])
		log.Print("foos all your bars")
		flag.PrintDefaults()
	}
	var (
		foo = flag.String("foo", "foo", "alternate `foo`")
		bar = flag.String("bar", "bar", "alternate `bar`")
	)
	flag.Parse()
	if *foo == *bar {
		log.Fatal("-foo and -bar must be different")
	}
	//...
}
```


## Only log in `main()` and write helpers for logging

Again, this keeps things testable and clarifies the control flow of the program.

Any type or function can trivially be rewritten to handle fatal errors simply by returning them. Sometimes it needs to log non-fatal errors and it's impractical to evert the control flow to allow this from main. In such cases, pass in a `func(err)` or something appropriate and have it do nothing if it is nil.

Ignoring debugging statements, there are a few patterns to logging that can be encapsulated nicely.

These are only worthwhile if the particular case comes up a lot: otherwise just write the code.

The following snippets all assume you're wrapping your errors, but extra context is trivial to add.

A nice helper for fatal errors is

```
func fatalIf(err error) {
	if err != nil {
		log.Fatal(err)
	}
}
```

which reads `fatalIf(err)`.

Sometimes you want to report the error, and then, at the end, exit with a nonzero exit status if any errors were so reported.

An easy way to achieve this is with a closure in main:

```
code := 0
warnIf := func(err error) {
	if err != nil {
		code = 1
		log.Print(err)
	}
}
```

which reads `warnIf(err)` and the end of main would read `os.Exit(code)`. This could be made safer by writing
```
func makeWarnIf() (warnIf func(error), exit func()) {
	code := 0
	warnIf = func(err error) {
	if err != nil {
		code = 1
		log.Print(err)
	}
	exit = func() {
		os.Exit(code)
	}
	return warnIf, exit
}
```
but that's overkill.


## allocate resources in `main()` and delegate them to helpers with least privilege

Avoid opening and closing files outside of main. Pass them to types or functions that take io.Reader or io.Writer or the like. But be careful about Closing them. If this can't be avoided, at least try to separate the "open/close file" part from the "process file contents" part so that the actual processing can be tested.

Avoid creating goroutines outside of main. Pass <-chan T and chan<- T whenever possible.

"Context managers" are fine. By this I mean things such as
```
func Read(fname string, read func(io.Reader) error) error {
	f, err := os.Open(fname)
	if err != nil {
		return err
	}
	defer f.Close()
	return read(f)
}
```
or [x/sync/errgroup](https://godoc.org/golang.org/x/sync/errgroup).

`main()` should lay out the flow and structure of the program, passing the (non-trivial) particulars to subsystems. The trivial can remain as long as it doesn't obscure the purpose.

Reading main should tell you everything the program does, but not too much about how it does it. It's okay for main to be long, as long as it's simple.


## If a subsystem becomes too large or unwieldy, make a package

Sometimes there's too much code or you just want to think about it is a unit rather than an undifferentiated mass. It's fine to put this in a library. Prefer an internal/ library unless the API is sufficiently well-defined and general to share with the world. If you're unsure, put it in internal/ until you are. If you've followed the above, extracting it into a library shouldn't be too bad.



