## DEFINITIONS

- Shell
	- textual interface to the computer
	- core language for the interacting with the computer
	- gives you the ability to combine programs in creative ways to automate nearly any task
- Terminal 
	- Visual interface to the shell/ the GUI window for the shell
- Bash 
	- Most commonly used shell 
	- Bourne Again SHell
- Argument
	- Things that follow the program name in the command
- White space
	- The shell parses the command by splitting it by whitespace, and then runs the program indicated by the first word, supplying each subsequent word as an argument that the program can access.
- `$PATH`
	- Execution of program - how does the shell know how to find programs like date or echo: it consults an environment variable called $PATH that lists which directories the shell should search for programs when it is given a command
- Permissions for users
	-  `r` (read), `w` (write), and `x` (execute)

## SHELL LANGUAGE

- Pipes (`|`) 
	- let you string together the output of one program with the input of another.
	- Run the left program, take its output and make that be the input of the program to the right 
- Redirects (`>` , `>>`, `<` )
	- `>file` lets you take the standard output of a program and write it to file instead of to your terminal.
	- `>>file` will append to file rather than overwrite it. 
	- `<file` tells the shell to read from file instead of from your keyboard as the standard input to a program. 
	- EXAMPLE: redired out and error in txt file
		- The shell has three standard streams: stdin (0), stdout (1), and stderr (2) 
		- `ls /nonexistent /tmp >out.txt 2> err.txt`
			- This will create two files out.txt and err.txt
		- `ls /nonexistent /temp >out.txt 2>&1`
			- This will create one file where both the output and the error 2 will be send to
- Conditionals:
	- `if command1; then command2; command3; fi` 
	- The most common command to use as `command1` is the `test` command, often abbreviated simply as `[` ... `]`
	- ....
- Loops:
	- WHOLE loop: `while command1; do command2; command3; done` 
	- FOR loop: `for varname in a b c d; do command; done`
		- executes `command` four times, each time with `$varname` set to one of `a, b, c, and d`. Instead of listing the items explicitly, you’ll often use “command substitution”, such as:  `for i in $(seq 1 10); do ..
- Exit status:
	- Exit statuses fall between 0 and 255
	- All of the Bash builtins return an exit status of zero if they succeed and a non-zero status on failure
	- exit status of the last command is available in the special parameter `$?` 
	- `&&` runs the next command only if the previous succeeded;
	- `||` runs it only if the previous failed. 
		- Example:  `[ -d /tmp/mydir ] || mkdir /tmp/mydir` 
			- - `[ -d /tmp/mydir ]` exits with 0 (success) if `/tmp/mydir` exists and is a directory, and with 1 (failure) otherwise. It prints nothing either way.
			- `||` runs `mkdir` only if the test failed, meaning only if the directory doesn't exist.
			- Note the spaces: after `[`, and before `]`.
- Conditional Expressions:
	- `-a file` - True if file exists.
	- `-b file` - True if file exists and is a block special file.
	-  `-c file` - True if file exists and is a character special file.
	- `-d file` - True if file exists and is a directory.
	- `-f file` - True if file exists and is a regular file.
	- ...
- Command substitution
	- `$(command)` - allows the output of a command to replace the command itself

## SHELL SCRIPTS

- Start of file: `#!/bin/bash`
	- first character is the “shebang”
	- When a file that starts with the magic incantation `#!/path` is executed, the shell will start the program at `/path`, and pass it the contents of the file as input. In the case of a shell script, this means passing the contents of the shell script to `/bin/bash`, but you can also write Python scripts with a shebang line `/usr/bin/python`
- Second line:  `set -euo pipefail`
	- `-e` makes it so that if any command fails, the script exits early;
	- `-u` makes it so that use of undefined variables crashes the script rather than just using an empty string;
	- `-o pipefail` makes it so that if programs in a `|` sequence fail, the shell script as a whole also exits early.
-  When writing a script to use the arguments that are passed to it in the script:
		- `$1`: first argument, `$2`: second argument, ... ; `$@` all the arguments together
		- Example: `bash check.sh out.txt` and in the `check.sh` script you have `if [ -f "$1" ]; then`
-  in shell scripting everything is designed around running programs and getting them to communicate with each other simply and efficiently
## PROGRAMS

