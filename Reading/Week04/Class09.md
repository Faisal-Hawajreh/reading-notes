# What Are Dunder Methods?
In Python, special methods are a set of predefined methods you can use to enrich your classes. They are easy to recognize because they start and end with double underscores, for example __init__ or __str__.

As it quickly became tiresome to say under-under-method-under-under Pythonistas adopted the term “dunder methods”, a short form of “double under.”

These “dunders” or “special methods” in Python are also sometimes called “magic methods.” But using this terminology can make them seem more complicated than they really are—at the end of the day there’s nothing “magical” about them. You should treat these methods like a normal language feature.

Dunder methods let you emulate the behavior of built-in types. For example, to get the length of a string you can call len('string'). But an empty class definition doesn’t support this behavior out of the box:
```
class NoLenSupport:
    pass

>>> obj = NoLenSupport()
>>> len(obj)
TypeError: "object of type 'NoLenSupport' has no len()"
```
To fix this, you can add a __len__ dunder method to your class:
```
class LenSupport:
    def __len__(self):
        return 42

>>> obj = LenSupport()
>>> len(obj)
42
```
Another example is slicing. You can implement a __getitem__ method which allows you to use Python’s list slicing syntax: obj[start:stop].
## Special Methods and the Python Data Model
This elegant design is known as the Python data model and lets developers tap into rich language features like sequences, iteration, operator overloading, attribute access, etc.

You can see Python’s data model as a powerful API you can interface with by implementing one or more dunder methods. If you want to write more Pythonic code, knowing how and when to use dunder methods is an important step.

For a beginner this might be slightly overwhelming at first though. No worries, in this article I will guide you through the use of dunder methods using a simple Account class as an example.


## Enriching a Simple Account Class
Throughout this article I will enrich a simple Python class with various dunder methods to unlock the following language features:

- Initialization of new objects
- Object representation
- Enable iteration
- Operator overloading (comparison)
- Operator overloading (addition)
- Method invocation
- Context manager support (with statement)

### Object Initialization: __init__
Right upon starting my class I already need a special method. To construct account objects from the Account class I need a constructor which in Python is the __init__ dunder:
```
class Account:
    """A simple account class"""

    def __init__(self, owner, amount=0):
        """
        This is the constructor that lets us create
        objects from this class
        """
        self.owner = owner
        self.amount = amount
        self._transactions = []
```
The constructor takes care of setting up the object. In this case it receives the owner name, an optional start amount and defines an internal transactions list to keep track of deposits and withdrawals.

This allows us to create new accounts like this:
```
>>> acc = Account('bob')  # default amount = 0
>>> acc = Account('bob', 10)
```
### Object Representation: __str__, __repr__
It’s common practice in Python to provide a string representation of your object for the consumer of your class (a bit like API documentation.) There are two ways to do this using dunder methods:

1. __repr__: The “official” string representation of an object. This is how you would make an object of the class. The goal of __repr__ is to be unambiguous.
2. __str__: The “informal” or nicely printable string representation of an object. This is for the enduser.

Let’s implement these two methods on the Account class:
```
class Account:
    # ... (see above)

    def __repr__(self):
        return 'Account({!r}, {!r})'.format(self.owner, self.amount)

    def __str__(self):
        return 'Account of {} with starting amount: {}'.format(
            self.owner, self.amount)
```
If you don’t want to hardcode "Account" as the name for the class you can also use self.__class__.__name__ to access it programmatically.

If you wanted to implement just one of these to-string methods on a Python class, make sure it’s __repr__.

