# CS 246 Lecture 2


## More commands
  - `sort`
  - `uniq` - picks out unique values


## Subshell
```
$ echo "Today is $(date) and I am $(whoami)"
```
The shell executes the commands `date` and `whoami` in a subshell and replaces them in the string with their output.

Note: We will not see this behaviour if we use `'...'`(single quotes)


## Pattern matching in text files
### `egrep`: "Extended Global Regular Expression Pattern"
```
$ egrep <pattern> <file>
```
The patterns matched by egrep are called regular expressions and they are not the same as globbing patterns

Examples for regular expressions:
  - `cs246` - lowercase
  - `CS246` - uppercase
  - `(cs246|CS246)` - uppercase or lowercase
  - `(c|C)(s|S)246` - mixture
  - `[^...]` - matches any one character not in the square brackets
  - `[cC][sS] ?246` - matches the same as #3 with an optional space between `cs` and `246`. `?` denotes 0 or 1 of the preceding pattern
  - `(cs)*246` - matches `246`, `cs246`, `cscs246`, ...; The `*` denotes 0 or more of the preceding pattern
  - `+` - indicates 1 or more of the preceding pattern
  - `.` indicates one character, `.*` matches all strings, `.+` matches all non empty strings
  - `^` denotes starting of a line
  - `$` denotes ending of a line
  - Match lines only with `cs246` - `^cs246$`
  - `^.*cs.*246.*$` = `cs.*246`
  - Match lines with even length - `^(..)*$`
  - Match lines with one `a` - `^[^a]*a[^a]+$`
  - Match lines that start with `e` and consist of 5 characters - `^e....$` or `^e.{4}$`


## Permissions
```
ls -l
drwxr-xr-x 2 aditya users 45 Sep 12 13:22 Linux
[type][user-perms][group-perms][others] [user] [group] [size] [date-modified] [name]
```
  1. Type: `-` for a file; `d` for a directory
  2. Group: A user can belong to many groups

Note: The `user-permission` supercedes the `group-permission` in giving and ytaking away permissions.

|Bit|Meaning for files|Meaning for directories|
|---|---|---|
|r|File's content can be read|Directory's contents can be read|
|w|File's content can be written/modified|Directory's content can't be modified|
|x|File can be executed|Directory can be navigated|

Note: If a directory's `x` bit is not set (for you) there is no access to that directory at all regardless of other bits

For changing permissions - use `chmod`
```
$ chmod mode file
```

Args:
  - `u` - for you
  - `g` - group
  - `o` - other
  - `a` - all

Operators:
  - `+` - Adds permissions
  - `-` - Subtracts permissions
  - `=` - sets perms


