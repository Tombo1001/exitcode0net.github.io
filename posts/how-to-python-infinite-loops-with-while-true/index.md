# How To: Python infinite loops with while true

> Write Python infinite loops with while True for polling and monitoring scripts, handle KeyboardInterrupt for a clean exit and add sleep intervals.

Source: https://exitcode0.net/posts/how-to-python-infinite-loops-with-while-true/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-05-05
Updated: 2026-10-08
Tags: code, python



Here is a quick guide on how to create an infinite loop in python using a ‘while true’ statement. There are number of reason that you might want to implement this; a great use case would be outputting a fluctuating variable to the terminal such as a temperature reading from a sensor.

*loops that make your brain hurt*
A ‘while true’ statement allows us to run a sequence of code until a particular condition is met. In order to make that sequence of code run in an infinite loop, we can set the condition to be one that is impossible to reach. Better still, we can simply omit the condition altogether to ensure that the while true loop never ends.

```
while True:
        print("The current time is: %s" % strTimeNow)
        time.sleep(5)
```

In cases where it would be useful to exit that loop if a given condition is met or exception is reached, we can encase our ‘while true’ statement with a ‘try except’ statement.

```
try:
    while True:
        print("The current time is: %s" % strTimeNow)
        time.sleep(5)
except KeyboardInterrupt:
    print("Hard Exit Initiated. Goodbye!")
```

There are a number of exception that can be handled by a try statement, I have chose to use the KeyboardInterupt (Ctrl+C) clause in this example.

You can learn more about exception handling here: [https://docs.python.org/3/tutorial/errors.html#handling-exceptions](https://docs.python.org/3/tutorial/errors.html#handling-exceptions "https://docs.python.org/3/tutorial/errors.html#handling-exceptions")

---

More Python Tips and tricks:
----------------------------

* More Python code samples and examples – https://exitcode0.net/code-samples/#python
* Kasa home automation with IFTTT and python webooks – [https://exitcode0.net/kasa-home-automation-with-ifttt-and-python-webooks/](https://exitcode0.net/posts/kasa-home-automation-with-ifttt-and-python-webhooks/)
* Getting started with Python 3 – a beginner’s cheat sheet – [https://exitcode0.net/getting-started-with-python-3-a-beginners-cheat-sheet/](https://exitcode0.net/posts/getting-started-with-python-3-a-beginners-cheat-sheet/)



---
Markdown version of https://exitcode0.net/posts/how-to-python-infinite-loops-with-while-true/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
