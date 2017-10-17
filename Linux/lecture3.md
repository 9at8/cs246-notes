# CS 246, Lecture 3


## Shell Scripts
Files containing sequences of shell commands, executed as programs. 

```bash
#!/bin/bash
date
whoami
pwd
```

### Shebang
`#!/bin/bash` is called the shebang line

`#!` indicates to the OS that the file should be executed as a script, using the interpreter located at the procesing path.

Also remember to give the file execute permissions

Execute!

```bash
$ chmod u+x filename
$ ./filename
```

### Variables
```bash
x=1           # NO spaces!
echo ${x}     # Prints x; Wrap your variables in curly's
```
  - All variables are strings
  - `x` is actually `"1"`

#### Global Variables
  1. `PATH` - list of directories; when you type a cmd, the shell searches through the directories in order, for a program with that name.
  2. `$1`, `$2`, ... are the command line args
  3. `$#` holds the number of args passed to the script

### Remarks
  - Redirecting output to `/dev/null` to supress output
  - Every program returns a status code
    - Returns 0 if the word was found
    - Returns 1 if the word wasn't found
    - 2 or greater for errors
  - `$?` is a var that holds the status code (return code) of the most recently finished program

### Correct usage
```bash
#!/bin/bash

usage() {
  echo "usage: ${0} password" 1>&2
  # '1>&2' redirects stdout to stderr
  exit 1
}

if [ $# -ne 1 ]; then
  usage
fi
```

### General format of `if`
```bash
if [ cond ]; then
  # some stuff
  # more stuff
elif [ cond ]; then
  # some stuff
  # more stuff
else
  # some other stuff
fi
```

### General format of a `while` loop
```bash
#!/bin/bash
x=1
while [ $x -le $1 ]; do
  echo ${x}
  x=$((${x}+1))
done
```

### General format of a `for` loop
```bash
#!/bin/bash
for name in *.cpp; do
  mv ${name} ${name%cpp}cc
  # % is like modulus and returns the string except the largest substring from the back
done
```

The for...in... syntax sets the variable to each word in the given list. The syntax `${name%cpp}` produces the value of the name without the trailing cpp.

#### How many times does word `$1` occurs in file `$2`?

#### Payday is on the last friday of each month, compute this month's payday
```bash
#!/bin/bash
answer() {
  if [ $1 -eq $1 ]; then
    echo "This month the 31st"
  else
    echo "This month the ${1}th"
  fi
}

answer $(cal | awk '{print $6}' | egrep "[0-9]" | tail -1)
```

##### Generalize to any month
```bash
answer() {
  if [ $2 ]; then
    preamble=$2
  else
    preamble="This month"
  fi

if [ $1 -eq 31 ]; then
  echo "${preamble} the 31st"
else
  echo "${preamble} the ${1}th"
}

answer $(cal $1 $2 | awk '{print $6}' | egrep "[0-9]" | tail -1) $1
```
