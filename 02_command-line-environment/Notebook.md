# Lecture 2: Command-line Environment
Part I: [The Command Line Interface](#the-command-line-interface)  

Part II: [Remote Machines](#remote-machines)  

Part III: [Terminal Multiplexers](#terminal-multiplexers)

Part IV: [Customizing the shell](#customizing-the-shell)  

Part V: [AI in the Shell](#ai-in-the-shell)

Part VI: [Terminal Emulators](#terminal-emulators)

Appendix: [Tips](#tips)


# The Command Line Interface
We will discuss concepts in this example: 
```bash
#!/usr/bin/env bash
if [[ -f $1 ]]; then
    echo "Target file already exists"
    exit 1
else
    if $DEBUG; then
        grep 'error' - | tee $1
    else
        grep 'error' - > $1
    fi
    exit 0
fi
```

- Arguments
- Streams
- Environment variables
- Return codes
- Signals

## Arguments
For example, in command `ls -l FOLDER/`, the program is `ls`, and its arguments are `-l` and `FOLDER/`.  

Most programming languages have a way to get their arguments.  
e.g. in Python, we have  
```python
# arguments.py
import sys
if __name__ == "__main__":
  print(sys.argv)
```

Then use the following command below, we can see the arguments:  
```bash
meow@MyPC:~$ python arguments.py -abcd xyz
['arguments.py', '-abcd', 'xyz']
```
  ### Arguments access
  `$1`–`$9`: access the first to ninth argument.
  `$0`: access program name (can be regarded as 0th argument).
  `$@`: access the list of all arguments.
  `$#`: retrieve the number of arguments.

  ### Flags
  Usually, arguments are mixture of *flags* and regular string. Flags starts with `-` or `--` (how they are identified). They tells the program how to behave.  

  By convention, single dash `-` is followed by one letter (`-a`, `-l`, `-h`), double dash is followed by letters (`--all`, `--help`, `--version`)  

  You can use multiple flags in one command. For example, `ls -a -l`. And usually, many single character flags can combine. `ls -a -l` is equivalent to `ls -al`. And the order usually doesn't matter. So `ls -la` is equivalent to `ls -al`.   

  Please note that **these features are not provided by shell**. They are implemented in different ways (or not implemented) for different programs.
  
  It depends on the program how to parse and handle these arguments (flags and regular strings). But there are some libraries (e.g. `argparse` in python) to help programmers parse these arguments.   

  ### Multiple arguments with same type
  Usually, CLI programs can accept multiple arguments with same type. 
  If you check `mkdir`'s help page. You'll find `DIRECTORY...`, which means `mkdir` can create multiple at the same time.  
  ```bash
  mkdir --help
  Usage: mkdir [OPTION]... DIRECTORY...
  ```  

  Instead of using `mkdir src` followed by `mkdir bin`, you can use `mkdir src bin` to create two folders directly.  

  ### Multiple arguments + golbbing
  This feature makes it powerful when combining with globbing. 

  For example, `touch test/{a,b,c}.py` is equivalent to `touch test/a.py test/b.py test/c.py`; and `ls {.,/}` is equivalent to `ls . /` (list all files in home`~` and root`/`); similarly, `ls /lib*` is equivalent to `ls /lib /lib32 /lib64 /lib.usr-is-merged /libx32`   

  When shell sees `touch test/{a,b,c}.py`, it will not pass `test/{a,b,c}.py` as an argument to `touch`. But instead, it will expend `test/{a,b,c}.py` to `test/a.py test/b.py test/c.py` and pass these 3 arguments to `touch`.

  Golobbing:
  - `*`: matches zero or more of anything. 
  - `**`: (only supported by some shells, e.g. `zsh`) matches all nested folders.  
  - `?`: matches exactly one character of anything. (e.g. `file?.txt` matches `file1.txt` and `file_.txt`. But it doesn't match `file.txt` or `file12.txt`)
  - `{}`: expand a comma-separated list of patterns into multiple arguments

## Streams
When we are using pipeline like 
`cat numbers.txt | grep -P '^\d$' | sort | uniq -c`  

We are **NOT** running those programs one by one. But we are running all of them at the same time. We just connect the standard output of previous command to the standard input of the next command. So, when a former program hasn't give an output, the programs behind it will wait for it.  
Here is a good example [![][YT_ICON]](https://youtu.be/ccBGsPedE9Q?t=683).  

  ### Standard input and output  
  In many programs, `-` is not a flag, it tells the program to read from standard input. For example: `grep "hello" -`. 

  Every program has one standard input (stdin) and two types of standard output (**standard output stream (stdout)** and **standard error stream (stderr)**). 
  By default, both stdout and stderr are pipe to the next program. e.g. `ls folder | cat`, no mtter `folder` exists or not, cat will print the result of `ls`.

  But you can also redirect stdin, stdout, and stderr.
  ```bash
  # If succeeded(1), goes into output.txt (>output.txt is equivalent to 1>output.txt)
  meow@MyPC:~$ ls Downloads >output.txt 2>err.txt
  meow@MyPC:~$ cat output.txt
  datalab-handout.tar
  meow@MyPC:~$ cat err.txt

  # If failed(2), goes into err.txt
  meow@MyPC:~$ ls nonexistance >output.txt 2>err.txt  # this command (ls nonexistance) will fail, its output should go be stored in err.txt
  meow@MyPC:~$ cat output.txt
  meow@MyPC:~$ cat err.txt
  ls: cannot access 'nonexistance': No such file or directory
  ```

## Environment variables
  ### Shell Variables - direct assign
  To assign an environment variable, use `foo=bar`. To access that variable, use `echo $foo`.  
  For example: 
  ```bash
  meow@MyPC:~$ foo=bar
  meow@MyPC:~$ echo $foo
  bar
  ```
  Please note that `foo = bar` is invalid. The shell will regard `foo` as a program with arguments `['=', 'bar']`.  

  We can also assign a command to an environment variable:  
  ```bash
  meow@MyPC:~$ foo=$(ls -a)
  meow@MyPC:~$ echo $foo
  . .. .android android .atuin .aws .azure #...
  ```  

  \[**Note**\]: Assigning environment variable in the way above only affects the current shell. It is invisible to other session and even to its scripts or programs.   
  ```bash
  meow@MyPC:~$ DEBUG=1
  meow@MyPC:~$ bash -c 'echo $DEBUG'  # Run the cmd in a child process (and nothing returns)

  meow@MyPC:~$ DEBUG=2 bash -c 'echo $DEBUG'  # To pass it to the process, we need to bind the assignment and the command 
  2
  meow@MyPC:~$ echo $DEBUG  # And it won't change $DEBUG outside
  1
  ```

  Another example:  
  ```bash
  meow@MyPC:~$ TZ=Asia/Tokyo date  # Change the time zone to Tokay then call date
  Fri Feb  6 05:49:40 PM JST 2026
  ```  

  ### Environment variables - export
  Another way is to use `export`. `export DEBUG=1` will make sure the current shell and its child processes can see the change.  
  ```bash
  meow@MyPC:~$ export foo=bar
  meow@MyPC:~$ bash -c "echo $foo"  # The child process can see foo now
  bar
  meow@MyPC:~$ foo=baz
  meow@MyPC:~$ echo $foo
  baz
  meow@MyPC:~$ bash -c "echo $foo"  # Note that even if I want to change it only in this shell, its child processes can also see the change
  baz
  ```  

  The example above seems tell us that: If we just say `foo=bar`, shell places `foo` in its private storage. If we say `export foo=bar`, shell places `foo` in a shared storage, so that its child processes can access it. <span style="color:#ef6f6f">***But this understanding is inaccurate!***</span>  

  ```bash
  meow@MyPC:~$ unset foo  # delete foo
  meow@MyPC:~$ export foo=bar
  meow@MyPC:~$ bash -c "echo $foo"
  bar
  meow@MyPC:~$ bash -c "foo=baz"
  meow@MyPC:~$ bash -c "echo $foo"
  bar
  meow@MyPC:~$ echo $foo  # child processes cannot change the variable outside
  bar
  ```   
  See, child processes cannot modify the `foo` in its parent, which means that there are **two copies of `foo`**. So, actually, when we type in `export foo=bar`, it will pass a **copy** of `foo` to every child process.  

  Please note that:
  ```bash
  meow@MyPC:~$ bash -c 'foo=baz; echo $foo'  # as expected, child can change its copy
  baz
  meow@MyPC:~$ bash -c "foo=baz; echo $foo"  # if we use double quotes, it will print the value in parent shell
  bar
  ```  
  Why is that? I checked the differences between single and double quotes [![][GNU_ICON]](https://www.gnu.org/software/bash/manual/bash.html#Quoting). 

  - In double-quote cases, parent shell will replace `$foo` with the `$foo` in itself (which is `bar`) before `"foo=baz; echo $foo"` is transferred to the child shell. So, what the child shell see is `foo=baz; echo bar`. Because *The characters `$` and `` ` `` retain their special meaning within double quotes*. 
  
  - However, in single-quote cases, *single quotes (`'`) preserves the literal value of each character within the quotes*. So parent shell will pass `'foo=baz; echo $foo'` to its child shell. So, what the child shell see is `foo=baz; echo $foo`. 
  

  ### Permanent environment variables - write in shell config file  
  The only way to set permanent environment variables is to write `export foo=bar` in **shell configuration file** (`~/.bashrc`, `~/.zshrc`, ...)  

  So, in my understanding, there is no actual "permanent" environment variable in Linux (except for those in `/etc/environment`). It seem permanent because we assign a value to it (with `~/.bashrc` or other scripts) when we create a new session.

  ### Remove a shell/environment variable
  Use `unset foo`.  

## Return codes
Use `echo $?` to check the return code of previous command.

## Signals
- When we press `Ctrl-C`, it shell will deliver a `SIGINT` (signal #2, means "interrupt") to the current process. Each program can define how to handle signals (in most cases).  
For example: 
  ```python
  #!/usr/bin/env python

  # This python program will not stop even if we press Ctrl-C
  import signal, time

  def handler(signum, time):
      print("\nI got a SIGINT, but I am not stopping")

  signal.signal(signal.SIGINT, handler)
  i = 0
  while True:
      time.sleep(.1)
      print("\r{}".format(i), end="")
      i += 1
  ```
  
- `Ctrl-\`: send `SIGQUIT` (#3) to current process, which is more aggressive than `SIGINT`.
  
- `Ctrl-Z`: suspend the process. To check it, use `jobs`. To recover it, use `fg` (foreground).   
  
- `kill`: send signal to specific process. See [Tips-kill](#tips-kill). Use `kill -l` to check all signals. 

Commonly used signals (summarized by Gemini, converted on [tabletomarkdown](https://tabletomarkdown.com/)):   
  | **#** | **Name** | **Action** | **Description** |
  | --- | --- | --- | --- |
  | **1** | **SIGHUP** | Terminate / Restart | **Hangup:** Used to tell a background process (daemon) to reload its configuration files without a full restart. |
  | **2** | **SIGINT** | Terminate | **Interrupt:** Triggered by `Ctrl+C`. Gracefully stops a process; often caught to perform cleanup. |
  | **3** | **SIGQUIT** | Terminate + Core | **Quit:** Triggered by `Ctrl+\`. Like SIGINT, but creates a "core dump" file for deep debugging. |
  | **9** | **SIGKILL** | Terminate (Forced) | **Kill:** Immediate exit. The process cannot catch or ignore this. Use only when a process is "stuck" or unresponsive. |
  | **10/12** | **SIGUSR1/2** | Terminate (Default) | **User Defined:** Custom triggers for your code (e.g. toggling debug logs or clearing internal caches). |
  | **11** | **SIGSEGV** | Terminate + Core | **Segfault:** The program tried to access restricted memory. Indicates a bug in memory management/pointers. |
  | **13** | **SIGPIPE** | Terminate | **Broken Pipe:** Sent when a program writes to a pipe that has no reader. Common in CLI tool chains. |
  | **15** | **SIGTERM** | Terminate | **Software Termination:** The "polite" kill. Standard for orchestrators (like Kubernetes) to request a shutdown. |
  | **17** | **SIGCHLD** | Ignore (Default) | **Child Status:** Notifies parent when a child process finishes. Must be handled to avoid "Zombie" processes. |
  | **18** | **SIGCONT** | **Resume** | **Continue:** Tells a process previously stopped by SIGSTOP/SIGTSTP to resume execution in the background. |
  | **19/20** | **SIGSTOP/TSTP** | Stop (Pause) | **Stop:** Pauses the process. SIGTSTP is sent by `Ctrl+Z`; SIGSTOP is a forced pause that cannot be ignored. |

Manual of all standard signals [![][GNU_ICON]](https://sourceware.org/glibc/manual/latest/html_node/Standard-Signals.html).

# Remote Machines
`ssh`: secure shell.  

You can use `ssh user@host` to login. If it's the first time to login, you will be asked to type in a password to identify yourself.  

After logging in, you will notice that the prompt shows that you are using the remote machine.  

## SSH keys
With SSH key, you don't need to type in password every time. For example, my private key is stored in `~/.ssh/id_ed25519`. And my public key is stored in `~/.ssh/id_ed25519.pub`.   

You can paste your **public key** to GitHub or remote machine to let it know how you are. But you should <span style="color:#ef6f6f">**NEVER** paste your **private key** to anywhere</span>.  

If you don't want to type in password again, you can manually send your public key to the remote machine (e.g. via `ssh-copy-id user@host`). The remote machine will add your public key to its library (`~/.ssh/authorized_keys`). So, next time you tries to login, if your public key matches any one in `authorized_keys`, the remote machine will let you login directly without password.  

## SSH matching
Please note that your public key is 'public', anyone can get it. But only you know your private key.

So the matching process mentioned above is not as simple as just check if it exists in the file `authorized_keys`. Searching in `authorized_keys` is the first step. If exists, remote machine will send a message encrypted with **public key**, which can be decrypted with your **private key**. So local machine will decrypt and send it back, if it the remote machine verifies the correctness of message, matching succeeds.

  ### Generating a SSH key
  Here is an example of generating a SSH key: `ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519`.  
  - `ssh-keygen`: The program for generating SSH keys.
  - `-a 100`: Run hashing function 100 to generate the key.  
  - `-t ed25519`: Specifies the algorithm. *Ed25519* is currently the most recommended algorithm for SSH keys.  
  - `-f ~/.ssh/id_ed25519`: Specifies the filename and location where the key will be saved.

## SSH execute
Use `ssh user@host command`, the remote will run the `command` (e.g. `ls`) and print the output to your screen, then goes back to the local machine, please notice the change (no change) of prompt.  
e.g. `ssh user@host ls | wc -l`: runs `ls` on **remote machine** and `wc` will count the number of lines on **local machine**.  

If we wish to run both commands on remote machine, warp them with single quotes—`ssh user@host 'ls | wc -l'`

## Copy files
`scp`: secure copy (using SSH under the hood).  
e.g. `scp test.py user@host:/home/`  
`rsync`: a better implementation of `scp`

# Terminal Multiplexers
`tmux`: Run multiple processes in the same environment.  

It uses strange key bindings. 
- Press `Ctrl-B`, then release it, then press `C`—create a new window (let's note it `<Ctrl b><c>`).
- `<Ctrl b><d>`: Detach from the current session. `tmux attach`: Re-attach.  
  Note that if we use `Ctrl-D` directly, it will close the current window (kill the process).
- `<Ctrl b><w>` or `<Ctrl b><0–9>`: Switch between sessions and windows.
- More `tmux` instructions.  
  - Easier—A Quick and Easy Guide to tmux <a href="https://hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/"><img style="height:16px" src="https://hamvocke.com/img/beerpig.svg"></a>.   
  - Detailed—Terminal Multiplexers <a href="https://linuxcommand.org/lc3_adv_termmux.php"><img style="height:16px" src="https://linuxcommand.org/favicon.png"></a>.   
  - Simpler—Section "Terminal Multiplexers" of Missing Semester. [![][MS_ICON]](https://missing.csail.mit.edu/2026/command-line-environment/)


  ### Tmux and remote machine
  Usually, when we disconnect the remote server, it will kill all running processes with signal `SIGHUP`.  

  But if the processes are running in `tmux` on remote machine, it will continue running when we disconnect the remote machine. And it can be brought back easily with `tmux attach`.

# Customizing the shell
You can customize the shell by editing the shell configuration file (for bash it's `~/.bashrc`). 

Configuration files of other programs
- bash - ~/.bashrc, ~/.bash_profile
- git - ~/.gitconfig
- vim - ~/.vimrc and the ~/.vim folder
- ssh - ~/.ssh/config
- tmux - ~/.tmux.conf
  
You can read others' dotfile for more information. For example, this one [![][GitHub]](https://github.com/mathiasbynens/dotfiles) (although it's designed for macOS, there are a lot of useful instructions in this repository).  

And it's a great practice to place your configuration file on GitHub so that you can sync them on different machines.
  
  ## Reload modified configuration 
  `source ~/.bashrc`: Run the bash configuration file immediately.  
  
  ## Frameworks and plugins
  There are a lot of useful plugins available. Like [powerlevel10k](https://github.com/romkatv/powerlevel10k) (a zsh theme including many useful functions), [starship](https://github.com/starship/starship), etc..
  
  But please note that installing too many plugins will make your shell run slowly. So, please install them one by one and remove those you don't need.  

# AI in the Shell
You can use AI tools to give you the correct program with flags to do the job you tell it.  
For example:  
```bash
$ llm cmd "find all python files modified in the last week"
find . -name "*.py" -mtime -7
```
```bash
$ cat users.txt
Contact: john.doe@example.com
User 'alice_smith' logged in at 3pm
Posted by: @bob_jones on Twitter
Author: Jane Doe (jdoe)
Message from mike_wilson yesterday
Submitted by user: sarah.connor
$ INSTRUCTIONS="Extract just the username from each line, one per line, nothing else"
$ llm "$INSTRUCTIONS" < users.txt
john.doe
alice_smith
bob_jones
jdoe
mike_wilson
sarah.connor
```
All examples in this section are from the official notes. [![][MS_ICON]](https://missing.csail.mit.edu/2026/command-line-environment/) 

  ## Claude Code
  Claude code is useful, try it.

# Terminal Emulators
*A terminal emulator is a GUI program that provides the text-based interface where your shell runs.* 

*Some of the aspects that you may want to modify in your terminal include:*

- *Font choice*
- *Color Scheme*
- *Keyboard shortcuts*
- *Tab/Pane support*
- *Scrollback configuration*
- *Performance (some newer terminals like [Alacritty](https://github.com/alacritty/alacritty) or [Ghostty](https://ghostty.org/) offer GPU acceleration).*

# Tips
- `ps`: information about running processes  
  
- Short-circuiting operators: `&&`—The right program will run only if the left one runs successfully. `||`—The right program will run only if the left one fails.  
  
- `grep -q`: it is equivalent to `grep -quiet`, don't print the output on screen.  
  
- `jobs`: show information about unfinished processes spawned by the current shell.  
  For example: 
  ```bash
  jobs -l  # List jobs with process IDs
  [1]- 24336 Stopped          sleep 30
  [2]+ 24409 Stopped          sleep 60
  # Where 1 and 2 are job_ids, 
  # 24336 and 24409 are process_ids
  ```

- `pgrep`: find or signal processes by name.  
  For example: 
  ```bash
  meow@MyPC:~$ sleep 60
  ^Z
  [1]+  Stopped         sleep 60
  meow@MyPC:~$ sleep 100
  ^Z
  [2]+  Stopped         sleep 100
  meow@MyPC:~$ pgrep sleep
  35984
  36191
  meow@MyPC:~$ jobs
  [1]- 35984 Stopped         sleep 60
  [2]+ 36191 Stopped         sleep 100
  ```

- `fg`: bring most recently **suspended** or **running background** job to foreground. `fg %job_id` (e.g. `fg $2`): Bring a specific job to foreground. `bg`: same as `fg`, but the job will run in background.   
  
- `top`: display Linux processes.  
  
- <span id="tips-kill"></span>`kill`: can send any signal to a process. You can use this syntax—`kill [-s sigspec | -n signum | -sigspec] process_id | %job_id ...`. If no signal is provided, it will be `SIGTERM` by default.    
  ```bash
  # To continue previous jobs, use
  kill -SIGCONT 24336 24409
  # or
  kill -SIGCONT %1 %2
  #or 
  kill -n 18 %1 %2
  # SIGCONT (#18): continue
  ```

- `wc`: Count lines, words, and bytes.  
  `wc -l` or `wc --lines`: Count all lines in a file. For more information, see `wc --help`.

- The programs' name while installing and running might be different. You can search for running command in [this website](https://command-not-found.com/). It also tells you how to install those programs.

# In the end 
Please read notes for more information. [![][MS_ICON]](https://missing.csail.mit.edu/2026/course-shell/)  
  ## Things not covered in video:
  - Process substitution, `<( CMD )`. *This is useful when commands expect values to be passed by file instead of by STDIN*
  - suffix `&`: run in background (e.g. `sleep 60 &`).  
  - `nohup`: tell the program to ignore `SIGHUP`.  
    If we use `tmux`, we don't need `nohup`.
  - `trap`: execute commands when signals are received.  
  For example:  
    ```bash
    trap cleanup EXIT  # Run cleanup when script exits
    trap cleanup SIGINT SIGTERM  # Also on Ctrl-C or kill
    ```
  - Configuration file of SSH `~/.ssh/config`.
  - *Warning about `curl | bash`*  

    *Instructions like `curl -fsSL https://example.com/install.sh | bash` downloads a script and immediately executes it, which is convenient but risky.*  

    Please follow the steps below to install a package.  
    ```bash
    curl -fsSL https://example.com/install.sh -o install.sh  # download the script
    less install.sh  # review the script
    bash install.sh  # run it
    ```
  - `alias`: rename commands. It can make your life much easier.
    - `\alias_name`: ignore alias. 
      ```bash
      meow@MyPC:~$ alias ls="ls -a"
      meow@MyPC:~$ ls  # ls -a
      .  ..  [other folders] 
      meow@MyPC:~$ \ls  # ls
      [other folders]
      ```
    - `unalias alias_name`: delete alias. 
    - `alias alias_name`: show alias's definition

    Alias cannot take arguments in the middle. Arguments can be appended to the end.  
    For example:  
    ```bash
    meow@MyPC:~$ alias ll='ls -l'
    meow@MyPC:~$ ll
    [result of ls -l]
    meow@MyPC:~$ ll -a  # arguments can be appended to the end of alias
    [result of ls -l -a]
    ```
  - `fzf`: fuzzy finder. I use `atuin`.



[GNU_ICON]: https://img.shields.io/badge/-GNU-white?style=flat&logo=gnu&logoColor=black
[YT_ICON]: https://img.shields.io/badge/YouTube-%23FF0000.svg?style=flat-square&logo=YouTube&logoColor=white
[GitHub]: https://img.shields.io/badge/-dotfiles-white?style=flat&logo=github&logoColor=181717
[MS_ICON]: https://missing.csail.mit.edu/static/assets/favicon-16x16.png