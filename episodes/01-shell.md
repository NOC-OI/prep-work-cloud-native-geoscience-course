---
title: Getting Started with the Shell
teaching: 30
exercises: 15
---


:::::::::::::::::::::::::::::::::::::::::: objectives

- "Install a shell (Bash or equivalent) and open a terminal on your operating system."
- "Explain what a shell is and why it is useful for scientific and data workflows."
- "Open a terminal and run simple commands safely."
- "Navigate the filesystem with `pwd`, `ls`, and `cd`."
- "Create, move, copy, rename, and remove files and directories with core shell commands."
- "Use tab completion and built-in help (`--help`, `man`) to discover command usage."

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::: questions

- "How do I install and open a shell on my computer?"
- "What is a shell, and why should I use it instead of only a graphical interface?"
- "How do I move around directories and inspect files from the command line?"
- "How do I do common file and folder operations in Bash?"
- "How can I get help when I do not remember a command?"

::::::::::::::::::::::::::::::::::::::::::::::::::


## Install and open a shell

If you do not already have shell software installed, follow the Carpentries installation instructions:

- https://carpentries.github.io/workshop-template/install_instructions/#shell

After installation, open a new terminal window.

- Windows: open Git Bash, WSL terminal, or another Bash-compatible terminal.
- macOS: open Terminal (Applications -> Utilities -> Terminal).
- Linux: open your system Terminal application.

Once the terminal opens, type:

```bash
cd
```

This returns you to your home directory and gives everyone a consistent starting point for the lesson.

If your terminal is already configured and opens correctly, you can continue directly to the next section.

## Why use the shell?

Many scientific workflows rely on command-line tools, including remote systems, HPC environments, and cloud processing pipelines.
The shell helps you:

- Automate repetitive tasks.
- Combine simple tools into larger workflows.
- Work efficiently on local and remote machines.

In this course we use Bash, one of the most common Unix shells.
A shell reads the commands you type, runs programs, and prints the output.

For prep work, think of the shell as a universal interface: even when tools differ across institutions, the basic command-line workflow is often the same. Learning this now makes later lessons smoother because you can focus on data concepts instead of terminal basics.

## Opening a shell and running first commands

Open a terminal and try:

```bash
echo "Hello World"
whoami
date
```

Each line is a command.
You are always working from a current directory.

These first commands help learners build confidence:

- `echo` prints text, so you can confirm command syntax quickly.
- `whoami` shows your current user, which is useful on shared systems.
- `date` confirms system time and demonstrates command output.

## Navigating files and directories

### Print working directory

```bash
pwd
```

`pwd` prints the path of your current directory.
It is good practice to run `pwd` whenever you are unsure where a command will run.

### List directory contents

```bash
ls
ls -F
ls -l
ls -la
```

- `ls` lists items in the current directory.
- `-F` marks directory names with `/`.
- `-l` shows a detailed list.
- `-a` includes hidden files.

Together, these options help you answer two quick questions before changing anything: "Where am I?" and "What is here?"

### Change directory

```bash
cd some_directory
cd ..
cd
```

- `cd some_directory` enters a directory.
- `cd ..` moves one level up.
- `cd` without arguments returns to your home directory.

A useful mental model is a tree of folders. `cd` moves your location in that tree, and every relative path depends on your current location.

Paths can be:

- Absolute: start from `/`, such as `/home/username/data`.
- Relative: start from your current location, such as `data/file.txt`.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1: Basic navigation

1. Print your current directory.
2. List all files, including hidden ones.
3. Move into a directory of your choice.
4. Move back up one level.
5. Return to your home directory.

::::::::::::::: solution