Now I can query the object in various ways and always get a nice string representation:
```
>>> str(acc)
'Account of bob with starting amount: 10'

>>> print(acc)
"Account of bob with starting amount: 10"

>>> repr(acc)
"Account('bob', 10)"
```
### Iteration: __len__, __getitem__, __reversed__
In order to iterate over our account object I need to add some transactions. So first, I’ll define a simple method to add transactions. I’ll keep it simple because this is just setup code to explain dunder methods, and not a production-ready accounting system:
```
def add_transaction(self, amount):
    if not isinstance(amount, int):
        raise ValueError('please use int for amount')
    self._transactions.append(amount)
```
I also defined a property to calculate the balance on the account so I can conveniently access it with account.balance. This method takes the start amount and adds a sum of all the transactions:
```
@property
def balance(self):
    return self.amount + sum(self._transactions)
```
Let’s do some deposits and withdrawals on the account:
```
>>> acc = Account('bob', 10)

>>> acc.add_transaction(20)
>>> acc.add_transaction(-10)
>>> acc.add_transaction(50)
>>> acc.add_transaction(-20)
>>> acc.add_transaction(30)

>>> acc.balance
80
```
Now I have some data and I want to know:

1. How many transactions were there?
2. Index the account object to get transaction number …
3. Loop over the transactions

With the class definition I have this is currently not possible. All of the following statements raise TypeError exceptions:
```
>>> len(acc)
TypeError

>>> for t in acc:
...    print(t)
TypeError

>>> acc[1]
TypeError
```
Dunder methods to the rescue! It only takes a little bit of code to make the class iterable:
```
class Account:
    # ... (see above)

    def __len__(self):
        return len(self._transactions)

    def __getitem__(self, position):
        return self._transactions[position]
```
Now the previous statements work:
```
>>> len(acc)
5

>>> for t in acc:
...    print(t)
20
-10
50
-20
30

>>> acc[1]
-10
```
To iterate over transactions in reversed order you can implement the __reversed__ special method:
```
def __reversed__(self):
    return self[::-1]

>>> list(reversed(acc))
[30, -20, 50, -10, 20]
```
To reverse the list of transactions I used Python’s reverse list slice syntax. I also had to wrapp the result of reversed(acc) in a list() call because reversed() returns a a reverse iterator, not a list object we can print nicely in the REPL. Check out this tutorial on iterators in Python if you’d like to learn more about how this approach works in detail.

All in all, this account class is starting to look quite Pythonic to me now.

### Operator Overloading for Comparing Accounts: __eq__, __lt__
We all write dozens of statements daily to compare Python objects:
```
>>> 2 > 1
True

>>> 'a' > 'b'
False
```
This feels completely natural, but it’s actually quite amazing what happens behind the scenes here. Why does > work equally well on integers, strings and other objects (as long as they are the same type)? This polymorphic behavior is possible because these objects implement one or more comparison dunder methods.

An easy way to verify this is to use the dir() builtin:
```
>>> dir('a')
['__add__',
...
'__eq__',    <---------------
'__format__',
'__ge__',    <---------------
'__getattribute__',
'__getitem__',
'__getnewargs__',
'__gt__',    <---------------
...]
```
Let’s build a second account object and compare it to the first one (I am adding a couple of transactions for later use):
```
>>> acc2 = Account('tim', 100)
>>> acc2.add_transaction(20)
>>> acc2.add_transaction(40)
>>> acc2.balance
160

>>> acc2 > acc
TypeError:
"'>' not supported between instances of 'Account' and 'Account'"
```
What happened here? We got a TypeError because I have not implemented any comparison dunders nor inherited them from a parent class.

