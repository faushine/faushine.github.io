---
layout: post
title: "02-Shell scripts"
date:   2019-06-06
author: Faushine
tags: 
- C++
---

# shell scripts

~/cs246/1195/lectures/shell/scripts

## what is shell scripts?

the file contains a sequence of commands, executed as it was a program

## How does the operating system know where to look for your program

it looks in the environment variable, PATH
eg, echo $PATH
can use 'env'

bin-binary
env-environment configuration

```bash
\#!/bin/bash
```

```bash
date
whoim
pwd
```

- comments start with a '#'
    1st line is called 'shebang' line; must start with #!

    followed by shell to use then options eg -x <-verbose mode

- file suffix is , by convention, .sh or .bash; but ,not required
  
- need to tell shell where the 'program' is 
    &rarr; current directory -> './'

    &rarr; make sure you add execute permission to file via : chmod u+x basic

- variable name are strings; so are the values
  
- no space on either side of equal sign; variable on left-hand side may not have a $ infront eg. x=1
  
- should surround variable name with {3 when used ie. value extracted eg:

    ```bash
    echo xy
    echo ${x}y
    ```

- variables can be global to the script
- use appropriate quotation marks 

    ```bash
    $Q <- name of the script
    $1 $2 .. <- command line argument
    $# <- number of command line argument

    $? - returen value of conmmand just executed
    $* - concentrate all command line arguments into a single string( by default, separated by a space)
    ```

### eg: good password
/dev/null: if redirect output here, it is discarded

```bash
if[...]; then
elif[...]; then
.
.
.
else
fi
```

## Function

```bash
[function] name ()<-nothing in there
{
    #body of function
    access call parameters cia $i, etc
}
called via: name list of parameters
N.B.: "exit" is a hard-kill i.e. terminates executtion
```

## Loops

```bash
#!/bin/bash
x=1
while [ ${x} -le 1 ]; do
    echo $x
    x=$((x+1)) -> how we do arithmetic
done

for variable in list; do
    #stuff
done
```

```bash
name%c => substring c
```

Paydays at UW far non-union staff is last Friday of the month

```bash
(1) cal
(2) | awk  '{print $6}'
(3) | egrep "[0-9]"
(4) | tail -1
```