- `echo` - like print" show a line of text or a variable's value in the terminal.
- `man` 
	- “Manual” - pass to it the name of another program and it explains how to use the program
	- Alternative to it -> short version of the help by passing `--help`  as an argument to the program
- `cd` 
	- “Change Directory” - This command is actually built into the shell, and isn’t a separate program
	- You pass it a path, and that path becomes your current working directory.
- `pwd` - produce the current working directory.
- `ls`  - list program
- `nano` - write a file
- `cat file` - prints the contents of `file`
- `sort file` - prints out the lines of `file` in sorted order
- `uniq file` - eliminates consecutive duplicate lines from `file`
- `head file` and `tail file` - print the first and last few lines of `file`
- `grep pattern file`
	- finds lines matching pattern in file
	- `pattern` is actually a regular expression which can express very complex patterns
	- can also specify a directory instead of a `file` (or leave it off for) and pass `-r` to recursively search all the files in a directory.
- `sed`
	- programmatic file editor.
	- has its own programming language for making automated edits to files
	- the most common use of it is: 
	- Example: `sed -i 's/pattern/replacement/g' file` 
		- This replaces all instances of `pattern` with `replacement` in `file`
		- The `-i` indicates that we want the substitutions to happen inline (as opposed to leaving file unmodified and printing the substituted contents). 
		- The `s/` is the way to express in the `sed` programming language that we want to do a substitution. 
		- The `/` separates the pattern from the replacement
		- The trailing `/g` indicates that we want to replace all occurrences on each line rather than just the first
- `find`
	- lets you find files (recursively) that match certain conditions.
	- Example: `find ~/Downloads -type f -name "*.zip" -mtime +30`
		- find ZIP files in the download directory that are older than 30 days
	- Example `find ~ -type f -size +100M -exec ls -lh {} \;`
		- Finds files larger than 100M in your home directory and lists them.
		- Note that `-exec` takes a command terminated with a stand-alone `;` (which we need to escape much like a space) where `{}` is replaced with each matching file path by `find`.
	- Example: `find . -name "*.py" -exec grep -l "TODO" {} \;`
		- Finds any `.py` files with TODO items in them.
- `awk`
	- Like `sed` has its own programming language
	- parsing files (reading a raw file and converting its content into a structured, machine-readable format)
	- Example: `awk '{print $2}' file
		- prints the second whitespace-separated column of every line of file
- `tocuh` -  change file timestamps or create an empty file if it doesn't exist.
- `cp source_file destination_file` - copy files and directories from one location to another
- `mv source_file destination_directory` - move or rename files and directories.

  
## GLOB 
Special pattern characters:
- `*` - Matches any string, including the null string.
- `?` - Matches any single character.
- `[...]` 
	- Matches any one of the characters enclosed between the brackets. This is known as a bracket expression and matches a single character.
	- A pair of characters separated by a hyphen denotes a range expression; any character that falls between those two characters, inclusive, using the current locale’s collating sequence and character set, matches.

## SHORTCUTS
- `Ctrl + L` - Clears the terminal screen (equivalent to typing `clear`, but faster).
- `Ctrl + C` - Kills or aborts whatever command is currently running.
- `Ctrl + A`: Jumps the cursor to the **beginning** of the line.
- `Ctrl + E`: Jumps the cursor to the **end** of the line.
- `Alt + B`: Moves the cursor **backward** by one full word.
- `Alt + F`: Moves the cursor **forward** by one full word.

## REMOTE MACHINES

It has become more and more common for programmers to work with remote servers in their everyday work. The most common tool for the job here is SSH (Secure Shell) which will help us connect to a remote server and provide the now familiar shell interface.

``` bash
ssh alice@server.mit.edu
```

Here we are trying to ssh as user `alice` in server `server.mit.edu`.

An often overlooked feature of `ssh` is the ability to run commands non-interactively. `ssh` correctly handles sending the stdin and receiving the stdout of the command, so we can combine it with other commands

``` bash
# here ls runs in the remote, and wc runs locally
ssh alice@server ls | wc -l

# here both ls and wc run in the server
ssh alice@server 'ls | wc -l'

```

For `ssh` to let us run commands in the remote server we need to prove that we are authorized to do so. We can do this via passwords or ssh keys. Key-based authentication utilizes public-key cryptography to prove to the server that the client owns the secret private key without revealing the key

To generate a pair you can run ssh-keygen.
```bash
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

...