Let’s add them. To not have to implement all of the comparison dunder methods, I use the functools.total_ordering decorator which allows me to take a shortcut, only implementing __eq__ and __lt__:
```
from functools import total_ordering

@total_ordering
class Account:
    # ... (see above)

    def __eq__(self, other):
        return self.balance == other.balance

    def __lt__(self, other):
        return self.balance < other.balance
```
And now I can compare Account instances no problem:
```
>>> acc2 > acc
True

>>> acc2 < acc
False

>>> acc == acc2
False
```
### Operator Overloading for Merging Accounts: __add__
In Python, everything is an object. We are completely fine adding two integers or two strings with the + (plus) operator, it behaves in expected ways:
```
>>> 1 + 2
3

>>> 'hello' + ' world'
'hello world'
```
Again, we see polymorphism at play: Did you notice how + behaves different depending the type of the object? For integers it sums, for strings it concatenates. Again doing a quick dir() on the object reveals the corresponding “dunder” interface into the data model:
```
>>> dir(1)
[...
'__add__',
...
'__radd__',
...]
```
Our Account object does not support addition yet, so when you try to add two instances of it there’s a TypeError:
```
>>> acc + acc2
TypeError: "unsupported operand type(s) for +: 'Account' and 'Account'"
```
Let’s implement __add__ to be able to merge two accounts. The expected behavior would be to merge all attributes together: the owner name, as well as starting amounts and transactions. To do this we can benefit from the iteration support we implemented earlier:
```
def __add__(self, other):
    owner = '{}&{}'.format(self.owner, other.owner)
    start_amount = self.amount + other.amount
    acc = Account(owner, start_amount)
    for t in list(self) + list(other):
        acc.add_transaction(t)
    return acc
```
Yes, it is a bit more involved than the other dunder implementations so far. It should show you though that you are in the driver’s seat. You can implement addition however you please. If we wanted to ignore historic transactions—fine, you can also implement it like this:
```
def __add__(self, other):
    owner = self.owner + other.owner
    start_amount = self.balance + other.balance
    return Account(owner, start_amount)
```
I think the former implementation would be more realistic though, in terms of what a consumer of this class would expect to happen.

Now we have a new merged account with starting amount $110 (10 + 100) and balance of $240 (80 + 160):
```
>>> acc3 = acc2 + acc
>>> acc3
Account('tim&bob', 110)

>>> acc3.amount
110
>>> acc3.balance
240
>>> acc3._transactions
[20, 40, 20, -10, 50, -20, 30]
```
Note this works in both directions because we’re adding objects of the same type. In general, if you would add your object to a builtin (int, str, …) the __add__ method of the builtin wouldn’t know anything about your object. In that case you need to implement the reverse add method (__radd__) as well. You can see an example for that here.

### Callable Python Objects: __call__
You can make an object callable like a regular function by adding the __call__ dunder method. For our account class we could print a nice report of all the transactions that make up its balance:
```
class Account:
    # ... (see above)

    def __call__(self):
        print('Start amount: {}'.format(self.amount))
        print('Transactions: ')
        for transaction in self:
            print(transaction)
        print('\nBalance: {}'.format(self.balance))
```
Now when I call the object with the double-parentheses acc() syntax, I get a nice account statement with an overview of all transactions and the current balance:
```
>>> acc = Account('bob', 10)
>>> acc.add_transaction(20)
>>> acc.add_transaction(-10)
>>> acc.add_transaction(50)
>>> acc.add_transaction(-20)
>>> acc.add_transaction(30)

>>> acc()
Start amount: 10
Transactions:
20
-10
50
-20
30
Balance: 80
```
Please keep in mind that this is just a toy example. A “real” account class probably wouldn’t print to the console when you use the function call syntax on one of its instances. In general, the downside of having a __call__ method on your objects is that it can be hard to see what the purpose of calling the object is.

Most of the time it’s therefore better to add an explicit method to the class. In this case it probably would’ve been more transparent to have a separate Account.print_statement() method.

### Context Manager Support and the With Statement: __enter__, __exit__
My final example in this tutorial is about a slightly more advanced concept in Python: Context managers and adding support for the with statement.

Now, what is a “context manager” in Python? Here’s a quick overview:

> A context manager is a simple “protocol” (or interface) that your object needs to follow so it can be used with the with statement. Basically all you need to do is add __enter__ and __exit__ methods to an object if you want it to function as a context manager.

Let’s use context manager support to add a rollback mechanism to our Account class. If the balance goes negative upon adding another transaction we rollback to the previous state.

