# Getting started with Python 3 - a beginner's cheat sheet

> Python 3 beginner cheat sheet covering variables, loops, functions and string handling. A quick reference with short, runnable examples.

Source: https://exitcode0.net/posts/getting-started-with-python-3-a-beginners-cheat-sheet/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-11-23
Updated: 2026-10-08
Tags: code, python



[*](https://www.python.org/ "https://www.python.org/")Python.Org*
I have been getting started with python 3 – I want to make this my primary scripting language. One way I like to assist myself whilst I learn the rope is to maintain a crib sheet filled with all the trivial things I would otherwise forget.

Python 3 Cheat Sheet
--------------------

```

#Define Variables
programming_languages: "Python", "VB", "C++", "C#"

#Print Variables
print(programming_languages)

print('--------------------')

#Basic for loop + variables
for language in programming_languages:
    print(language)

print('--------------------')

#Basic Function
def FuncExample():
    i: 1
    for language in programming_languages:
        #Concatinate Strings and Integers in print statements
        print("Language " + str(i) + ":" + language)
        #Increment Integer
        i += 1

FuncExample()

print('--------------------')

#Functions with Variables #1 - Strings
def FuncVarExample1(fname, lname):
    #Print with CRLF
    print("First Name: " + fname + "\r\n" + "Last Name: " + lname)

FuncVarExample1("Joe","Bloggs")

print('--------------------')

#Functions with Variables #2 - Integers + Returning Values
def FuncVarExample2(x, y):
    #Basic integer maths
    return x+y

#Concatenating Strings and Integers
print("33 + 42: " + str(FuncVarExample2(33,42)))

print('--------------------')
```

You can also find more code snippets here: https://exitcode0.net/code-samples/



---
Markdown version of https://exitcode0.net/posts/getting-started-with-python-3-a-beginners-cheat-sheet/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
