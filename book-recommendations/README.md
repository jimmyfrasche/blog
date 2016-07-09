# Book Recommendations

I may add to this page on occasion. There is no organization other than: the last book I thought to add is at the bottom.

Links change, new editions appear, and search engines are really good, so I'm just leaving links out entirely.

I'll avoid common recommendations. Yes, read K&R, read SICP, but you knew that. This is not a list of books every programmer should read. This is a list of books I have not seen discussed much that you may be interested in reading.

Many of these books are out of date, the technology they discuss is outmoded or obsolete. That's unimportant. I am not including books that merely outline how to use some hot new technology. Though they can be inarguably useful at times. I will only include books that contain wisdom and knowledge applicable to more than just the particular version of the particular technology discussed.


# The Standard C Library
## by P. J. Plauger

Despite the unassuming title, this book is not simply a tour of the C stdlib. Along with the descriptions are rationales and detailed notes about how to implement and test each function. It contains many lessons in bootstrapping and API design that apply far outside the C standard library and C itself.


# Principles of Operating Systems
## by Brian L. Stuart

This book covers those things covered by most books on operating systems. Like some operating system books, it contains snippets of source code from real operating systems. What makes this book interesting is that it includes code from two different OSes—Linux and Inferno—and compares the approaches and the stark contrast design philosophies and implementation strategies can have on code.


# Project Oberon: The Design of an Operating System and Compiler
## by Niklaus Wirth and Jürg Gutknecht

If the title is confusing, that's because there's an Oberon language and an Oberon Operating System written in that language. This book details both. And a GUI editor. That makes the book sound long: it's not. That makes it sound as if they are not rendered in great detail: they are. Both Oberons are quite capable but ruthlessly minimal and worthy objects of study, if just to see how much can be built from so little.


# The Reader Over Your Shoulder
## by Robert Graves and Alan Hodge

This is a book about writing, not programming.

Unlike most books on writing—which are too often lists of "don't do this, I hate it"—The Reader Over Your Shoulder derives its rules from the first principles of clarity and succinctness. It contains a large appendix that's essentially a code review of famous writers. The authors take the writing samples, detail stylistic violations, and show how the application of the book's rules improve the author's intent. If there were ever a book on writing for a programmer, this would be it.

(N.B. This was later re-released under the generic title, The Use and Abuse of the English Language—those editions are unchanged but often much cheaper).


# The Elements of Programming Style
## by Brian W. Kernighan and P. J. Plauger

As the title suggests this is meant to be the programmer's equivalent to Strunk and White's Elements of Style, though it takes an approach closer to Graves and Hodges's The Reader Over Your Shoulder. Many of the code snippets examined were from extant textbooks of the day. They are rewritten until clear, often uncovering bugs along the way.

The language covered in this book is PL/I and it was written when structured programming was still the hot new thing. Nevertheless, the lessons of this book are timeless and apply equally as well today.


# Foundations of Databases
## by Serge Abiteboul, Richard Hull, and Victor Vianu

AKA: The Alice book.

This is THE book on database theory. It is dense and mathematical, but, if you can solider through, it reveals the hidden beauties of the relational database that are shrouded by the superficial suck of SQL syntax. It's primarily for implementers of databases, but it has value for any programmer who wants to know the nature and limits of the database.


# Software Tools
## by Brian W. Kernighan and P. J. Plauger

There are two version of this book, one written in Pascal and one written in Fortran. I have only ever read the version written in Fortran and that is the one I will discuss, though I'd like to read the Pascal version one day.

Software Tools is a surreptitious manifesto for the unix philosophy of small tools. The book details primordial implementations of grep, ed, roff, and m4, among others. The Fortran version, at least, includes a Fortran transpiler from a dialect, Ratfor (RATional FORtran), that adds structured programming constructs and adegree of platform independence to the many older but, at the time, predominant Fortrans that still made due with jumps and line numbers. While this is the last chapter, all the code in the book is written in Ratfor.


# Code Reading: The Open Source Perspective
## by Diomidis Spinellis

Code Reading is about how to read code: how to dive into an alien codebase and find what you need to know to achieve your goals. All of the example code is from open source projects, largely NetBSD, so there's no didactic sleight of hand involved, just good practical advice. Reading code is one of the most important things we do and even if you've been doing it for yours taking the time to pick up some new tricks is never a waste of time.