We can leverage the Pythonic with statement by adding two more dunder methods. I’m also adding some print calls to make the example clearer when we demo it:
```
class Account:
    # ... (see above)

    def __enter__(self):
        print('ENTER WITH: Making backup of transactions for rollback')
        self._copy_transactions = list(self._transactions)
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print('EXIT WITH:', end=' ')
        if exc_type:
            self._transactions = self._copy_transactions
            print('Rolling back to previous transactions')
            print('Transaction resulted in {} ({})'.format(
                exc_type.__name__, exc_val))
        else:
            print('Transaction OK')
```
As an exception has to be raised to trigger a rollback, I define a quick helper method to validate the transactions in an account:
```
def validate_transaction(acc, amount_to_add):
    with acc as a:
        print('Adding {} to account'.format(amount_to_add))
        a.add_transaction(amount_to_add)
        print('New balance would be: {}'.format(a.balance))
        if a.balance < 0:
            raise ValueError('sorry cannot go in debt!')
```
Now I can use an Account object with the with statement. When I make a transaction to add a positive amount, all is good:
```
acc4 = Account('sue', 10)

print('\nBalance start: {}'.format(acc4.balance))
validate_transaction(acc4, 20)

print('\nBalance end: {}'.format(acc4.balance))
```
Executing the above Python snippet produces the following printout:
```
Balance start: 10
ENTER WITH: Making backup of transactions for rollback
Adding 20 to account
New balance would be: 30
EXIT WITH: Transaction OK
Balance end: 30
```
However when I try to withdraw too much money, the code in __exit__ kicks in and rolls back the transaction:
```
acc4 = Account('sue', 10)

print('\nBalance start: {}'.format(acc4.balance))
try:
    validate_transaction(acc4, -50)
except ValueError as exc:
    print(exc)

print('\nBalance end: {}'.format(acc4.balance))
```
In this case we get a different result:
```
Balance start: 10
ENTER WITH: Making backup of transactions for rollback
Adding -50 to account
New balance would be: -40
EXIT WITH: Rolling back to previous transactions
ValueError: sorry cannot go in debt!
Balance end: 10
```
# Tutorial: Basic Statistics in Python — Probability
## What is probability?
At the most basic level, probability seeks to answer the question, “What is the chance of an event happening?” An event is some outcome of interest.
To calculate the chance of an event happening, we also need to consider all the other events that can occur.
The quintessential representation of probability is the humble coin toss. In a coin toss the only events that can happen are:

- Flipping a heads
- Flipping a tails
These two events form the sample space, the set of all possible events that can happen. To calculate the probability of an event occurring, we count how many times are event of interest can occur (say flipping heads) and dividing it by the sample space. Thus, probability will tell us that an ideal coin will have a 1-in-2 chance of being heads or tails. By looking at the events that can occur, probability gives us a framework for making predictions about how often events will happen. However, even though it seems obvious, if we actually try to toss some coins, we’re likely to get an abnormally high or low counts of heads every once in a while. If we don’t want to make the assumption that the coin is fair, what can we do? We can gather data! We can use statistics to calculate probabilities based on observations from the real world and check how it compares to the ideal.
## From statistics to probability
Our data will be generated by flipping a coin 10 times and counting how many times we get heads. We will call a set of 10 coin tosses a trial. Our data point will be the number of heads we observe. We may not get the “ideal” 5 heads, but we won’t worry too much since one trial is only one data point. If we perform many, many trials, we expect the average number of heads over all of our trials to approach the 50%. The code below simulates 10, 100, 1000, and 1000000 trials, and then calculates the average proportion of heads observed.
```
import random
def coin_trial():
heads = 0
for i in range(100):
    if random.random() <= 0.5:
        heads +=1
    return heads
def simulate(n):
    trials = []
    for i in range(n):
        trials.append(coin_trial())
    return(sum(trials)/n)
simulate(10)
>>> 5.4
simulate(100)
>>> 4.83
simulate(1000)
>>> 5.055
simulate(1000000)
>>> 4.999781
```
The coin_trial function is what represents a simulation of 10 coin tosses. It uses the random() function to generate a float between 0 and 1, and increments our heads count if it’s within half of that range. Then, simulate repeats these trials depending on how many times you’d like, returning the average number of heads across all of the trials. The coin toss simulations give us some interesting results.

