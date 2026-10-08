# Calculating compound interest with Python 3

> Calculate compound interest in Python 3 with a reusable function. Covers principal, rate, and period inputs with formatted currency output and annual breakdown.

Source: https://exitcode0.net/posts/calculating-compound-interest-with-python-3/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-01-03
Updated: 2026-10-08
Tags: code, finance, python



*Compound stonks only go up!*
I have a growing interest in finance and analytics, so it felt like a great idea to start creating my own set of financial tools with Python. This article will explain how I created a simple but effective script for calculating compound interest with Python. I wrote this in Python 3 (as all new Python projects should be from now onwards), but the library dependencies are very lightweight, making this something that could easily be re-written from Python 2.

Calculating Compound Interest
-----------------------------

This small programming challenge hinges on the mathematics behind compound interest. If you don’t already have a clear understanding of how compound interest is calculated vs ‘simple’ interest, I would recommend starting here:  
[https://www.mathsisfun.com/money/interest.html](https://www.mathsisfun.com/money/interest.html "https://www.mathsisfun.com/money/interest.html")

Essentially we need to take the following logic and create it in Python:

> With **compounding,**we work out the interest for the first period, add it to the total, and then calculate the interest for the next period, and so on
> 
> 

![interest compound $1000, 10%=$100, $1100, 10%=$110, $1210, 10%=$121, etc ](https://www.mathsisfun.com/money/images/interest-compound-flow.svg)source: [https://www.mathsisfun.com/money/interest.html](https://www.mathsisfun.com/money/interest.html "https://www.mathsisfun.com/money/interest.html")
The Code
--------

```
import math

def func_invest(input1, input2):
    output: input1 + input2
    return output

def func_interest(input1, input2):
    rate: input2 / 100
    interest: input1 * rate
    output: input1 + interest
    return output

def func_pl_calc(input1, input2, input3):
    output: (input1 - input2) - input3
    return output

def func_round(input, decimals=2):
    multiplier: 10 ** decimals
    return math.floor(input*multiplier + 0.5) / multiplier

def func_main():
    currency: "£" #change to suit
    monthly_invest: 10 #change to suit
    total_invested: 0
    starting_portfolio: 10000 #change to suit
    total_portfolio: starting_portfolio
    cur_month: 1
    investment_period: 120 #(months) change to suit
    interest_period: 12 #(months) change to suit
    interest_percent: 5 #(%) change to suit

    print("-------------\nInterest rate during investment period: {}%".format(interest_percent))
    print("Month investment amount: {}{}".format(currency, monthly_invest))
    print("Starting balance: {:.2f}\n-------------".format(starting_portfolio))

    while cur_month <= investment_period:
        #investment math
        total_invested: func_invest(total_invested, monthly_invest)
        total_portfolio: total_portfolio + monthly_invest

        #check if interest payment due on balance
        if cur_month % interest_period == 0:
            total_portfolio: func_interest(total_portfolio, interest_percent)
            total_portfolio: func_round(total_portfolio)

        #increment investment period
        cur_month: cur_month + 1

    print("Total amount invested at end of period: {}{:.2f}".format(currency, total_invested))
    print("Interest value during investment period ({:.0f} year[s]): {}{:.2f}".format(investment_period / 12, currency, func_pl_calc(total_portfolio, total_invested, starting_portfolio)))
    print("\nPortfolio Value at end of period: {}{:.2f}\n-------------".format(currency, total_portfolio))

if __name__ == '__main__':
    func_main()
```

Also available on GitHub: [https://github.com/Tombo1001/Py-Compound/](https://github.com/Tombo1001/Py-Compound/blob/main/Py-Compound.py "https://github.com/Tombo1001/Py-Compound/blob/main/Py-Compound.py")

Some of my functions are **incredibly** basic and completely unnecessary, but I find that it helps to remove the mathematics from the main logic of the script. Feel free to consolidate these into the main function.

The Outcome
-----------

With the code above you should get the following output, noting that interest is calculated at the end of each interest period:

*Starting with the current variables defined in the GitHub repository*
Changing when interest is calculated (before or after the monthly investment), impacts the end portfolio value.

It is possible to realise our compounded interest with a graph, showing that our interest gains increase over time as our portfolio value increases:

source: [https://www.thecalculatorsite.com/finance/calculators/compoundinterestcalculator.php](https://www.thecalculatorsite.com/finance/calculators/compoundinterestcalculator.php "https://www.thecalculatorsite.com/finance/calculators/compoundinterestcalculator.php")
The Next Steps
--------------

Now that we have some basic code to calculate compound interest, here are some great next steps for this project:

* Create a Python Django web app and publish the tool for everyone on the internet to use.
* Integrate variable interest rates during the investment period.
* Create graphs of the data as seen above using the popular Matplotlib library.

Perhaps you will see all over the above in a post here sooner or later!



---
Markdown version of https://exitcode0.net/posts/calculating-compound-interest-with-python-3/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