```bash
pwd
ls -la
cd some_directory
cd ..
cd
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Working with files and directories

In command-line workflows, file and directory operations are the foundation for reproducible analysis. Before running scripts or notebooks, you usually create a clean folder structure and move files into predictable locations.

### Create directories and files

```bash
mkdir my_project
cd my_project
touch file1.txt file2.txt
ls
```

`mkdir` creates a new directory.
`touch` creates an empty file if it does not exist, and updates a timestamp if it does.

In the example above, we created a new directory called `my_project`, entered it, and created two empty text files. The output of `ls` confirms the files exist.

### Copy, move, and rename

```bash
cp file1.txt file1_copy.txt
mv file1_copy.txt renamed.txt
mv renamed.txt ../renamed.txt
```

Use the `cp` command to copy files (it will create a duplicate).
Use the `mv` command to move or rename files.

If you try `ls` after the last command, you will see that `renamed.txt` is no longer in the current directory because it was moved to the parent directory.

### Remove files and directories

```bash
rm file2.txt
rm -r some_directory
```
The `rm` command removes files. With the `-r` option, it removes directories and their contents recursively.

Be careful with `rm` and especially `rm -r`, since removal is usually permanent.
For beginners, a safe habit is to run `ls` before and after removal so you can verify exactly what changed.

## Viewing and editing text files

### Write text to files with echo, >, and >>

You can use `echo` together with redirection operators to create and update text files directly from the shell.

```bash
echo "Sea surface temperature notes" > notes.txt
echo "Added a second line" >> notes.txt
cat notes.txt
```

How this works:

- `>` writes output to a file (and overwrites the file if it already exists).
- `>>` appends output to the end of a file (keeps existing content).

This pattern is very common in command-line workflows and is used again in the Git lesson when creating and updating tracked files.

### View file contents with cat

Use `cat` to print a text file in the terminal:

```bash
cat data1.txt
cat data1.txt data_main.txt
```

This is useful for quickly checking file contents.
For very large files, tools like `less` are often more practical than `cat`, but `cat` is perfect for small text files:

```bash
less data1.txt
```

### Edit files with nano

`nano` is a simple terminal text editor:

```bash
nano notes.txt
```

In `nano`, type your text, then:

- Press `Ctrl+O` to save.
- Press Enter to confirm the filename.
- Press `Ctrl+X` to exit.

`nano` is a good starter editor because it shows keyboard shortcuts directly in the interface.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2: cat and nano basics

1. Create a file called `notes.txt` using `nano`.
2. Add two short lines of text and save the file.
3. Display the file contents using `cat`.

::::::::::::::: solution

```bash
nano notes.txt
cat notes.txt
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 3: echo and redirection

1. Create a file called `log.txt` with one line using `echo` and `>`.
2. Add a second line using `echo` and `>>`.
3. Display the final file content with `cat`.

::::::::::::::: solution

```bash
echo "First log line" > log.txt
echo "Second log line" >> log.txt
cat log.txt
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Getting help and using tab completion

Use built-in help:

```bash
ls --help
man ls
```

Use tab completion by typing part of a filename or directory name, then pressing Tab.
This reduces typos and speeds up command entry.

When learning new commands, combine these habits:

- Check options with `--help`.
- Try commands in a small practice directory first.
- Use tab completion to avoid path mistakes.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 4: File operations practice

1. Create a directory called `shell_practice`.
2. Enter it and create `data1.txt` and `data2.txt`.
3. Create a subdirectory called `backup`.
4. Copy `data1.txt` into `backup`.
5. Rename `data2.txt` to `data_main.txt`.
6. List the contents of the main directory and `backup`.
7. Remove `data1.txt` from the main directory.

::::::::::::::: solution

```bash
mkdir shell_practice
cd shell_practice
touch data1.txt data2.txt
mkdir backup
cp data1.txt backup/
mv data2.txt data_main.txt
ls
ls backup
rm data1.txt
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Summary

This episode introduced the shell as a practical interface for navigating files, running programs, and performing common file operations.
These skills are foundational for later lessons where we work with scripts, remote systems, and larger scientific datasets.

:::::::::::::::::::::::::::::::::::::::::: keypoints

- "The shell is a powerful interface for scientific workflows and automation."
- "`pwd`, `ls`, and `cd` are core commands for navigation."
- "`mkdir`, `touch`, `cp`, `mv`, and `rm` cover most basic file operations."
- "`echo` with `>` and `>>` lets you create files and append text from the command line."
- "`cat` displays text file contents quickly in the terminal."
- "`nano` is a beginner-friendly terminal editor for creating and updating text files."
- "Use `--help`, `man`, and tab completion to work faster and more safely."

::::::::::::::::::::::::::::::::::::::::::::::::::
