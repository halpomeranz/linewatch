# Linewatch

In Phil Hagen's excellent SANS FOR572 class, he discusses the problem of visually scanning through large data sets (netflow data, logs, etc). One low-technology trick is what Phil calls a _"visual grep"_:  scroll through the data rapidly and pause when you see an area of irregularity. In other words, you're looking for sections of the data which are different from the data around them.

Phil was bemoaning the fact that there was no tool to automatically perform this task. At first I though you could use the Linux "watch" utility for this. But "watch" only looks for diffs between the output of successive runs of a single command. It doesn't work on a file of data.

So I wrote "linewatch" as a tool to tell you when a given line of data is more than X% different from the line that comes before (the threshold is settable by the user). In its simplest form, you specify the threshold and your input file:

```
$ linewatch -t 20% testdata 
1:	      0123456789012345678901234567890123456789012345678901234567890123456789
7:	      0123456789012345678901234567890123456789
9:	      012345678901234567890123456789
11:	      01234567890123456789
13:	      0123456789
14 lines processed
```

Here we're asking for linewatch to show us any lines that differ by 20% or more from the previous line (you can say "-t 20%", "-t 20", or "-t .2"). The output is the line number and the content of the line. You will always see line #1 because it's 100% different from the "no data" that came before. The output includes the total lines in the file so you can see if there's a long gap between the last "different" line and the end of file.

Some data, such as Unix logs or netflow data, start with a variable timestamp value, which you may wish to ignore when doing your comparisons. Or you might want to only compare certain portions of each line. So linewatch has the -o, -c, -f options to "slice and dice" each input line before processing.

- -o is a simple offset. All characters prior to the offset are simply
ignored. So, for example, "-o 15" would ignore the normal Unix time/date
stamp that's in typical Unix log files.
- -c gives you a more flexible way of selecting ranges of characters
from your input. The syntax is Perl-like: "-c 20..40,60..80".
Character position is numbered starting from zero, so "0.19" is
the first 20 characters of the line.
- -f allows you to select fields using a delimiter (-d). The delimiter
can be a Perl regular expression. The default delimiter is whitespace
(in Perl regex language, "\s+"). Here's a trivial example which looks
for the login shell field changing in a Unix password file:

```
$ linewatch -f6 -d: -t0 /etc/passwd
11:	      nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
12:	      root:*:0:0:System Administrator:/var/root:/bin/sh
13:	      daemon:*:1:1:System Services:/var/root:/usr/bin/false
14:	      _uucp:*:4:4:Unix to Unix Copy Protocol:/var/spool/uucp:/usr/sbin/uucico
15:	      _taskgated:*:13:13:Task Gate Daemon:/var/empty:/usr/bin/false
86 lines processed
```

We use ":" as the delimiter ("-d:"). Note that we're comparing on  field number 6 ("-f6"), because fields are also numbered from zero. Our tolerance for change is zero ("-t0"), so any change at all will trigger output.
