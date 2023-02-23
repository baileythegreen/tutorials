
# Untangling a Gordian knot of bash strings
#### * In bash much of what I'll talk about falls under 'variable substitution', but 'string manipulation' is a more precise, if less accurate, name in this instance.
#### **It's fun. I promise.

### by Bailey Harrington

### Day before the Vernal Equinox, 2020 — Day 4 of coronavirus-induced isolation

Has this ever happened to you?: You're scrolling through a stackoverflow thread, minding your own business, looking for a solution for some command line operation you want to do and—**BAM!!!**—


```bash
${f%/*}/${f%%/*}_${f##*/}
```

    bash: /_: No such file or directory




you're caught completely unawares by a salvo of unprovoked punctuation to the face?

I get it. I've been there. So, here's what's actually going on there.

### Bash variable assignment

We're going to need a variable to work with. Remember, bash assignments do not allow spaces on either side of the assignment operator (the `=` sign).


```bash
var=summary
```

Perfect. Now we can reference our variable using bash's `$` syntax.


```bash
echo $var
```

    summary


For the rest of this tutorial, you'll notice I will use curly braces when I reference this variable. For more complex usage, this is required. I can get away without them in the example above because `$file` is separated from the following text (there isn't actually any there, but still) by whitespace, or an escaped character. This would have been just as valid (and helps with consistency):


```bash
echo ${var}
```

    summary


In the next three cells, you can see where the `{}` can become critical — for connecting the variable value with other text. In this example I will be trying to append `_file` to the end of the variable output.


```bash
echo $var_file
```

    



```bash
echo $var\_file
```

    summary_file



```bash
echo ${var}_file
```

    summary_file


The first example doesn't work because the variable is not separated from the following text. You'll see that in the second example I have succeeded in my aim by escaping the underscore, but I prefer the third option for consistency.

Now that you hopefully understand that `$var` is *sometimes*, but most of the time not, equivalent to `${var}`, let's move on to the actual interesting stuff.

Now, what if you only want some of the string?

### Isolating substrings

This is something I commonly encounter when working with file names. Manipulating file name strings for batch moving/renaming/copying/et cetera is my most common reason for doing string manipulation in bash. There are two ways you can do this: with string matching or by index.

For the following examples I'm going to make some new variables. You may notice that my examples will now all be bone-based. I work with skeletal data. \*in Jedi mind trick voice\* This is normal.


```bash
file=sumstats.tsv
path=bone/gwases/ukbb
```

#### Substring matching

First, let's remove the .tsv extension from `file`. This is useful for renaming it later on, or saving output to files with the same base name, but a different extension.

To remove a portion of the string *from the end*:


```bash
echo ${file%.tsv}
```

    sumstats


This works just as well with wildcards:


```bash
echo ${file%.*}
```

    sumstats


The general representation for this is: `${var\%pattern}`. What is actually happening is bash searchs within the value of `var` for a specified `pattern`. The `\%` operator tells it to begin its search from the end and remove the **shortest** matching substring.

If we want to only obtain the extension from `file`'s value, this is done very similarly  (with or without the use of wildcards), using the form `${var#pattern}`, which begins its search at the beginning of the string and removes the **shortest** matching substring:


```bash
echo ${file#sumstats.}
echo ${file#*.}
echo .${file#*.}
```

    tsv
    tsv
    .tsv


NB: If you want to use this for file validation, you'll have to think about whether you need the . as this will entail altering your expression slightly. Here, I've explicitly added the . to the last line.

If you have a longer string and you want to remove a larger chunk of it, you may be able to use the slightly more extreme versions of these commands, `${var\%\%pattern}` and `${var##pattern}`.

These will use the same start locations as their counterparts, but they remove the **longest** matching substring. For this use case, wildcards are more helpful.


```bash
echo $path
echo ${path%/*}
echo ${path/%%/*}
echo ${path#*/}
echo ${path##*/}
```

    bone/gwases/ukbb
    bone
    ukbb


These commands are extremely useful when dealing with file paths.

The bad news is that there is not a straightforward way to get `gwases` out of `path` in a single step using a hybrid of these two. You can, however, use indexing.

#### Indexing

The general form here is: `${var:start}` or `${var:start:length}`, where `start` is the starting index (this is zero-indexed), and `length` is the number of characters long the resulting substring should be. If the `length` parameter is omitted, the result will be the substring from `start` until the end of the string.


```bash
echo $path
echo ${path:0}
echo ${path:0:7}
echo ${path:8}
echo ${path:8:6}
```

    bone/gwases/ukbb
    bone/gwases/ukbb
    bone/gw
    ses/ukbb
    ses/uk


By getting creative, there are many things that can be accomplished just with these substring isolation techniques. However, somethings have their own syntax for accomplishing them more efficiently, such as replacing parts of strings or changing the capitalisation of some, or all, of a string.

### Subbing out substrings

This is basically like 'find and replace', but for the command line. The syntax is very similar to what we've been seeing and takes the general form of `${var/pattern/replace}`:


```bash
echo $file
echo ${file/tsv/txt}
echo ${file/stats/mary}
```

    sumstats.tsv
    sumstats.txt
    summary.tsv


A (natural (if you're used to programming stuff)) exension of this is that the same syntax can, with a slight truncation, be used to *delete* a substring, as well.


```bash
echo $file
echo ${file/.tsv}
echo ${file/sum}
```

    sumstats.tsv
    sumstats
    stats.tsv


Now, a nuance of what we've just done, is that *only the first match* of the pattern is being replaced or deleted. We can verify this:


```bash
dance=jiggety-jig
dance=jiggety-jig
echo $dance
echo ${dance/jig/dog}
echo ${dance/jig}
```

    jiggety-jig
    doggety-jig
    gety-jig


But what if you need to replace *every* occurrence of a substring? Use the form `${var//pattern/replace}`:


```bash
echo $dance
echo ${dance//jig/dog}
echo ${dance//jig}
```

    jiggety-jig
    doggety-dog
    gety-


We can take it even one step further and choose to replace things **iff** (if, and only if) they occur at the very beginning, or the very end using `${var/#pattern/replace}` and `${var/%pattern/replace}`, respectively:


```bash
echo $dance
echo ${dance/#jig/dog}
echo ${dance/%jig/dog}
```

    jiggety-jig
    doggety-jig
    jiggety-dog

