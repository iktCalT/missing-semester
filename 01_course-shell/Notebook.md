# Lecture 1: Course Overview + Introduction to the Shell [![][MS_ICON_MIDDLE]](https://missing.csail.mit.edu/2026/course-shell/)


## Introduction to shell
Shell is the textual interface to the computer.  

Shell runs in the context of terminal.  

The most commonly used shell in Unix-like system is bash. But it's very old.  
zsh is a newer shell.

For windows, they are batch (older) and PowerShell (newer).  
PowerShell is good, but it uses different syntax. This course will focus entirely on bash.

## Why shell
- In the shell, you can automate tasks.  
- It can combine programs. (pipeline)  
- Knowing it helps you interact with open-source community as well as work in a company.

## Prompt
`USR@HST:DIR$` or `[USR@HST:DIR]$`  
`USR`: username  
`HST`: hostname of machine  
`DIR`: current directory  
`$`: you are not admin user  

eg. Prompt at my WSL home directory: `meow@MyPC:~$`

## Execute
Followed by prompt, you can type a program name to run it.  
eg. `meow@MyPC:~$ date`

We can also run a program with arguments (separated by white spaces).  
eg.   
  `echo Hello      World` (two args) -> 
  ```bash
  meow@MyPC:~$ echo Hello      World
  Hello World
  ```  
  `echo "Hello      World"` (one arg) -> 
  ```bash
  meow@MyPC:~$ echo "Hello      World"
  Hello      World
  ```  
  ### Escape characters
  `\` can convert a special character (like white spaces or quotes) to a plain (literal) character. (`\` can also escape itself: `\\\\` represents two plain backslashes)  

  `echo Hello\ \ \ \ \ \ World` (one arg: `Hello + 6 SPACES + World`) ->
  ```bash
  meow@MyPC:~$ echo Hello\ \ \ \ \ \ World
  Hello      World
  ```
  `echo \"Hello      World\"` (two args: `"Hello` and `World"`)  
  ```bash
  meow@MyPC:~$ echo \"Hello      World\"
  "Hello World"
  ```

## Manual 
`man` (means manual) is a useful program. By default, it uses a text reader (read-only) called `less`.  
eg. `man echo`, `man bash`, `man gcc`, `man python`, `man less`, `man nethack`  

Some projects have no manual pages. For example vscode (`man code` / `man vscode` -> No manual entry for code / vscode)  
Instead, they support `--help` or `-h`. `code --help`  

`nethack` have manual page but doesn't support `--help` (or `-h` for short).

Some even doesn't support `--help` (`asciiquarium`).  

## Relative path & absolute path
*Relative paths are any path that does not start with a slash.* (or tilde? I guess)  

`cd bin` is equivalent to `cd ./bin`  
`cd` is equivalent to `cd ~`, `~` is home directory, its absolute path is `/home/username`   
`.` refers to means current directory  
`..` refers to the parent directory  

You can combine them
```bash
meow@MyPC:/$ cd home/././meow/.././../bin # meow is my username
meow@MyPC:/bin$
```

