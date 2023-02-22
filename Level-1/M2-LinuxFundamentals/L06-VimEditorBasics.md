# What is Vim ?

Vim is a Unix text editor that's included in Linux, BSD, and macOS. It's known for being fast and efficient, in part because it's a small application that can run in a terminal (although it also has a graphical interface), but mostly because it can be controlled entirely with the keyboard with no need for menus or a mouse. For instance, to insert text into a file, you press I and type. To navigate or to issue a command (such as Save, Backspace, Home, End, and so on), you press Esc on your keyboard and then press whatever key or key combination corresponds with the action you want to take. It's a very different way of editing text compared to what modern computer users expect, but it's the way Unix admins all over the world edit config files, changelogs, scripts, and more.

Vim is also commonly referred to as Vi because when it was written by Bill Joy in the late 1970s, it was short for visual editor. Before Vi, few people even imagined that a computer could act as a sort of interactive typewriter. Text files were edited with commands (like ed) that would find a specific line and either insert or remove text; literally all text was manipulated with what amounted to a rudimentary version of your favorite office application's find-and-replace menu (but without the office application). Vi was a breath of fresh air, enabling users to enter a screen session that showed them their entire file and allowed them to edit it live.

While many people claim to love and use Vi, few people use Vi over Vim on a daily basis. In casual conversation, Vi and Vim are interchangeable and usually refer to Vim (Vi Improved). On some POSIX systems, the vi command is a pointer to Vim (or else Vim is just called Vi). However, some systems ship just with Vi and you have to install Vim separately.

To verify whether you have Vim installed, use this command:

                which vim

If you get nothing in return, you don't have Vim installed. Install Vim from the project's homepage.

Furthermore, some distributions offer a few different builds of Vim. Usually, this means there's a minimal build that mimics the old Vi, with just enough Vim-isms included to make it usable, and an enhanced build that offers a truly modern Vim experience. The best way to determine what's available is to search your package manager for any package containing the string "vim" and then install the one with the most features (like vim-enhanced on Fedora).

![](Images/vim.png)

# Why use Vim?
Vim is the default fallback editor on all POSIX systems. Whether you've just installed the operating system, or you've booted into a minimal environment to repair a system, or you're unable to access any other editor, Vim is sure to be available. While you can swap out other small editors, such as GNU Nano or Jove, on your system, it's Vim that's all but guaranteed to be on every other system in the world.

Its efficiency in both design and function is hard to ignore, too. Vim's native terminal-based interface doesn't rely upon menus or fancy peripherals or even "extra" keys like Ctrl or Alt. Vim (mostly) uses keys common to any keyboard, regardless of language, layout, or device (and those that aren't common can be remapped pretty easily).

Vim on mobile devices
If you run a terminal on your mobile phone, Vim is a great choice for editing except for one problem: You probably don't have an Esc key. While there are alternate keyboards available for download, they often add extra keys at the expense of key size. If that's not a change you're willing to make, then you can set Vim to map Esc to some other key sequence. Add this imap command to your environment's .vimrc (this example uses two commas as the escape sequence, but you can use anything that you don't anticipate usually typing in quick succession, like qq or yy):

imap ,, <Esc>

# How to use Vim
There's no illusion that Vim is intuitive, so its developers have created vimtutor, a simple, interactive walkthrough of the basics. Although Vim is bursting with potential, there are only a few controls you need to know in order to use it. Paraphrasing vimtutor's first lesson, here are the essentials:

- Start Vim from a terminal by typing vim or on your desktop by launching gvim.
- Press I to enter insert text mode. When in insert mode, all you can do is type text into your document. There are no commands in insert mode.
- Press Esc to enter normal mode, used for commands.
- In normal mode, you can move your cursor with h (left), j (down), k (up), and l (right). It might help to remember that j is down by equating it, visually, with a Down arrow.
- To exit Vim, type :wq if you want to save your work or :q! to discard unsaved changes.
- Beyond these basics, all other Vim commands are arguably for convenience and efficiency.

## Vim Modes
There are two modes in vim. One is the command mode and another is the insert mode.

1. In the command mode, user can move around the file, delete text, etc.

2. In the insert mode, user can insert text.

### Changing mode from one to another

1. From command mode to insert mode type a/A/i/I/o/O. Any of the key can be pressed after opening the document in vim to enable insert mode.

2. From insert mode to command mode type Esc (escape key). Now one can use vim commands applicable with vim within or out of scope of the opened file.

3. From command mode to visual mode type v and use any arrow keys to select text lines. You can use Ctrl + v to take control over vertical text selection.

##  Vim insert mode commands
        a    Append text following current cursor position

        A    Append text to the end of current line

        i     Insert text before the current cursor position

        I     Insert text at the beginning of the cursor line

        o    Open up a new line following the current line and add text there

        O   Open up a new line in front of the current line and add text there

### Note: For a beginner this is not required to learn all the options but use the important one. Even me(satish) using 'i' option always.

## Editing Commands
```

d: delete the characters from the cursor position up the position given by the next command (for example d$ deletes all character from the current cursor position up to the last column of the line).

c: change the character from the cursor position up to the position indicated by the next command.

y: copy the characters from the current cursor position up to the position indicated by the next command.

p: paste previous deleted or yanked (copied) text after the current cursor position.
```

### Note: Doubling d, c or y operates on the whole line, for example yy copies the whole line.

## Exit Commands

```
:wq Write file to disk and quit the editor

:q! Quit (no warning)

:q Quit (a warning is printed if a modified file has not been saved)

ZZ Save workspace and quit the editor (same as :wq)

:f to find out the line numbers you want.
```
### For more information
https://www.youtube.com/watch?v=-txKSRn0qeA&t=364s

https://www.howtoforge.com/vim-basics 

