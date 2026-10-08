# How To call a bash command with variables in Python

> Call bash commands with variable arguments from Python 3 using subprocess, pass dynamic values to the shell and capture stdout for automation scripts.

Source: https://exitcode0.net/posts/how-to-call-a-bash-command-with-variables-in-python/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-12-28
Updated: 2026-10-08
Tags: bash, how-to, linux, python



A quick Google of the above only seemed to give me answers for the inverse, calling a python script from a bash shell and handing it variables. At least, the first 3 results showed this and I’m probably not alone when it comes to scrolling down past the StackOverflow articles.

So I am had to go it alone and the way that I figured out how to call a bash command with variables in Python (3), is a total hack… But it works. This is commonly referred to as ‘getting the job done’ code.

[*](https://xkcd.com/979/ "https://xkcd.com/979/")https://xkcd.com/979/*
Background
----------

Here is some quick background info about what I am trying to do, to help you understand some of my variable names and the particular bash commands I am using.

I am currently playing around/working on a tool to build easy listening playlist videos with ffmpeg. For millions of reasons, but specifically one (I gave up on figuring out the python library), I want to use ffmpeg in bash, but write everything else my script does is python. Therefore, I need a way to call bash commands from python: easy; and pass variables to that bash command: less easy.

*Maxiumum effort photoshop*
The Code
--------

This is all based on using the ‘Subprocess’ library rather than just the classic ‘os.system’ method. Subprocess lets us do some cool things – technical things – so we must first go ahead and import it (install it with pip3 if required):

```
Import subprocess
```

Next, I will define my command which I want to send over to bash for execution, and the variables I want to pass to it (not in that order):

```
my_variable_1: "Goodbye"  
my_variable_2: "Jupiter"  
command: ("echo {}".format(my_variable1, my_variable_2)
```

At this point we can choose to print out the command to the python console to check our syntax:

```
print(command) # optional
```

Now we can send our command to the bash shell and await it’s completion.

```
process: subprocess.call(command, shell=True)
```

Upon execution of the command, the process variable can be used to check the status of our efforts in bash, allowing us to form a condition for proceeding or returning if we are inside a function:

```
if process == 0:  
   print("we did it, exitcode0 (.net)")  
   #return  
else:  
   print("check your command syntax")  
   exit(1)
```

The key to passing variables revolves around building our command before calling it with subprocess: “*command: (“echo {}”.format(my\_variable1, my\_variable\_2)*“

With the code above we are currently only checking the status code of the command which we have run. However, it is possible to capture the standard output of the command and bring that back into python for further programming; to do this we need to use subprocess.popen:

```
command: "ls -la /home/foo"
```

```
process: subprocess.Popen(command, stdout=subprocess.PIPE)
```

```
output: process.stdout.read()
```

If we were to print ‘*output*‘ we should hope to see the standard output of ‘*ls -la*‘ in the directory ‘*/home/foo*‘. You could take things further in bash by piping the output to grep before reading it back in with python; an alternative to further string processing in Python.

---

More Python snippets:
---------------------

* KASA HOME AUTOMATION WITH IFTTT AND PYTHON WEBHOOKS – [https://exitcode0.net/kasa-home-automation-with-ifttt-and-python-webhooks/](https://exitcode0.net/posts/kasa-home-automation-with-ifttt-and-python-webhooks/)
* PYTHON 3 SSH WITH PARAMIKO – [https://exitcode0.net/posts/python-3-ssh-with-paramiko/](https://exitcode0.net/posts/python-3-ssh-with-paramiko/ "https://exitcode0.net/posts/python-3-ssh-with-paramiko/")



---
Markdown version of https://exitcode0.net/posts/how-to-call-a-bash-command-with-variables-in-python/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