## Useful commands 
- `cd`: change directory. Options—see `cd --help`  
- `clear`: clear the terminal screen (not `clean`—the one I've always mistaken for)  
- `ls [OPTION]... [FILE]...`: list (ls) all files and folders under `[FILE]` (file or directory). 
  If `[FILE]` is not provided, it will be current directory by default.  
  Options—see `ls --help` or `man ls`.  
- `cat`: print out the contents of a file.  
  eg. `meow@MyPC:~$ cat ~/.bashrc`  
- `sort`, `uniq`, `head`, `tail`: read files in specific behavior. [![][YT_ICON]](https://youtu.be/MSgoeuMqUmU?t=2012)  
  #### Useful combination:  
  `man python | head -n20`: print first 20 line of python's manual;  
  `man python | grep -A 5 "DESCRIPTION"`: Show the word "DESCRIPTION" and the 5 lines after it.  
- `grep`: a file searcher. Search for thing match particular pattern **in a file**.   
  eg. `grep PATH= ~/.bashrc`: find all lines with string "PATH=" in `.bashrc`.  
  `grep -r PATTERNS [FILE]`: `[FILE]` here is a folder. Find lines in all files under that folder ([FILE]) recursively.  
  `grep` is very powerful because it supports **REGULAR EXPRESSION (regex)**.
- `sed`: is used to modify files. [![][YT_ICON]](https://youtu.be/MSgoeuMqUmU?t=2264)
- `find`: used to find file.   
  ```bash
  meow@MyPC:~$ find ~/Downloads -type f -name "*.tar" # -type {f | d}: file or directory
  /home/meow/Downloads/datalab-handout.tar
  ```
  There are a lot of arguments can be passed to `find` (check `man find` or ask LLMs). A more complicated example (at [![][YT_ICON]](https://youtu.be/MSgoeuMqUmU?t=2751))  
  ```bash
  meow@MyPC:~$ find ~/Downloads -type f -size -10M -exec cp {} {} \;
  cp: '/home/meow/Downloads/datalab-handout.tar' and '/home/meow/Downloads/datalab-handout.tar' are the same file
  ```  
  \[Note\]: `find` is recursive by default. You can use option `-maxdepth` to control its searching depth.  
- `awk`: used to parse files. By default, it will split a file by white spaces and lines breaks.  
- `ssh`: run a command on a remote machine.  
- `chmod`: change file mode—read(r), write(w), execute(x), ...

## Bash language
- `|` (pipe character): Take the output of left program as the input of right program.  
- `>` and `<`: write to (overwrite - delete old data and write) and read from a file. `>>` append to the end of a file. For example: `date > thedate.txt`   
- Bash supports  
  - Conditional expression: 
    ```bash
    meow@MyPC:~$ date -u > thedate.txt
    meow@MyPC:~$ date -u >> thedate.txt
    meow@MyPC:~$ cat thedate.txt
    Wed Feb  4 02:41:59 UTC 2026
    Wed Feb  4 02:42:25 UTC 2026
    meow@MyPC:~$ if grep 2026 thedate.txt; then echo "it's 2026"; fi
    Wed Feb  4 02:41:59 UTC 2026
    Wed Feb  4 02:42:25 UTC 2026
    it's 2026
    ```
    If you don't know what a program exits with, search "exit status" block in its manual.  

  - While loop:  
    ```bash
    while CONDITION; do EXPRESSION_1; EXPRESSION_2; ...; done
    ```
  - For loop:  
    ```bash
    meow@MyPC:~$ for var in Hello , World \!; do echo "$var"; done
    Hello
    ,
    World
    !
    ```
- `"$(prog)"`: program; `"$var"`: variable
  ```bash
  for var in $(seq 1 2 10); do echo "$var"1; done # seq 1 2 10: sequence of 1 to 10, step length is 2
  # out: 11 31 51 71 91 
  ```
- `test` or `[]`:  
  ```bash
  meow@MyPC:~$ if [ "hello" = "world" ]; then echo "equal"; else echo "not equal"; fi
  not equal
  if test "hello" = "world"; then echo "equal"; else echo "not equal"; fi
  not equal
  ```
- Sleep:  
  eg. `sleep 10`
- You can store commands in a `*.sh` file.
  [![][YT_ICON]](https://youtu.be/MSgoeuMqUmU?t=4119)  
  If a line starts with `#!/FILE` (called **shebang** line), it means: give the contents of this program to that `FILE`.

  For example, if `lecture.sh` has a line `#!/bin/sh`, then `./lecture.sh` is equivalent to `/bin/sh lecture.sh`. So that bash (`/bin/sh`) can run the scripts in `lecture.sh`.   

  **To run `lecture.sh` you have you make it executable first.** Use `ls -l lecture.sh` to see your permission to that file. Use `chmod +x lecture.sh` to add execute permission to this file.  
  For safety reason, don't execute with `lecture.sh`, it will try to find and run it in `$PATH`, if there is another program with same name in your `$PATH`, you may get unexpected result. Instead, use `./lecture.sh` to specify that your program is in the current directory.  

  Moreover, it's not limited to bash. For example, if I have a bash file `python.sh` with a shebang line `#!/usr/bin/python3`, followed by python codes `import numpy as np ...`. Then when I type in `./python.sh` and press `Enter`, python will run the python codes in the python script (a bash file) `python.sh`.

  ```bash
  # Generate a test file
  meow@MyPC:~$ echo '#!/usr/bin/sh' > test.sh
  meow@MyPC:~$ echo 'echo "The name of this script is: $0"' >> test.sh
  meow@MyPC:~$ /bin/sh ./test.sh
  The name of this script is: ./test.sh
  # Check its contents
  meow@MyPC:~$ cat test.sh
  #!/usr/bin/sh
  echo "The name of this script is: $0"

  # /bin/sh test.sh and /bin/sh < test.sh have different behaviors. 
  # Although in most cases, they have the same behavior
  meow@MyPC:~$ /bin/sh < ./test.sh
  The name of this script is: /bin/sh
  meow@MyPC:~$ /bin/sh ./test.sh
  The name of this script is: ./test.sh

  # ./test.sh is equivalent to /bin/sh ./test.sh 
  # rather than /bin/sh < ./test.sh
  meow@MyPC:~$ chmod +x test.sh
  meow@MyPC:~$ ./test.sh
  The name of this script is: ./test.sh
  ```

## Tips
- `Tab`: autocomplete
- double press `Tab`: all possible options  
  ```bash
  meow@MyPC:/$ cd b # double press Tab
  bin/               boot/
  ```
- `^C` (`Ctrl` + `C`): can not only kill the program, but also clear the command line you are typing
  ```bash
  meow@MyPC:/$ cd hkaskhjakvbcxjz^C
  meow@MyPC:/$
  ```
- `which`: find program in `$PATH` environment variable  
  ```bash
  meow@MyPC:~$ which which
  /usr/bin/which
  ```
- `echo $PATH`: print the directories stored in the `$PATH` environment variable (separated by colons)  
  ```bash
  meow@MyPC:/$ echo $PATH
  /home/meow/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:...
  ```

  We have two ways to run a program
  1. find it and run it
  2. if it's in $PATH, just run it 
  
  ```bash 
  meow@MyPC:~$ which date
  /usr/bin/date
  meow@MyPC:~$ /usr/bin/date --date='TZ="America/Los_Angeles" 09:00 next Fri' # find & run
  Sat Feb  7 01:00:00 CST 2026
  meow@MyPC:~$ date --date='TZ="America/Los_Angeles" 09:00 next Fri' # just run it
  Sat Feb  7 01:00:00 CST 2026
  ```

  **\[NOTE\]**  
  If a program represents in multiple paths, it will run the first one it encounters.  

  You can use `which {-a|--all} FILENAME` to list all programs with `FILENAME`  
  eg. I have two copies of python 
  ```bash
  meow@MyPC:~$ which -a python3
  /usr/bin/python3
  /bin/python3
  ```

- Like Regex, **Glob** (short for global) is another way for pattern matching. For instance, in `ls *.txt`, `*.txt` is glob expression (it uses `*`, `?`, `[]`, `{}`). [![][YT_ICON]](https://youtu.be/MSgoeuMqUmU?t=2386)   
  Glob is also called Wildcards.  
  Usually, paths are matched by glob, while text or string are matched by regex   

## An interesting example
[![](./static/example_combining_commands.png)](https://youtu.be/MSgoeuMqUmU?t=3228)
[![][YT_ICON]](https://youtu.be/MSgoeuMqUmU?t=3228)

## In the end 
Please read notes for more information. [![][MS_ICON_SMALL]](https://missing.csail.mit.edu/2026/course-shell/)  
  #### Things not covered in video:   
  - `set`
  - Exercises



[YT_ICON]: https://img.shields.io/badge/YouTube-%23FF0000.svg?style=flat-square&logo=YouTube&logoColor=white

[MS_ICON_SMALL]: https://missing.csail.mit.edu/static/assets/favicon-16x16.png
[MS_ICON_MIDDLE]: https://missing.csail.mit.edu/static/assets/favicon-32x32.png