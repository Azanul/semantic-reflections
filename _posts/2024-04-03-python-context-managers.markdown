---
layout: post
title:  "Python context managers"
date:   2024-04-03 14:41:41 +0000
categories: python expert
---
The [with] statement in Python is primarily used for resource management. It ensures that a resource is properly initialized and released when it's no longer needed, even if exceptions occur during the execution. It can also be used for suppressing certain exceptions or outputs.
Basically, [context-managers] can be leveraged to create a temporary state change and revert it to original once the intended work is done.
A few common use cases for [context-managers] are:
- File I/O Operations: ensuring that file is properly closed after use.
- Network Connections: ensuring the connection is properly closed after use.
- Transaction Management: managing transactions - beggining, commiting, aborting, etc.
- Resource Management: automatically releasing the lock, freeing the resource after use.

Let's take an example to explore [context-managers] further. Let's say that we have a function which prints a message to standard output. We want to capture this output instead of releasing it to standard output.
Imagine you're a super spy with a secret message, but you don't want anyone to see it! That's where our special code comes in. It's like a magic box that hides messages until you open it again.
1. We need a secret message to hide. To do this we simply define a function which will print a string.
{% highlight python %}
def secret_message():
    print("This message is secret and should be suppressed!")
{% endhighlight %}

2. Let's create our message box. Our message box is going to be a context manager, so it has to implement the necessary __enter__() and __exit__() methods.
__enter__() is called at the start of the context and __exit__() is called at the end, duh! 
__enter__() will return an object which can be accessed with `as` keyword. __exit__() is called with 3 arguments: exception type, exception value and traceback, these values are going to be `None` is no exception occured in the context.
In __enter__() method, we're storing the stdout in an instance valiable `_original` and setting the stdout to a `StringIO()` object. In __exit__() method, we're resetting the stdout and storing the captured output in `_original`.
{% highlight python %}
from io import StringIO
import sys

class SecretMessageBox:
    def __enter__(self):
        self._original = sys.stdout
        sys.stdout = StringIO()
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        sys.stdout, self._original = self._original, sys.stdout
{% endhighlight %}

3. Now, let's use our box to hide a message. We're using [with] statement wo invoke our context manager and `as` to capture the object returned by __enter__() method of the context manager `SecretMessageBox` into a variable `secret_box`.
{% highlight python %}
print("Start hiding messages!")
with SecretMessageBox() as secret_box:
    secret_message()
print("Stop hiding messages!\n")
{% endhighlight %}

4. Once we've hidden the message, we want to read the message later
{% highlight python %}
hidden_message = secret_box._original.getvalue()
if hidden_message:
    print("This is what we hid:\n", hidden_message)
else:
    print("There are no hidden messages yet.")
{% endhighlight %}

5. Putting it all together
{% highlight python %}
from io import StringIO
import sys

def secret_message():
    print("This message is secret and should be suppressed!")

class SecretMessageBox:
    def __enter__(self):
        self._original = sys.stdout
        sys.stdout = StringIO()
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        sys.stdout, self._original = self._original, sys.stdout

print("Start hiding messages!")
with SecretMessageBox() as secret_box:
    secret_message()
print("Stop hiding messages!\n")

hidden_message = secret_box._original.getvalue()  # Get the hidden message
if hidden_message:
    print("This is what we hid:\n", hidden_message)
else:
    print("There are no hidden messages yet.")
{% endhighlight %}

6. In case we face an exception but want to ignore certain exception or certain type of exceptions.
We define a `RuntimeError`, raise it from secret_message() function and handle it in __exit__() method.
`True` returned from __exit__() implies the exception raised has been handled else the exception is raised further down the sttack.
{% highlight python %}
NOT_REALLY_AN_EXCEPTION = RuntimeError("Just kidding")

def secret_message():
    print("This message is secret and should be suppressed!")
    raise NOT_REALLY_AN_EXCEPTION
...

class SecretMessageBox:
...
    def __exit__(self, exc_type, exc_value, traceback):
        sys.stdout, self._original = self._original, sys.stdout
        if exc_value == NOT_REALLY_AN_EXCEPTION:
            return True
...

try:
    with SecretMessageBox() as secret_box:
        secret_message()
except Exception as e:
    print("Exception occurred:", e)
    sys.exit()
finally:
    print("Stop hiding messages!\n")
{% endhighlight %}

7. Final script
{% highlight python %}
from io import StringIO
import sys

NOT_REALLY_AN_EXCEPTION = RuntimeError("Just kidding")

def secret_message():
    print("This message is secret and should be suppressed!")
    raise NOT_REALLY_AN_EXCEPTION

class SecretMessageBox:
    def __enter__(self):
        self._original = sys.stdout
        sys.stdout = StringIO()
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        sys.stdout, self._original = self._original, sys.stdout
        if exc_value == NOT_REALLY_AN_EXCEPTION:
            return True
        
print("Start hiding messages!")
try:
    with SecretMessageBox() as secret_box:
        secret_message()
except Exception as e:
    print("Exception occurred:", e)
    sys.exit()
finally:
    print("Stop hiding messages!\n")

hidden_message = secret_box._original.getvalue()
if hidden_message:
    print("This is what we hid:\n", hidden_message)
else:
    print("There are no hidden messages yet.")
{% endhighlight %}

Also checkout [contextlib], it provides some common utilities involving the [with] statement.

[with]: https://docs.python.org/3/reference/compound_stmts.html#with
[context-managers]: https://docs.python.org/3/reference/datamodel.html#context-managers
[contextlib]: https://docs.python.org/3/library/contextlib.html
