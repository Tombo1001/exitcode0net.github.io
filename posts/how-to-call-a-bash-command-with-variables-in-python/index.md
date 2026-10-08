# How To call a bash command with variables in Python

> Call a bash command with variables from Python 3 using subprocess.run: list arguments, captured output, exit codes and when shell=True is safe. Tested on 3.14.

Source: https://exitcode0.net/posts/how-to-call-a-bash-command-with-variables-in-python/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-12-28
Updated: 2026-10-08
Tags: bash, how-to, linux, python

> **Updated October 2026:** the code in my 2020 version of this post didn't run. It used `:` where it needed `=`, wrote `Import` with a capital, and passed a command string to `Popen` without a shell, which raises `FileNotFoundError`. I've rewritten the examples, run every one on Python 3.14.4, and added the part I should have warned you about, which is shell injection.

A quick Google of the title only gave me answers for the inverse, calling a Python script from bash and handing it variables. At least the first three results did, and I'm probably not alone in scrolling past the Stack Overflow answers.

So I went it alone. The way I first worked it out was a total hack, but it worked. There's an xkcd about finding your exact question and no answer: [xkcd.com/979](https://xkcd.com/979/).

## Quick answer

Pass the program and each variable as separate items in a list to `subprocess.run`:

```python
import subprocess

my_variable_1 = "Goodbye"
my_variable_2 = "Jupiter"

result = subprocess.run(["echo", my_variable_1, my_variable_2])
print(result.returncode)
```

```text
Goodbye Jupiter
0
```

No string building, no quoting, and the variables can contain spaces or semicolons without anything odd happening. That's the whole answer. The rest of this post is why, and what to do when you need a pipe.

## Background

Some context so my variable names make sense. I was building a tool to turn a folder of music into easy listening playlist videos with ffmpeg. I'd given up on the Python ffmpeg libraries, so I wanted to call ffmpeg as a bash command and write everything else in Python. Calling a bash command from Python is easy. Passing variables into it is the less easy bit.

## The code

I use `subprocess` rather than the classic `os.system`, because `subprocess` gives you the exit code, the output and the error text as separate things. It's in the standard library, so there's nothing to install.

For the ffmpeg job the command looks like this, with the file names as variables:

```python
import shlex
import subprocess

video = "/tmp/in.mp4"
output = "/tmp/out.m4a"

command = ["ffmpeg", "-i", video, "-vn", "-c:a", "copy", output]
print(shlex.join(command))   # optional: shows the command as a shell would see it
result = subprocess.run(command)
```

```text
ffmpeg -i /tmp/in.mp4 -vn -c:a copy /tmp/out.m4a
```

I haven't run that ffmpeg command on the machine I tested this on, since it doesn't have ffmpeg installed. The pattern is the same as the `echo` one, only with more list items.

### Checking whether it worked

`run` returns a `CompletedProcess`. Its `returncode` is the exit code, and 0 means success:

```python
if result.returncode == 0:
    print("we did it, exitcode0 (.net)")
else:
    print("check your command")
    exit(1)
```

Or let Python do the checking. `check=True` raises `CalledProcessError` on any non-zero exit code, and `capture_output=True, text=True` hands you the error text as a string:

```python
try:
    subprocess.run(["ls", "/does/not/exist"], capture_output=True, text=True, check=True)
except subprocess.CalledProcessError as error:
    print("returncode:", error.returncode)
    print("stderr:", error.stderr.strip())
```

```text
returncode: 2
stderr: ls: cannot access '/does/not/exist': No such file or directory
```

If the program itself isn't installed you get a different error, `FileNotFoundError`, before any exit code exists:

```text
FileNotFoundError: [Errno 2] No such file or directory: 'not-a-real-program'
```

### Getting the output back into a variable

Add `capture_output=True` and `text=True`, then read `stdout`:

```python
result = subprocess.run(["ls", "-la", "/tmp/pysub"], capture_output=True, text=True, check=True)
print(result.stdout.splitlines()[0])
```

