# Python 3 SSH with Paramiko

> Run remote SSH commands from Python 3 with Paramiko. Connect with a password or key, execute commands and capture stdout and stderr.

Source: https://exitcode0.net/posts/python-3-ssh-with-paramiko/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-05-15
Updated: 2026-10-08
Tags: code, paramiko, python, ssh



I have a number of projects which I am currently working on which usually involve a raspberry pi and so other TCP enabled object. One project, in particular, is to control a KVM server with buttons connected to a raspberry pi (follow on social media or RSS for that future post). I have been using Python 3 SSH with Paramiko to send commands KVM server from the pi. Here is how I do that…

What is Paramiko?
-----------------

*Paramiko is a Python (2.7, 3.4+) implementation of the SSHv2 protocol, providing both client and server functionality. While it leverages a Python C extension for low level cryptography, Paramiko itself is a pure Python interface around SSH networking concepts.* – [http://www.paramiko.org/](http://www.paramiko.org/ "http://www.paramiko.org/")

**TLDR – Paramiko is a python library which lets you do SSH things.**

The how-to: Python 3 SSH with Paramiko
--------------------------------------

For the purpose of this blog, I will be focusing on using paramiko to make client connections. Let’s include this library and dig into sending some dangerous commands to Linux boxes.

```
import paramiko
```

I have been using a function to complete my SSH command and passing my SSH variables when calling that function. Lets start with that funciton:

```
def func_do_ssh_Stuff(address, usr, pwd, command):
        try:
            print("ssh " + usr + "@" + address + ", running : " +
                         command)
            client: paramiko.SSHClient()
            client.load_system_host_keys() # this loads any local ssh keys
            client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

            client.connect(address, username=usr, password=pwd)
            _, ss_stdout, ss_stderr: client.exec_command(command)
            r_out, r_err: ss_stdout.readlines(), ss_stderr.read()
            print(r_err)
            if len(r_err) > 5:
                print(r_err)
            else:
                print(r_out)
            client.close()
        except IOError:
            print(".. host " + address + " is not up")
            return "host not up", "host not up"
```

**What’s going on here?** The function is expecting to be handed a username, password (or none if you plan to use key based authentication (recommended)), host address and the shell command to run once the connection is established.

Paramiko checks for local keys on the system which it can use when connecting to the host, then attempts a connection. Once the connection is established the command we passed to the function will be executed.

Finally, we print the output or error returned by the execution of that command. This is all encased in a ‘try, catch’ which will inform us if the SSH target is offline.

### Calling the Paramiko SSH function

Now that we have our SSH function declared, we can call it:

```
func_do_ssh_Stuff("192.168.0.33", "root", "", "echo Hello World!")
```

**Note:** you will need to change the IP to suit and add a password (“”) if required. 

I would highly recommend using key-based authentication on this one, otherwise, it will require you to hard code SSH passwords into your scripts… **not a great idea**.

And now that you can SSH from a Pi (or anything else which runs python), the world is your oyster!

---

More wonderful Python things:
-----------------------------

* How To: Python infinite loops with while true – [https://exitcode0.net/posts/how-to-python-infinite-loops-with-while-true/](https://exitcode0.net/posts/how-to-python-infinite-loops-with-while-true/ "https://exitcode0.net/posts/how-to-python-infinite-loops-with-while-true/")
* Kasa home automation with IFTTT and python webooks – [https://exitcode0.net/kasa-home-automation-with-ifttt-and-python-webooks/](https://exitcode0.net/posts/kasa-home-automation-with-ifttt-and-python-webhooks/)
* Getting started with Python 3 – a beginner’s cheat sheet – [https://exitcode0.net/getting-started-with-python-3-a-beginners-cheat-sheet/](https://exitcode0.net/posts/getting-started-with-python-3-a-beginners-cheat-sheet/)



---
Markdown version of https://exitcode0.net/posts/python-3-ssh-with-paramiko/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
