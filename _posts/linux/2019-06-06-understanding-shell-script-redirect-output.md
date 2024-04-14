---
layout: post
title:  "Understanding shell script redirect output"
description: 'Understand shell script redirect output'
slug: 'understanding-shell-script-redirect-output'
date:   2019-06-06 12:01:06 +0300
category: linux
tags: [linux, shell, logs, redirect, output]
icon: angle-right
---

The redirection capabilities built into Linux environment provides a robust set of tools used to make all sorts of 
tasks easier to accomplish. Knowing how they work and how they can help to manipulate the I/O (Input/Output) streams 
will greatly increase your productivity.

### Standard streams

Input and output in the Linux environment is distributed across three streams. These streams are:

- stdin (the keyboard)
- stdout (the screen)
- stderr (error messages output to the screen)

The streams are also numbered:

- stdin (0)
- stdout (1)
- stderr (2)

### Introduction to stream redirection

Redirection simply means capturing output from a file, command, program, script, or even code block within a script
and sending it to another place being it a file, a command or a script. 

#### Overwrite the destination's existing contents.

- `<` - standard input
- `>` - standard output
- `2>` - standard error

#### Append to the destination's existing contents.

- `>>` - standard output
- `<<` - standard input
- `2>>` - standard error

> Note! If you want to try the examples by yourself first create a `frameworks.txt` file with your favourite frameworks.

By default when we use `cat` command to look inside of a file, the output will be printed on the screen:

```
$ cat frameworks.txt
Symfony
Laravel
Zend
```

But sometimes is useful to redirect this output somewhere else, like a different file, `output.txt` in our short example:

```
$ cat frameworks.txt > output.txt

$ cat output.txt
Symfony
Laravel
Zend
```

As you can see when the first command is executed there is no output on the screen. That's because we redirected the 
standard output (stdout) to the `output.txt` file, so it is not printed on the screen anymore.

> Note that even that the `output.txt` file didn't existed, it was created prior to writing, exactly like you'll do 
it with `touch` command.

Another important stream called standard error (stderr), is the place to where programs can send their error 
messages. To easily see this, let's cat my personal `passwords.txt` file and redirect the output to the `output.txt`
file exactly like before:

```
$ cat passwords.txt > output.txt
cat: passwords.txt: No such file or directory
```

Even if we redirect the *stdout* to a file, we can see the error output on the screen, because we are redirecting 
just the standard output, not the stderr.

To accomplish the exact same thing for `stderr` as with the `stdout` we have to use the `2>` instead of simple `>`. 

```
$ cat passworkds.txt 2> output.txt
$ cat output.txt
cat: passworkds.txt: No such file or directory
```

### File descriptors

In order to understand why errors are redirected with `2>`, like in our previous example, we have to know about 
descriptors.

A file descriptor is nothing more than a positive integer that represents an open file. 
The Standard Input, Standard Output and Standard Error are the default file descriptors where a process can write output.

Shortly, we can identify those locations with `ids`, and it will always be 0 for stdin, 1 for stdout and 2 for stderr.

#### Making use of descriptors

Going back to our first example, when we redirected the output of the `cat frameworks.txt` to `output.txt`, we could 
rewrite the command with descriptors:

```
$ cat frameworks.txt 1> output.txt
```

The `1` is just the file descriptor for `stdout`. The syntax for redirecting is `[FILE_DESCRIPTOR]>`. Omitting the file 
descriptor from the command is just a nice shortcut to `1>`.

Now that everything is more clear, you now understand why earlier the `2>` worked to redirect the `stderr` to the file.

### 2>&1 meaning

By now you already know half of the command `2>&1`. We are trying to say redirect the `stderr` to `&1`. Finally, the 
`&1` is used to reference the value of the file descriptor 1 (stdout). 
That means when we're using `2>&1` we are basically redirecting the `stderr` to the same place as `stdout`.

```
$ cat frameworks.txt > output.txt 2>&1
$ cat output.txt
Symfony
Laravel
Zend
```

```
$ cat passwords.txt > output.txt 2>&1
$ cat output.txt
cat: passwords.txt: No such file or directory
```

### Conclusions

In Linux there are two streams where programs send output to: Standard output (stdout) and Standard Error (stderr). 
We can redirect these outputs to a different location, a file, a command or a script.

We can use file descriptors to identify stdin (0), stdout (1) and stderr (2) and `&[FILE_DESCRIPTOR]` to 
reference a file descriptor value.

Finally using `2>&1` will redirect stderr to exact same place stdout is being sent.
