# CS 246 Linux Notes


## Kill signal
`^c`


## Close
`^d`


## Output Redirection
`>` (Default output in stdout)


## Input Redirection
`<` (This replaces the input that would come from the keyboard)
  - `cat < somefile.txt` OS opens up `somefile.txt` and feeds it to `cat` using `stdin`
  - `cat somefile.txt` cat opens up `somefile.txt`
#### Notes
  - `cat < *.txt` throws an error because the shell will only open ONE file.
  - We can do both `<` and `>` at the same time.


## Globbing Patterns
  - `*`
    - `cat *.txt` - All non hidden text files
    - `cat a*.txt` - All text files starting with a


## Streams
1. `stdin` - Standard Input (Redirected by `<`)
2. `stdout` - Standart Output (Redirected by `>`)
3. `stderr` - Standard Error - NON BUFFERED - (Redirected by `2>`)


## Pipes
Allow us to use output from one program as an input to another
Example: How many words are there in the first 20 lines of all my .txt files
`$ cat *.txt | head -20 | wc -w`
