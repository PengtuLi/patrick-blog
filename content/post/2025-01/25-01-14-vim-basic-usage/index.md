---
title: "Vim Basic Usage"
author : "tutu"
description:
date: '2025-01-14'
lastmod: '2025-01-14'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['tool']
---

参考资料：
- https://learnxinyminutes.com/vim/
- vim自带教程：vimtutor
- https://www.youtube.com/watch?v=KYDG3AHgYEs&t=2s
- ...

## mode of vim
- Normal Mode - vim starts up in this mode, used to navigate and write commands
- Insert Mode - used to make changes in your file
- Visual Mode - used to highlight text and do operations to them
- Command-line mode is used to enter Ex commands (":"), search patterns ("/" and "?"), and filter commands ("!").

In big approximation: normal (command) mode is for keybindings and command line is for "shell" commands (similarity to Bash, Zsh, sh, CMD etc.)

```text
# mode
i   # Puts vim into insert mode, before the cursor position
a   # Puts vim into insert mode, after the cursor position
v   # Puts vim into visual mode
:   # Puts vim into ex mode
<esc>   # 'Escapes' from whichever mode you're in, into Normal mode

# Copying and pasting text
y   # Yank whatever is selected
yy  # Yank the current line

d   # Delete whatever is selected
dd  # Delete the current line

p   # Paste text in vim register after the current cursor position
P   # Paste text in vim register before the current cursor position

x   # Delete character under current cursor position

u   # Undo
CTRL+R  # Redo
```
## The 'Grammar' of vim

**Vim can be thought of as a set of commands in a 'Verb-Modifier-Noun' format, where:**
1. Verb - your action
3. Modifier - how you're doing your action
2. Noun/Motion - the object on which your action acts on

```raw
# 'Verbs'
d                 # Delete
c                 # Change
y                 # Yank (copy)
v                 # Visually select

# 'Modifiers'
i                 # Inside
a                 # Around
NUM               # Number (NUM is any number)
/                 # Finds a string from cursor onwards
?                 # Finds a string before cursor

# 'Nouns' or 'Motion'
w                 # Word
s                 # Sentence
p                 # Paragraph
b                 # Block

# Sample 'sentences' or commands
d2w               # Delete 2 words
cis               # Change inside sentence,就是把选中的句子替换为cursor
yip               # Yank inside paragraph，即为复制这个段落
d$                # Delete till end of line

```

## Macros

宏基本上是可录制的操作。当您开始录制宏时，它会记录您使用的每个操作和命令，直到您停止录制。

```raw
qa                # Start recording a macro named 'a'
q                 # Stop recording
@a                # Play back the macro
```

## 速记

```raw
vim <filename>      # Open <filename> in vim
:help <topic>       # Open up built-in help docs about <topic> if any exists
crtl+d,tab          # list completing command, completing command
crtl-w [h,j,k,l]    # switch from window
:split,:vsplit [fn] # split
crtl-w [H,J,K,L]    # move window
crtl-w [=,<,>,-,+]  # adjust window size
:omly               # close other window

i,a,A,o,O           # 插入模式,cursor前，后，句子末尾,下一行，上一行
d[w,0,$],c[w,0,$]   # delete,delete and inset | 光标左边至下个单词，行首，行末
[NUM]dd             # 删除几行
p,P                 # paste register after,before cursor, for a line after,before current line
r[x],R[x...]        # replace the word of cursor with 'x', replace multiple

:q                  # Quit vim
:w                  # Save current file
:wq                 # Save file and quit vim
:q!                 # Quit vim without saving file
:! [command]        # exec external shell command

u,U,crtl+r          # undo one,all actions;redo one action
hjkl                # 移动
[NUM]+b,w,e         # 移动到前NUM个单词，后NUM个单词，后NUM个单词最后字符
0,^,$               # 每行-开始，非空，最后字符
gg,[NUM]G,:NUM      # goto文件首行，[NUM行]或文件末行，指定行数
crtl+o,crtl+i       # jump forward or downward
crtl+g              # show file status
%                   # jump between paired ()[]{}
H,M,L               # goto屏幕首行，屏幕中间，屏幕末行
crtl+u,crtl+d       # scroll between screen

?word,/word         # Highlights all occurrences of word before,after cursor
N,n                 # Moves before,after search

:%s/foo/bar/g       # Change 'foo' to 'bar' on the file,没有g只会替换每行的第一个
:s/foo/bar/g        # Change 'foo' to 'bar' on the current line
:'<,'>s/foo/bar/g   # Change 'foo' to 'bar on every line in the current visual selection
:[#,#]s/foo/bar/gc  # change for line # to # and prompt whether to delete

f<                  # Jump forward and land on < in a line
t<                  # Jump forward and land right before < in a line
```



代整理
```raw
>                 # Indent selection by one block
<                 # Dedent selection by one block
:earlier 15m      # Reverts the document back to how it was 15 minutes ago
:later 15m        # Reverse above command
ddp               # Swap position of consecutive lines, dd then p
.                 # Repeat previous action
:w !sudo tee %    # Save the current file as root
:set syntax=c     # Set syntax highlighting to 'c'
:sort             # Sort all lines
:sort!            # Sort all lines in reverse
:sort u           # Sort all lines and remove duplicates
~                 # Toggle letter case of selected text
u                 # Selected text to lower case
U                 # Selected text to upper case
J                 # Join the current line with the next line

# Fold text
zf                # Create fold from selected text
zd                # Delete fold on the current line
zD                # Recursively delete nested or visually selected folds
zE                # Eliminate all folds in the window
zo                # Open current fold
zO                # Recursively open nested or visually selected folds
zc                # Close current fold
zC                # Recursively close nested or visually selected folds
zR                # Open all folds
zM                # Close all folds
za                # Toggle open/close current fold
zA                # Recursively toggle open/close nested fold
[z                # Move to the start of the current fold
]z                # Move to the end of the current fold
zj                # Move to the start of the next fold
zk                # Move to the end of the previous fold

```

## plugin

- neotree
```raw
:neotree,\          # open neotree
a,r,d             # new,rename,delete


```