First, the data confirm that our average number of heads does approach what probability suggests it should be. Furthermore, this average improves with more trials. In 10 trials, there’s some slight error, but this error almost disappears entirely with 1,000,000 trials. As we get more trials, the deviation away from the average decreases. Sound familiar? Sure, we could have flipped the coin ourselves, but Python saves us a lot of time by allowing us to model this process in code. As we get more and more data, the real-world starts to resemble the ideal.

Thus, given enough data, statistics enables us to calculate probabilities using real-world observations. Probability provides the theory, while statistics provides the tools to test that theory using data. The descriptive statistics, specifically mean and standard deviation, become the proxies for the theoretical. You may ask, “Why would I need a proxy if I can just calculate the theoretical probability itself?” Coin tosses are a simple toy example, but the more interesting probabilities are not so easily calculated.

What is the chance of someone developing a disease over time? What is the probability that a critical car component will fail when you are driving?
There are no easy ways to calculate probabilities, so we must fall back on using data and statistics to calculate them.
Given more and more data, we can become more confident that what we calculate represents the true probability of these important events happening.
That being said, remember from our previous statistics post that you are a sommelier-in-training.
You need to figure out which wines are better than others before you start purchasing them.
You have a lot of data on hand, so we’ll use our statistics to guide our decision.
The data and the distribution

