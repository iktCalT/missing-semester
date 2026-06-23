# Lecture 3: Development Environment and Tools [![Missing_Semester][ms32]](https://missing.csail.mit.edu/2026/development-environment/)

*A development environment is a set of tools for developing software. At the heart of a development environment is text editing functionality, along with accompanying features such as syntax highlighting, type checking, code formatting, and autocomplete.*

You can classify development environments into **terminal-based** and **integrated** (IDE). "integrated" means "everything in one place".  

For example, VS Code and Cursor are **IDEs** (Cursor is a fork of VS Code).

While Vim (a text editor) is a part of **terminal-based** development workflows. To develop in terminal, we usually need a terminal multiplexer (e.g. `tmux`), a text editor (e.g. `vim`), a shell (e.g. `bash`), and several language-specific command-line tools.

Although (graphical) IDEs are easier to learn and more extensible, we still need to learn terminal-based workflows. Because some environment doesn't support GUI or doesn't allow you to install software.  

---

Part I: [Text editing and Vim](#text-editing-and-vim)

Part II: [Code intelligence and language servers](#code-intelligence-and-language-servers)

Part III: [AI-powered development](#ai-powered-development)

Part IV: [Extensions and other IDE functionality](#extensions-and-other-ide-functionality)

## Text editing and Vim

When writing codes. We often need to jump from one file or snippet to another. Vim is optimized for this case.

Moreover, switching between keyboard and mouse is slow. I'd be much faster if we could keep our hands on keyboard. Vim supports this.

Vim is so powerful that even if in some non-vim softwares, vim mode key bindings are supported. e.g. VS Code, Zsh, Bash, Claude Code.

### How to use Vim

Some features are supported only by Neovim but Vim. And Neovim even support mouse click. So, please use Neovim.

Here are some examples of vim key bindings (only a tiny part). Just to illustrate how differently vim behaves compared to text editors like MS Word.

Please check [![missing_semester][ms16]](https://missing.csail.mit.edu/2026/development-environment/#modal-editing) and [![nvim][nvim]](https://neovim.io/doc/user/) for more vim instruction.  
Cheat sheet: [Vim cheat sheet](https://vim.rtorr.com/)

- `j` / `k`: move cursor down / up.  
  `10j`: move the cursor down by 10 lines.
- `ci`: change inside.  
  `ci(`: change text in next `()`.  
  `ci[`: change text in next `[]`.
- `f`: find one character forward on the current line (`/`: search regex).  
  Then you can use `;` / `,` to navigate matches. Or you can use `%` to find the matching character.  
  e.g. `f]`: find next `]` -> press `%`: jump to the matching `[`.

### Model editing

You can switch between different modes.

#### In Normal mode

- `i`: Insert mode
- `R`: Replace mode
- `v`: Visual plain mode
- `V`: Visual line mode
- `Ctrl-v` or `Ctrl-q`: Visual block mode
- `:`: Command-line mode

#### In any other mode

- `Esc`: Normal mode

### Use vim interface as a programming language

We will focus on **Normal mode** in this section.

#### Movement

- `h`, `j`, `k`, `l` or `↑`, `↓`, `→`, `←`: move around.
- `w`: move to the beginning of next word.  
  `b`: move to the beginning of the current / last word.
  `e`: move to the end of the word current / next word.
- `0`: go to the beginning of the line.  
  `$`: go to the end of the line.  
  `^`: go to the first non-blank character of the line.
- `H`, `M`, `L`: go to the top (head), middle, bottom (last) of screen.
- `Ctrl-d` / `Ctrl-u`: scroll down / up.
- `gg` / `G`: go to the beginning / end of the file.
- **[In command mode]** `:123`: go to the 123rd line.  
  Or, **[in normal mode]** `123G`: go to the 123rd line.
- `%`: find the matching item (like braces).
- `f` / `t`: find one character forward on the current line (`/`: search regex).  
  `F` / `T`: find one character backward on the current line.  
  After that, use `;` / `,` to move to next / previous result.
- `\{regex}`: search in the file.  
  `?{regex}`: search in the file (backward).  
  Press `Enter` to confirm. Then press `n` or `N` to go to the next or previous result.  
  Then you can use `Ctrl-o` to go to where your were. Or `Ctrl-i` to go forward.

#### Selection

Move in Visual mode to select.

- `v`: Visual (plain) mode
- `V`: Visual line mode
- `Ctrl-v` or `Ctrl-q`: Visual block mode  
  
In windows terminal, `Ctrl-v` might not trigger Visual Block mode. Because windows may replace `Ctrl-v` with paste for you.

##### In visual mode, you can

- You can use search (`/regex` or `?regex`) in visual mode to make it more efficient.

- You can use save (`:w filename`) to save the selected text.

- You can press `x` or `d` to delete. Press `c` or `s` to substitute.

#### Edits

In Normal mode:

- `i`: enter Insert mode.
- `a`: append after the cursor.  
  `A`: append at the end of the line.  
  `i`, `a`, `A` all go to Insert mod.
- `o` / `O`: create a new line below / above and then enter Insert mode.
- `c{movement}`: change (delete and enter insert mode).  
  `cc`: change the whole line (delete current line and enter insert mode).
- `d{movement}`: delete.  
  `dd`: delete current line.
- `x`: delete character (equivalent to `dl`).
- `s`: substitute character (=`cl`).
- Select text in Visual mode, then press `d` or `c` to delete or modify.
- **`u`: undo.**  
  `U`: restore last changed line. You can undo (`u`) a `U`.
- **`y`: copy ("yank").**
- **`p`: paste.**
- Substitute in command mode:  
  `:s/{old}/{new}`: replace the **first** `{old}` to `{new}` in the **current line**. e.g. `:s/thee/the`: change the first `thee` to `the` in current line.  
  `:s/{old}/{new}/g`: replace the **first** `{old}` with `{new}` in the **current line**.  
  `:{start},{end}s/{old}/{new}/g`: substitute between `{start}` and `{end}`. Where `{start}` and `{end}` are line numbers.  
  `:%s/{old}/{new}gc`: change in the whole file with prompt whether to replace or not.  
  `s` means substitute; `g` means globally; `{start},{end}` or `%` are the range (`%` represents the whole file); `c` means with prompt.
There are many other to learn.

#### Counts

You can combine nouns (movements) and verbs (edits) with a count.

- `3j`: move down 3 lines (do `j` 3 times).
- `10dw`: delete next 10 words (do `dw` 10 times).
- `5u`: undo 5 times.

#### Modifiers

- `i`: inside / inner
- `a`: around

For example: *`ci(`: change the contents inside the current pair of parentheses; `ci[`: change the contents inside the current pair of square brackets; `da'`: delete a single-quoted string, including the surrounding single quotes.*

`ciw`: change entire word (change contents inside a word).

#### Others

- `:q!`, `ZQ`: quit without saving.
- `:wq`, `:x`, `ZZ`: write (save) and quit.
- `:w`: just write (save) current changes. `:w filename`: save as filename. e.g. `:w test.txt` or `:w ~/test.txt`
- `Ctrl-g`: show your location in the file and file status.
- `!{command}`: execute command in shell. e.g. `!ls`.
- `:r filename`: retrieve a file (get and paste its contents after cursor).  
  `:r !command`: paste the output of command. e.g. `:r !ls`
- `:set setting` / `:set nosetting`: enable / disable some settings. (Search vim configuration for more information)  
  `:set number`: show line number. `:set nonumber`: disable `:set number`.  
  `:set ic`: ignore upper/lower case while searching. `:set noic`: disable `:set ic`.  
  `:set hls` or `:set hlsearch`: highlight searching results.  
  `:set is` or `:set incsearch`: highlight matching result while typing in.
- `:help` or `:help {something}`: to open help window.  
  `{something}` are something like `w`, `c_CTRL-D`, `insert-index`, `user-manual`.  
- If you have multiple windows.  
  Use `Ctrl-w Ctrl-w` to switch between windows. (Yes! type `Ctrl-w` twice.)  
  Uee `:q` to quit unnecessary windows.
- In command mode, you can use `Ctrl-D` to list all possible commands. And use `Tab` to complete.
- To configure your vim—edit `~/.vimrc`.

#### Practice

Fix this broken python code with vim.

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print("fizz", end="")
        if i % 5 == 0:
            print("fizz", end="")
        if i % 3 and i % 5:
            print(i, end="")
        print()

def main():
    fizz_buzz(20)
```

Use `vimtutor` or play [vim adventures](https://vim-adventures.com/) (NOT free) to improve your vim skills.

## Code intelligence and language servers

Language-specific support in an IDE is achieved by communicating with language server (who provide language-specific support) through Language Server Protocol (LSP).

*By installing the extension and language server for the languages you work with, you can enable many language-specific features in your IDE, such as:*  
![Language-specific support](./static/language-specific-support.png)

## AI-powered development

There are 3 main ways for people to use AI tools helping them write codes.

- [Autocomplete](#autocomplete)
- [Inline chat](#inline-chat)
- [Coding agents](#coding-agents)

Privacy issues: ummm.... [![47:16][yt]](https://youtu.be/QnM1nVzrkx8?t=2836)

These tools are continuously evolving. So, please pay attention to that.

### Autocomplete

You can passively accept AI's suggestions or steer it by writing comments.

Autocomplete has limited scope.

In VS Code, press `Tab` to accept AI's suggestions.

### Inline chat

In VS Code, you can select the text first and then press `Ctrl-i` to trigger AI in inline-chat mode.

Inline chat can only modify selected section.

### Coding agents

Will be covered in [Lecture 7: Agentic coding](https://missing.csail.mit.edu/2026/agentic-coding/).

### Recommended software

- GitHub Copilot in VS Code (free for students and teachers)
- Cursor (a VS Code fork)

## Extensions and other IDE functionality

There are a lot of awesome VS Code extensions making your life easier.

There are 3 types of extensions which are useful for your future career. The follow part is copied from official notes.

- *Development containers: supported by popular IDEs (e.g., supported by VS Code), dev containers let you use a container to run development tools.* **Lecture 6: Packaging and Shipping Code** [![missing semester][ms16]](https://missing.csail.mit.edu/2026/shipping-code/) will cover more about this topic.
- *Remote development: do development on a remote machine using SSH (e.g., with the Remote SSH plugin for VS Code).*
- *Collaborative editing: edit the same file, Google Docs style (e.g., with the Live Share plugin for VS Code).*

[yt]: https://img.shields.io/badge/YouTube-%23FF0000.svg?style=flat-square&logo=YouTube&logoColor=white
[nvim]: https://img.shields.io/badge/Neovim-%57A143.svg?style=flat&logo=neovim&logoColor=white
[ms16]: https://missing.csail.mit.edu/static/assets/favicon-16x16.png
[ms32]: https://missing.csail.mit.edu/static/assets/favicon-32x32.png
