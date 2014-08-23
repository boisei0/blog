Introduction to Object Oriented Programming
###########################################

:date: 2014-08-17 12:30
:tags: python, oop
:category: python
:slug: object-oriented-programming
:authors: Rob Derksen
:summary: A more difficult subject this time: an introduction to objects and object oriented programming, explained with furniture.

Objects and classes, we've seen them before, but what are they and how do they work: at the end of this chapter, answers
to these questions are given. I was planning to put in an example with the twitter streaming API, but I think it is
better to put that one in a separate post. This post is first and foremost theoretical, but it contains essential
information to understand subjects we'll be using in future chapters. I've chosen to explain this subject with
furniture, chairs and tables in particular. Several years ago, my teacher used the same examples to explain this subject
to me and I consider this *easy* ever since. In the years that followed, this subject was explained several times during
other classes, but I've yet to hear an explanation that everyone in the room was able to understand, apart from the time
with the furniture.

Furniture
---------
There we go, furniture. According to the Oxford Dictionary of English, the definition of a chair is *a separate seat
for one person, typically with a back and four legs*. Next, we have a stool, *a seat without a back or arms, typically
resting on three or four legs or on a single pedestal*. When comparing definitions, we can tell that a stool is most
likely not a chair, because it has no back or arms. We can assume they both have at least one leg. By definition, they
also have a seat.
Table, *a piece of furniture with a flat top and one or more legs, providing a level surface for eating, writing or
working at*. Is a table a chair or a stool? No, it is not a seat, *a thing made or used for sitting on*. You are able
to sit on a table, however a table is not made for sitting on. You are able to eat with your plate on a chair, however
a chair is not made to use as a surface for eating or writing.

Objects
-------
What has furniture to do with programming? More than you might expect. Every piece of mentioned furniture has attributes
and actions that can be performed with these *objects*. In the previous posts, we've worked with objects. It started
with the ``authentication`` object in the introduction:

.. code-block:: python

    authentication = tweepy.OAuthHandler(api_key, api_secret)
    authentication.set_access_token(access_token, access_token_secret)

The action ``tweepy.OAuthHandler(api_key, api_secret)`` returns an object. The action ``set_access_token(access_token,
access_token_secret)`` can be performed with this object. Later in the introduction, we've seen the ``api`` object,
which had the action ``search``. Even the tweets were objects: every ``item`` in ``search`` was an object of the type
``tweepy.models.SearchResults``.

In *object oriented programming*, there are things as objects and classes. A class is a model, a prototype, with common
attributes and actions. For example, the class ``Table`` is a prototype of the table. It's attributes: legs, surface,
the material it is made of. The table I'm sitting at while writing this, is made of wood, has four legs and has a flat
surface. This particular table can be seen as a, so called, *instance* of the class ``Table``. An object, based on a
class, is called an instance of that class. The actions of the class are called *methods*. Standalone, we call them
*functions*.

Functions in Python
-------------------
In Python, standalone functions are defined with the word ``def``, followed by the name of the function and any number
of arguments, also known as *parameters*. Next, add a colon and start a new block on the new line. Inside, write your
code. Every time you call this function, the code inside the block will be executed. I wrote a few examples for this:

.. code-block:: python

    def example_function(first_argument, second_argument, third_argument):
        do_something(first_argument, second_argument)
        do_something_else(third_argument)

Another example for a function without arguments:

.. code-block:: python

    def function_without_arguments():
        print('hello world!')


Within functions, there is a special keyword: ``return``. The return keyword returns the value, which can be an object,
to the parent code, where it can be saved in a variable or directly used. See the following example for a function which
sums it's arguments and returns them.

.. code-block:: python

    def sum_arguments(a, b):
        return a + b

Classes in Python
-----------------
In Python, classes are defined by the ``class`` keyword, followed by the name of the class and a colon. According to
PEP8, the style guidelines, class names start with a capital character, and every word that follows starts with a
capital as well. So instead of an ``example_class``, we see an ``ExampleClass``. Every class should have a constructor,
a method that is called when the class is created. The arguments of this method are passed in the creation of an object.
We've seen this in the second post about data control structures. When we created the ``translate`` object, which is an
instance of the class ``YandexTranslate``, we passed the variable ``yandex_api`` as an argument. The constructor is a
special method with the name ``__init__``. The first argument of any method within a class should be the special keyword
``self``. Values can be stored within the class, as attribute, by assigning them to a variable in ``self``. For example,
see the following snippet:

.. code-block:: python

    class ExampleClass:
        def __init__(self, argument):
            self.example = argument

        def example_method(self):
            print('An example method without arguments')

Conclusion
----------
Object oriented programming is a difficult subject. I hope I was able to introduce the subject in a way, that you were
able to understand. If not, please ask questions, either here or via `Twitter <https://twitter.com/boisei0>`_. In the
next post, I'll continue on this subject. In the longer example, featuring the Twitter streaming API, we'll be using
a part of object oriented programming called inheritance: we are going to write a custom class based on an existing one.