Before we can tackle the question of “which wine is better than average,” we have to mind the nature of our data. Intuitively, we’d like to use the scores of the wines to compare groups, but there comes a problem: the scores usually fall in a range. How do we compare groups of scores between types of wines and know with some degree of certainty that one is better than the other? Enter the normal distribution. The normal distribution refers to a particularly important phenomenon in the realm of probability and statistics. The normal distribution looks like this:
![image1](https://i.imgur.com/3vDS2Au.png)

The most important qualities to notice about the normal distribution is its symmetry and its shape. We’ve been calling it a distribution, but what exactly is being distributed? It depends on the context. In probability, the normal distribution is a particular distribution of the probability across all of the events. The x-axis takes on the values of events we want to know the probability of. The y-axis is the probability associated with each event, from 0 to 1.

We haven’t discussed probability distributions in-depth here, but know that the normal distribution is a particularly important kind of probability distribution. In statistics, it is the values of our data that are being distributed. Here, the x-axis is the values of our data, and the y-axis is the count of each of these values. Here’s the same picture of the normal distribution, but labelled according to a probability and statistical context:

![image2](https://i.imgur.com/egqrj58.jpg)

In a probability context, the high point in a normal distribution represents the event with the highest probability of occurring. As you get farther away from this event on either side, the probability drops rapidly, forming that familiar bell-shape. The high point in a statistical context actually represents the mean. As in probability, as you get farther from the mean, you rapidly drop off in frequency. That is to say, extremely high and low deviations from the mean are present but exceedingly rare.

If you suspect there is another relationship between probability and statistics through the normal distribution, then you are correct in thinking so! We will explore this important relationship later in the article, so hold tight. Since we’ll be using the distribution of scores to compare different wines, we’ll do some set up to capture some wines that we’re interested in. We’ll bring in the wine data and then separate out the scores of some wines of interest to us. To bring back in the data, we need the following code:
```
import csv
with open("wine-data.csv", "r", encoding="latin-1") as f:
    wines = list(csv.reader(f))
```
The data is shown below in tabular form. We need the points column, so we’ll extract this into its own list. We’ve heard from one wine expert that the Hungarian Tokaji wines are excellent, while a friend has suggested that we start with the Italian Lambrusco. We have the data to compare these wines!
```
# Extract the Tokaji scores
tokaji = []
non_tokaji = []
for wine in wines:
    if points != '':
        points = wine[4]
    if wine[9] == "Tokaji":
    tokaji.append(float(points))
    else:
        non_tokaji.append(points)
# Extract the Lambrusco scores
lambrusco = []
non_lambrusco = []
for wine in wines:
    if points != '':
        points = wine[4]
    if wine[9] == "Lambrusco":
        lambrusco.append(float(points))
    else:
        non_lambrusco.append(float(points))
```
If we visualize each group of scores as normal distributions, we can immediately tell if two distributions are different based on where they are. But we will quickly run into problems with this approach, as shown below. We assume the scores will be normally distributed since we have a ton of data. While that assumption is okay here, we’ll discuss later when it may actually be dangerous to do so.
![image3](https://i.imgur.com/h4FR5XL.jpg)

When the two score distributions overlap too much, it’s probably better to assume thy actually come from the same distribution and aren’t different. On the other extreme with no overlap, it’s safe to assume that the distributions aren’t the same. Our trouble lay in the case of some overlap. Given that the extreme highs of one distribution may intersect with the extreme lows of another, how can we say if the groups are different? Here, we must again call upon the normal distribution to give us an answer and a bridge between statistics and probability.

## Revisiting the normal
The normal distribution is significant to probability and statistics thanks to two factors: the Central Limit Theorem and the Three Sigma Rule.

## Central Limit Theorem
In the previous section, we demonstrated that if we repeated our 10-toss trials many, many times, the average heads-count of all of these trials will approach the 50% we expect from an ideal coin. With more trials, the closer the average of these trials approach the true probability, even if the individual trials themselves are imperfect. This idea is a key tenet of the Central Limit Theorem. In our coin-tossing example, a single trial of 10 throws produces a single estimate of what probability suggests should happen (5 heads). We call it an estimate because we know that it won’t be perfect (i.e. we won’t get 5 heads every time).

If we make many estimates, the Central Limit Theorem dictates that the distribution of these estimates will look like a normal distribution. The zenith of this distribution will line up with the true value that the estimates should take on. In statistics, the peak of the normal distribution lines up with the mean, and that’s exactly what we observed. Thus, given multiple “trials” as our data, the Central Limit Theorem suggests that we can hone in on the theoretical ideal given by probability, even when we don’t know the true probability. Central Limit Theorem lets us know that the average of many trials means will approach the true mean, the Three Sigma Rule will tell us how much the data will be spread out around this mean.

## Three Sigma Rule
The Three Sigma rule, also known as the empirical rule or 68-95-99.7 rule, is an expression of how many of our observations fall within a certain distance of the mean. Remember that the standard deviation (a.k.a. “sigma”) is the average distance an observation in the data set is from the mean. The Three Sigma rule dictates that given a normal distribution, 68% of your observations will fall between one standard deviation of the mean. 95% will fall within two, and 99.7% will fall within three. A lot of complicated math goes into the derivation of these values, and as such, is out of the scope of this article. The key takeaway is to know that the Three Sigma Rule enables us to know how much data is contained under different intervals of a normal distribution. The picture below is a great summary of what the Three Sigma Rule represents. 

![image3](https://i.imgur.com/Mt3RyE0.png)

We’ll connect these concepts back to our wine data. As a sommelier, we’d like to know with high confidence that Chardonnay and Pinot Noir are more popular than the average wine. We have many thousands of wine reviews, so by Central Limit Theorem, the average score of these reviews should line up with a so-called “true” representation of the wine’s quality (as judged by the reviewer). Although the Three Sigma rule is a statement of how much of your data falls within known values, it is also a statement of the rarity of extreme values. Any value that is more than three standard deviations away from the mean should be treated with caution or care. By taking advantage of the Three Sigma Rule and the Z-score, we’ll finally be able to prescribe a value to how likely Chardonnay and Pinot Noir are different from the average wine.

## Z-score
The Z-score is a simple calculation that answers the question, “Given a data point, how many standard deviations is it away from the mean?” The equation below is the Z-score equation.

![image4](https://i.imgur.com/3TuDF4G.jpg)
By itself, the Z-score doesn’t provide much information to you. It gains the most value when compared against a Z-table, which tabulates the cumulative probability of a standard normal distribution up until a given Z-score. A standard normal is a normal distribution with a mean of 0 and a standard deviation of 1. The Z-score lets us reference this the Z-table even if our normal distribution is not standard. The cumulative probability is the sum of the probabilities of all values occurring, up until a given point.

An easy example is the mean itself. The mean is the exact middle of the normal distribution, so we know that the sum of all probabilites of getting values from the left side up until the mean is 50%. The values from the Three Sigma Rule actually come up if you try to calculate the cumulative probability between standard deviations. The picture below provides a visualization of the cumulative probability.

![image5](https://i.imgur.com/utLmb5p.jpg)

We know that the sum of all probabilities must equal 100%, so we can use the Z-table to calculate probabilities on both sides of the Z-score under the normal distribution.

![image6](https://i.imgur.com/pGyuZ1q.jpg)

calculation of probability of being past a certain Z-score is useful to us. It lets us ask go from “how far is a value from the mean” to “how likely is a value this far from the mean to be from the same group of observations?” Thus, the probability derived from the Z-score and Z-table will answer our wine based questions.
```
import numpy as np
tokaji_avg = np.average(tokaji)
lambrusco_avg = np.average(lambrusco)
tokaji_std = np.std(tokaji)
lambrusco = np.std(lambrusco)
# Let's see what the results are
print("Tokaji: ", tokaji_avg, tokaji_std)
print("Lambrusco: ", lambrusco_avg, lambrusco_std)
>>> Tokaji: 90.9 2.65015722804
>>> Lambrusco: 84.4047619048 1.61922267961
```
This doesn’t look good for our friend’s recommendation! For the purpose of this article, we’ll treat both the Tokaji and Lambrusco scores as normally distributed. Thus, the average score of each wine will represent their “true” score in terms of quality. We will calculate the Z-score and see how far away the Tokaji average is from the Lambrusco.
```
z = (tokaji_avg - lambrusco_avg) / lambrusco_std
>>> 4.0113309781438229
# We'll bring in scipy to do the calculation of probability from the Z-table
import scipy.stats as st
st.norm.cdf(z)
>>> 0.99996981130231266
# We need the probability from the right side, so we'll flip it!
1 - st.norm.cdf(z)
>>> 3.0188697687338895e-05
```
The answer is quite small, but what exactly does it mean? The infinitesimal smallness of this probability requires some careful interpretation. Let’s say that we believed that there was no difference between our friend’s Lambrusco and the wine expert’s Tokaji. That is to say, we believe that the quality of the Lambrusco and the Tokaji to be about the same. Likewise, due to individual differences between wines, there will be some spread of the scores of these wines. This will produce normally distributed scores if we make a histogram of the Tokaji and Lambrusco wines, thanks to Central Limit Theorem.

Now, we have some data that allows us to calculate the mean and standard deviation of both wines in question. These values allow us to actually test our belief that Lambrusco and Tokaji were of similar quality. We used the Lambrusco wine scores as a base and compared the Tokaji average, but we could have easily done it the other way around. The only difference would be a negative Z-score. The Z-score was 4.01! Remember that the Three Sigma Rule tells us that 99.7% of the data should fall within 3 standard deviations, assuming that Tokaji and Lambrusco were similar.

The probability of a score average as extreme as Tokaji’s in a world where Lambrusco and Tokaji wines are assumed to be the same is very, very small. So small that we are forced to consider the converse: Tokaji wines are different from Lambrusco wines and will produce a different score distribution. We’ve chosen our wording here carefully: I took care not to say, “Tokaji wines are better than Lambrusco.” They are highly probable to be. This is because we calculated a probability which, though microscopically small, is not zero. In order to be precise, we can say that Lambrusco and Tokaji wines are definitively not from the same score distribution, but we cannot say that one is better or worse than the other.

This type of reasoning is within the domain of inferential statistics, and this article only seeks to give you a brief introduction into the rationale behind it. We covered a lot of concepts in this article, so if you found yourself getting lost, go back and take it slow. Having this framework of thinking is immensely powerful, but easy to misuse and misunderstand.

# Bookmark/Skim
[Statistics Module](https://docs.python.org/3/library/statistics.html)









