# Bash Scripting
Bash Scripting is a powerful part of system administration and development used at an extreme level. It is used by the System Administrators, Network Engineers, Developers, Scientists, and everyone who use Linux/Unix operating system. They use Bash for system administration, data crunching, web application deployment, automated backups, creating custom scripts for various pages, etc.

## Script

In Computer programming, a script is a set of commands for an appropriate run time environment which is used to automate the execution of tasks.

## Bash Script:
A Bash Shell Script is a plain text file containing a set of various commands that we usually type in the command line. It is used to automate repetitive tasks on Linux filesystem. It might include a set of commands, or a single command, or it might contain the hallmarks of imperative programming like loops, functions, conditional constructs, etc. Effectively, a Bash script is a computer program written in the Bash programming language.

## How to create and run a Bash Script?
To create an empty bash script, first, change the directory in which you want to save your script using cd command. Try to use text editor like gedit in which you want to type the shell commands.

- Use touch command to create the zero bytes sized script.

        touch file_name  

- To open the script in the text editor (eg., gedit), type

        gedit file_name.sh  

Here, .sh is suffixed as an extension that you have to provide for execution.

Type the shell commands for your bash script in the newly opened text window or the text editor. Before typing bash shell commands, first, look at the base of any bash script.

Each Bash based Linux script starts by the line-

        #! /bin/bash  

Where #! is referred to as the shebang and rest of the line is the path to the interpreter specifying the location of bash shell in our operating system.

Bash use # to comment any line.

Bash use echo command to print the output.

At the end, execute the bash script prefixing with ./.

Have a look at the basic terms of a Bash Script, i.e., SheBang and echo command.

## SheBang (#!)
The She Bang (#!) is a character sequence consisting of the characters number sign (#) and exclamation mark (!) at the beginning of a script.


Under the Unix-like operating systems, when a script with a shebang runs as a program, the program loader parses the rest of the lines with the first line as an interpreter directive. So, SheBang denotes an interpreter to execute the script lines, and it is known as the path directive for the execution of different kinds of Scripts like Bash, Python, etc.

Here is the correct SheBang format for the discussed Bash Script.

            #!/bin/bash  

The formatting for shebang is most important. Its incorrect format can cause improper working of commands. So, always remember these two points of SheBang formatting while creating a Script as follows:

It should always be on the very first line of the Script.
There should not be any space before the hash (#), between the hash exclamation marks (#!), and the path to the interpreter.

## echo
echo is a built-in command in Bash, which is used to display the standard output by passing the arguments. It is the most widely used command for printing the lines of text/String to the screen. Its performance is the same on both the platforms: Bash Shell and Command Line Terminal.

Syntax:

```
echo [option] [string]  
echo [string]  
```
### Practice Test

1. Write a Shell Script to take backup of the database and upload it to a private S3 bucket. 

2. Execute this script at 3 AM every Weekday using cron(which needs to be set up in /etc/cron.d directory and execution logs redirected to /var/log/db-backup.log ).

3. Move the content from S3 standard to S3-IA after 3 hrs.

### For more information refer the following 

[Bash Script for beginners](https://www.freecodecamp.org/news/shell-scripting-crash-course-how-to-write-bash-scripts-in-linux/)