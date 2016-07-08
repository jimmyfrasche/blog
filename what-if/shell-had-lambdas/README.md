# What if the shell had lambdas?

By shell I mean the collection of programs such as sh, bash, zsh, rc, and so on.

By lambda I mean an anonymous command, not necessarily an anonymous shell function.

I won't go into great detail about **how** this would/could work. Let's just go with magic for now and explore the fallout.

For those unfamiliar with shell scripting, you can define functions in a shell like so: (arbitrarily choosing bash, as it's very common)
```
	my-function() {
		sed s/hello/world/ | tr a-z A-Z
	}
```

This can be used in pipelines like
```
	echo hello | my-function
```
which outputs WORLD.

But some commands accept other commands as arguments, albeit not many, as it's currently awkward.

Let's just say we have a program called pipe that executes a command using its stdin and stdout so
```
	a | pipe b
```
would be essentially no different than not calling pipe:
```
	a | b
```
Not especially useful, as a program, but useful to illustrate the problem without introducing more conceptual overhead.

If we try
```
	echo hello | pipe my-function
```
we get back the error
```
	pipe: my-function: No such file or directory
```
—or, worse, something unrelated will happen because you have a poorly named program, `my-function`, somewhere in your `$PATH`.

This happens because pipe execs its argument so the kernel looks in the path for an executable file named my-function. But the shell's functions and pipelines exist only in the process of the shell.

If we took the contents of `my-function` and wrote it into its own file in `$PATH` and `chmod +x`'d it then we could use it with pipe—it's a command now. But we're just littering. There's no reason for it to be its own command other than technological restriction.

We could pull some shenanigans by changing `$PATH` in the outer shell script to conditionally expose the inner script, but that's not very satisfying and could best be described as clunky.

Neither of those help much when we're at the command line rather than writing a shell script.

If we need to pass something complex to pipe, we need to write the shell script first and delete it when we're done.

The simplest solution is to use the `-c` flag on the shell itself, like
```
	echo hello |
		pipe bash -c "sed s/hello/world/ |
		tr a-z A-Z"
```
but this introduce a new level of quoting so writing anything much longer than the above would be treacherous, and you're probably better off writing a temporary shell script.

What if there were an operator for automatically, temporarily, and safely pulling a bit of shell script out into an external program?

For notation let's use
```
	λ(cmd)
```
which is probably a bad choice for an actual syntax but serves the purpose of illustration.

We define it so that
```
	echo hello | pipe λ(my-function)
```
or even
```
	echo hello | pipe λ(sed s/hello/world/ | tr a-z A-Z)
```
are the same as passing a named command that performs the function denoted in the parentheses.

I'm assuming that all substitution, including command line arguments, happen each time the anonymous command is invoked. There might be some thorny issues there, even though I can't think of any.

(Silently promoting all shell functions to anonymous commands might work as well but let's go with the explicit syntax for clarity).

Some shells already do similar things with file descriptors. It's called process substitution and
```
	<(a | b | c)
```
creates an anonymous file like
```
	/dev/fd/63
```
allowing you to pass the output of pipeline to a command that expects a named file as an argument.

In this hypothetical universe where we can easily create anonymous commands in the shell, what could we do? If other programs are (re-)built to take advantage of this, perhaps quite a lot.

Let's look at a more realistic example. Say we have a file containing something like
```
	stuff	STUFF	stuff
	etc		Etc.	etc.
	...
```
and we want to sort it by the second column, ignoring case, and characters outside of [a-z]. We could use a Schwartzian transform like so
```
	<file awk '{
			k = tolower($2);
			gsub(/[^a-z]/, "", k);
			print k ":" $0 }' |
		sort -k1 |
		cut -d':' -f2- |
		sponge file
```
but this requires a great deal of forethought and effort, and it'd break in half if `$2` ever contained a ':' character. This is certainly a reasonable point to think, "Eh, I'll just do it in a different language."

But let's say sort took a command to generate the sort key to use for a given line, say, `--by` and then the above reduces to
```
	sort --by λ(awk '{
		k = tolower($2);
		gsub(/[^a-z]/, "", k);
		print k }') file
```
or, now that we only need to worry about extracting the key and not retaining its context in the line, the simpler
```
	sort --by λ(cut -f2 |
		tr A-Z a-z |
		tr -dc a-z) file
```

Existing programs that take commands as arguments would benefit, programs that do not could be enhanced now that it is convenient, but what about entirely new programs that didn't really make sense before?

A big one would be reduce (foldl) which could take a seed, a command, and stdin.

Given a file containing one number per line, we could compute the sum as
```
	<numbers awk '
		BEGIN { acc = 0 }
		{ acc += $1 }
		END { print acc}'
```
or with reduce
```
	<numbers reduce 0 λ(expr $1 + $2)
```
which as a bonus is more robust in the face of errors.

The other functional programming primitives are fairly well covered, as is, though there could perhaps be a number of command combinators whose only use is with these lambdas. I leave that as an exercise to the reader.

Another area would be tools for working with structured data that doesn't necessarily play well with the more traditional line-oriented toolkit.

If you want grep but for a particular column in a CSV you need to write or find something of the sort. If we had a program called `csvf` that takes a column and a command and shows the whole row if the command executed on that column returns true, then you could do
```
	csvf 3 λ(grep -qv banana)
```
While there's nothing stopping anyone from writing such a program now it wouldn't be very useful unless you had a `grep-not-banana` shell script lying around.

I know I said "magic" but how would something like this work? One strategy, glossing over a ton of details, is for the shell to mount a synthetic file system and creates executable files in it that, upon execution, call back into the shell to run the appropriate code. As with process substitution, it would allow the shell to rewrite
```
	cmd λ(a | b | c)
```
to
```
	cmd /some/magic/shell/fs/cmd-f54gf5dh645df64
```
and, whenever `cmd` execs that mouthful of a path, it's the same as running as a shell script containing
```
	a | b | c
```

There are probably better and more sensible ways to implement it—that's just the first thing I thought of that didn't involve kernel changes.

I can see why something like this wasn't included in shells, especially historically. It's slow and tricky to set up, except on something like Plan 9. Maybe it would even be a categorically bad idea.

But there are definitely times when I've been on the command line and thought, "If I only I could . . ."