```text
total 4
```

That replaces the `Popen` and `stdout.read()` pair from my original post. `Popen` is still useful, for streaming output while the command runs:

```python
with subprocess.Popen(["seq", "3"], stdout=subprocess.PIPE, text=True) as process:
    for line in process.stdout:
        print("line:", line.strip())
```

```text
line: 1
line: 2
line: 3
```

## The hack from 2020, and why it's a bad idea

My original approach built the whole command as one string and handed it to a shell:

```python
command = "echo {} {}".format(my_variable_1, my_variable_2)
process = subprocess.call(command, shell=True)
```

It works, and it printed `Goodbye Jupiter` and returned 0. It's also the way to get a shell injection. Watch what happens when one of the variables is not what you expected:

```python
name = "x; echo INJECTED"
subprocess.run(f"echo hello {name}", shell=True)
```

```text
hello x
INJECTED
```

The shell saw a semicolon and ran a second command. With the list form there's no shell to interpret anything, so the same value is just text:

```python
subprocess.run(["echo", "hello", name])
```

```text
hello x; echo INJECTED
```

That matters the moment a variable comes from a file name, a web request or anything you didn't type yourself. If you really do need a shell string, wrap each variable in `shlex.quote`:

```python
import shlex
subprocess.run(f"echo hello {shlex.quote(name)}", shell=True)
```

```text
hello x; echo INJECTED
```

The other thing that went wrong in my 2020 post was this, which people copy all the time:

```python
subprocess.Popen("ls -la /tmp", stdout=subprocess.PIPE)
```

```text
FileNotFoundError: [Errno 2] No such file or directory: 'ls -la /tmp'
```

Without `shell=True`, Python looks for a program literally called `ls -la /tmp`. Split it into a list and it works.

## Pipes and redirects

A pipe is the one thing that genuinely needs a shell, because `|` is shell syntax:

```python
result = subprocess.run("ls -1 /tmp/pysub | grep examples", shell=True, capture_output=True, text=True)
print(result.stdout.strip())
```

```text
examples.py
```

Only use this with strings you wrote yourself. Better, do the second half in Python and skip the shell entirely:

```python
ls = subprocess.run(["ls", "-1", "/tmp/pysub"], capture_output=True, text=True, check=True)
print([line for line in ls.stdout.splitlines() if "examples" in line])
```

```text
['examples.py']
```

## Troubleshooting

### FileNotFoundError and the command exists

You passed the whole command as one string without `shell=True`. Split it into a list.

### My variable has spaces in it and the command breaks

You're building a string. Use the list form and the spaces stop mattering.

### Nothing is printed

`run` doesn't show the child's output in your variable unless you add `capture_output=True`. Without it the output goes straight to your terminal.

### Output comes back as bytes

Add `text=True`.

## The version I'd trust

The 2020 hack was fine for a script only I ran, with file names I'd typed myself. The list form is the first version I'd trust with a name I didn't choose, and it's shorter.

## More Python snippets

- [Kasa home automation with IFTTT and Python webhooks](https://exitcode0.net/posts/kasa-home-automation-with-ifttt-and-python-webhooks/)
- [Python 3 SSH with Paramiko](https://exitcode0.net/posts/python-3-ssh-with-paramiko/)

## Or just hand this page to your agent

If you've got a script full of `os.system` and `shell=True` calls, paste the following into your coding agent:

```text
Read https://exitcode0.net/posts/how-to-call-a-bash-command-with-variables-in-python/ and convert the shell calls in my Python script to subprocess.run with list arguments. Before editing, list every os.system, subprocess.call, Popen and shell=True call in the file and tell me which ones use variables that could come from outside the script. Leave any call that needs a pipe or a redirect as it is and ask me about it. Show me the diff and don't run any of the commands.
```

The page gives the agent the list-argument pattern, how to check exit codes, and when `shell=True` is unavoidable. Only you know which variables are trusted and which commands are safe to run.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/how-to-call-a-bash-command-with-variables-in-python/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
