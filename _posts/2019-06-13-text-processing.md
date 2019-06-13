---
layout: post
title: "Nine Months Later: System Software. Part I, Text Processing"
date: 2019-06-13
---

A brief look at what I learned from the System Software class this term,
starting with Unix text processing utilities.

Turns out it's been nine months since I've last posted here. I guess I have yet to
reach the point where I have enough confidence in my skills and knowledge to share it.
This note doesn't break the trend; I'm writing this only to refresh my memory before the exam.

With the intro out of the way...

* TOC
{:toc}

# grep: Search

`grep` is your friend for line-based text search. Something to keep in mind:
out of the box, most of regex-specfic metacharacters require backslashes: `+`, `?`, `|`;
`(` and `)` for capturing groups; `{` and `}` for specifying repetition count.

Slightly counterintuitively, `.` and `*` work, as do character classes (`[...]`).
I ain't got time to memorize the difference between basic and extended regular expressions, so while
doing the labs I came up with this simple rule:

> Most of the time I'm looking for a literal match, and `grep "expression"` just works.

> When I need to turn to regular expressions, I switch to `grep -E "expression"`.
> It beats trying to remember what needs escaping and what doesn't.

# sed: Filter and Replace

For simple text substitution and filtering, `sed` is the tool.
`sed -i 's/one thing/another thing/g' file` to edit a file in place is a classic, and
something I actually use from time to time, despite spending relatively little time inside the shell.

`s` is just one of the commands, though. `d` is another useful one: it removes lines from the output
based on a pattern — which is specified the same way for all commands, `/pattern/ [commands]`.

As an example, here's how to remove the lines that contain `delet this`:

```sh
sed '/delet this/ d'
```

To reverse the effect, put a bang before the command:

```sh
sed '/delet this/ !d' # only print lines containing 'delet this'
```

`p` command is the opposite, and is usually used with `sed -n`, which doesn't automatically output anything.
`sed -n '/delet this/ p` and `sed '/delet this/ !d` are, to my limited knowledge, equivalent.

The pattern can be replaced with a line number (starting from `1`).

Operations can be applied to ranges: `1,3d` deletes the first three lines (same with patterns).

There's also a built-in global variable, the *hold space*:
* `h`: hold space = current line
* `H`: hold space += current line (separated by a newline)
* `g`: replace the current line with the hold space
* `G`: append the hold space to the current line (separated by a newline)

I'm not sure what the practical usage of that is but it looks kinda cool.

The catch with `-E` applies to `sed` just as to `grep`.

# awk: Records and Complex Processing

I never understood the appeal of `awk`, but I think that can be attributed to the fact I didn't need to do
any elaborate text processing. As far as I can judge, for basic replace/filter tasks `sed` one-liners are always shorter and more concise.

Files with data separated into fields (CSV) are a major exception. Printing the 2nd and the 4th fields of records with the 3rd field matching
a pattern? I have no idea how to even think about it in terms `sed` operations. In `awk` it's just:

```sh
awk -F , '$3 ~ /pattern/ { print $2, $4 }'
```

And it looks pretty self-explanatory too.

Passing shell variables to `awk` scripts through escaping can get ugly fast, although it's possible:
```sh
awk 'BEGIN { print "'"$HOME"'" }'
```

Using the `-v` option is preferred:
```sh
awk -v shell="$SHELL" -v home="$HOME" 'BEGIN { print shell, home }'
```

Finally, `awk` is a surprisingly powerful language for more complex scripts:

```awk
BEGIN {
  # executed before processing the file
  FS = "," # same as awk -F ,
}
{
  # executed for each line
  total += $1 # tallying? easy!
  fields_per_line[NR]["count"] = NF # multi-dimensional associative arrays
}
END {
  # executed after processing the file
  print total
  # iteration!
  for (line in fields_per_line)
    print "Fields on line #" line ": " fields_per_line[line]["count"]
}
```

Example output:
```
> cat test
25,0,0
35,0
10,0,0,0

> awk -f script.awk test
70
Fields on line #1: 3
Fields on line #2: 2
Fields on line #3: 4
```

Modern versions of `GNU awk` even have a debugger!
```
> awk -D -f script.awk test
gawk> b script.awk:7 "NR == 3"
Breakpoint 1 set at file `script.awk', line 7
gawk> r
Starting program: 
Stopping in Rule ...
Breakpoint 1, main() at `script.awk':7
7         total += $1 # tallying? easy!
gawk> p $1
$1 = "10"
```

# sort: Sort Lines

```sh
sort -t [delimiter] -k [field #]
```

* `-n`: numeric sort
* `-r`: sort in descending order

# join: Merge Records

```sh
join -t [delimiter] -1 [file1 field #, starting from 1] -2 [file2 field #] file1 file2
```

Input files should be sorted by the field they're being merged on.

In this case one example is all you need:
```
> cat characters.csv
Shinji,M
Asuka,F
> cat roles.csv
Pilot,Asuka
Doormat,Shinji
> join -t , -1 1 -2 2 <(sort characters.csv) <(sort -t , -k 2 roles.csv):
Asuka,F,Pilot
Shinji,M,Doormat
```
