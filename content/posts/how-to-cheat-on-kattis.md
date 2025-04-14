+++
date = '2025-04-13T21:00:00-00:00'
title = 'How to cheat on Kattis (code golf edition)'
tags = ['golf', 'polyglot']
+++

[Kattis](https://open.kattis.com/) is one of those popular competitive programming
sites, where people go to practice ICPC-style coding problems. I don't use it
personally, but a while ago I found out they have a shortest code leaderboard.
The easiest problem on the site is called [Hello World!](https://open.kattis.com/problems/hello),
where the task is simply to print `Hello World!` to STDOUT.

![Kattis problem statement](/kattis-problem-statement.png)

It seems straightforward at first, but if you check the [leaderboards](https://open.kattis.com/problems/hello/statistics), the shortest submission to the problem is an astonishing
**5 characters** long (in Bash)! Common sense tells us that
`Hello World!` costs 12 characters alone, so this score should not be possible
from an entropy perspective...

I did some digging myself, and after re-discovering the solution, it is both
disappointing and awesome at the same time (but mostly awesome).

### Exploring the submitter

For the most part, Kattis' submission system is similar to other online judges.
There is a dropdown menu to choose the language of your choice, a built-in code
editor, and a big green submit button:

![Kattis submission box](/kattis-submission-box.png)

What is not so standard among other judges is the editable *Entry point* field,
which allows one to rename the file of the program being executed. The
reason this exists is that we are actually allowed to submit multiple files
through the *+New* button in the top right corner. This is probably to allow
submissions to import other files. Usually CP'ers just code everything in one
file anyway, but it's a cool feature, I guess. ¯\\\_(ツ)\_/¯

From testing, it appears that the score is then calculated as the *total* size
of all submitted files -- so we can't cheat by just placing all the logic in
a non-entrypoint file.

However, this entrypoint feature leads to an evil idea: what if we put our code
as the *filename*, and execute the filename in the actual program?

### Filename smuggling

So here's the setup. We set our entrypoint to `echo Hello World! #.sh` and our
code as `$0`, which expands to the filename and executes it. The judge requires
the filename to end in `.sh` for Bash, hence the awkward comment.

![Kattis filename smuggling example](/kattis-filename-smuggling.png)

If this works, we would technically have a valid 2-byte *Hello World!* program.
But if we try to submit it, Kattis complains:

> *Error: Filename containing non-valid characters.
> Filename must begin and end with alphanumeric characters and may only contain
> alphanumeric characters and the special characters (.), dash (-), and
> underscore (_)*

Also, submitting a filename that is too large will result in another error that
says the filename must be between 2-64 characters long. For the regex enthusiasts,
the filename must match:

```re
/^[a-zA-Z0-9](?:[a-zA-Z0-9_.-]{,59}[a-zA-Z0-9])?\.sh$/
```

The task now reduces to finding a restricted Bash/filename polyglot matching the
above regular expression. However, this proves problematic for Bash because the
filename will inevitably get parsed as a single command that ends in `.sh`.

### Switching languages

Well, let's forget about Bash for now! This raises the question: what other
language has the esoteric property of being expressible with very few symbolic
characters?

Why **Perl**, of course! It is [well known](https://www.mcmillen.dev/sigbovik/2019.pdf)
that nearly anything can be a Perl program, and this regex is no exception.

Would you believe this is a valid Perl program that prints `Hello World!`:

```perl
eval-prin.t.v34.Hello.v32.World.v33.34.v59.pl
```

It is mostly abusing Perl's "v-strings", which converts ASCII characters to
characters (even though its intended usage is specifying version numbers). So
`v33` by itself produces ASCII 33 (a.k.a. `!`) and the whole thing `eval`s
a large concatenated string (since `.` is Perl's concatenation operator) that
equals

```perl
-print"Hello World!";pl
```

The `-` sign is just to avoid a space after the `eval`, but does nothing
because Perl is chill like that.

Using this program as the entrypoint leads to a 6-byte solution in Perl with
`eval$0`, very similar to the invalid Bash solution we saw earlier.

To understand the timeline, I submitted this Perl solution to Kattis on
`01/30/2024`, and snatched the top score (before the Bash solutions existed).

Just one day later, "Chenzhang Hu" stole the top spot with a 5-byte solution
in Bash. My score only lasted one day, how sad!

### Reaching 5 bytes

Fast forward one year later, I finally decided to have a crack at the Bash
solution. It turns out there is one major trick in Bash that immediately widens
the playing field -- **globbing**.

In the context of a filesystem, globbing is used to obtain a list of files
that match a particular pattern, e.g. `*.txt` to get all files ending in `.txt`.

Recall that the Kattis submitter allows us to submit any number of files to
the judge, whether we use them or not. These files are all placed in the same
directory as the entrypoint file.

This prompts the following idea: say we wish to execute a command like `cowsay`.
Instead of using up all 6 characters, we can:

- Submit an empty file called `cowsay` along with the entrypoint.
- In the *code*, replace `cowsay` with `c*`, which expands to `cowsay` since that is likely
the only file that begins with `c` in the working directory.

This type of glob injection lets us write virtually any command in just 2 characters!
In addition, if we supply multiple files that begin with the same letter, let's
say `echo` and `eggsandham`, then executing `e*` in the entrypoint should
expand to `echo eggsandham`, which prints `eggsandham`.

Putting this into practice, we can outsource to Perl and copy over our previous
solution to get this 7-byte solution: `p* -e$0`.

But we can do better! Instead of using the `-e` flag to specify a program
string, we can pipe the input of our program into `perl`. This may not
seem intuitively shorter, but it can actually be done in 5 bytes:

```perl
e*|p*
```

To force the correct glob expansion, we create the following empty files:

```
echo
eval-prin.t.v34.Hello.v32.World.v33.34.v59.pl
perl
```

With this configuration, it becomes clear that `e*|p*` should expand to

```perl
echo eval-prin.t.v34.Hello.v32.World.v33.34.v59.pl|perl
```

And voilà! Submitting those three dummy files and an entrypoint file containing
just `e*|p*` lets us reach the final score of 5 characters.

### Can we do better?

That is a natural question. I spent about half a day trying to shorten it, but
could not manage it. Despite this, I would not be surprised if it were possible.
There are plenty of binaries in `/usr/bin` which could be useful targets, and
a sufficiently fancy mix of globs and Bash trickery might break past the 5-byte
barrier.

I am also not sure if the other two top solutions use the same method. It is
possible that they are different, in which case it may be possible to combine them.

If you are reading this and are looking for a challenge, I encourage you to try
to find a shorter solution, maybe even in some other language!

### Other competitive programming sites

Besides Kattis, there are some other CP sites with a shortest code leaderboard.
What I've noticed from most of them, however, is that they usually have *some*
flaw.

For example, another popular one called [CSES](https://cses.fi/) allows
any problem to be solved in **16 bytes** in Ruby, because *whitespace* isn't
counted. I won't spoil the solution here, but it is also a fun exercise and
uses some pretty cursed Ruby techniques.

There is also [CodinGame](https://www.codingame.com/start/) which has a dedicated
Code Golf section, but I find that half of the challenges have weak test cases,
or the optimal solution involves outsourcing to Bash.

On the topic of code golf, I recommend checking out [code.golf](https://code.golf/).
It has a great competitive atmosphere for getting serious about golfing.
