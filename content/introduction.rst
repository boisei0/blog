Python for Investigative Journalists: An introduction
#####################################################

:date: 2014-07-23 10:30
:tags: python, twitter
:category: python
:slug: introduction-to-python
:authors: Arlena Derksen
:summary: An introduction to the programming language Python: why is it useful for (investigative) journalists and what are it's possibilities

This post is the first of a series about the use of Python, a programming language, and it's uses for investigative
journalists. That might sound a bit strange, because programming is not the first thing to think of when doing research.
Starting with simple examples going to more advanced cases, I would like to show how to create your own toolkit for
future use.

Why programming
---------------
The first thing which comes to mind when talking about programming is (one of) it's goal(s): to automate common tasks.
In journalism and research, a common task can be very broad. For example, logging all tweets posted using a similar
hashtag from within a given area can be achieved by using the geo-location send with the tweets in a specialized search.
A far more advanced example is the detection of a moving object in a video and search for recognition of this object
using public systems. Combine these two examples: search for tweets within an area linking to videos, detect moving
objects in these videos and let the program write a summary with added stills from the video to describe what happened.
This might not be accurate enough at first, however when combining enough techniques and public systems with this,
the results get better. Using machine learning, you can even have the program learn from every case it analyzes.

Why Python
----------
In these days of social media and video, sites are an important source of information. Programs need to be able to talk
to these systems; a Twitter client on your smartphone is an example of such a program. Websites combining several
sources of information are a more advanced example of this.

Python is highly extensible with libraries containing new features not available in the basic installation. For a lot of
sites, toolkits for programmers, so called APIs, are available; and with Python being a popular language of choice,
libraries containing these toolkits can be used to extend the language.
Another important point is it's readability. When choosing logical names and writing with the style guidelines in mind,
people without or with only a little bit of programming experience will be able to understand a bit what you are doing.
Furthermore, in a couple of years time, you might still understand what you were trying to do when writing pieces of
code with only scanning what you wrote. This might be able to achieve in other languages as well, however in my eyes
it is easier in a language like Python.

And finally, Python is cross-platform. This means that when you write a program on your Windows computer, it will work
exactly the same when running from your Macbook. When reading this, it does not matter what operating system you are
using, because almost everything will work. With that being said, Windows RT (running on systems like the Surface RT
tablet) will likely not work.

Getting Python and an editor
----------------------------
For this series of tutorials, we will be using Anaconda. Anaconda is a distribution of Python which includes apart from
the basic install a lot of libraries. This might seem a bit overkill for the basic scripts, however when using more
advanced techniques, a lot of the included libraries are needed. These libraries, mostly for scientific usage, are
a real pain-in-the-ass to install on Windows.
Point your browser to `https://store.continuum.io/cshop/anaconda <https://store.continuum.io/cshop/anaconda/>`_. Don't
worry, the distribution is available free of charge. Click *Download Anaconda*, fill in your e-mail and let it redirect
you to the download page. Choose there for a distribution which matches your platform. Download the package, somewhere
between 300 and 500mb and install it on your computer.

You can write Python programs using **any** editor you would like. However, some editors are especially made for
writing Python programs. For these tutorials, I'm using PyCharm. Included with PyCharm as part of it's settings,
is a graphical user interface for installing libraries. Although the Anaconda distribution includes almost 200
libraries, we're going to use quite a few which are not included. In my experience, it is not fun to install these on
platforms like Windows, even when you are used to the commandline. When not used to commandline interfaces, it can be a
real hell.

The community version of PyCharm is available free of charge from `JetBrains <http://www.jetbrains.com/pycharm/>`_.
Click on 'Get PyCharm now', choose your platform and download the Community Edition. When installed, open it,
click 'settings' and point the 'Python Interpreter' to the place where you installed Anaconda. Move to the 'bin'
directory and choose for ``python`` (or ``python.exe``, if you are using Windows). This should be enough to set up
Python and PyCharm for usage.

A practical example
-------------------
A simple setup of the first example can be managed in Python with the addition of a library: Tweepy. Tweepy is not
included in the Anaconda distribution. For this, we use PyCharm to install it. Start PyCharm and again, go to the
settings. Select your Python installation (which is the Anaconda install by default, if you've never used or installed
Python before) and click the '+' button to install a new package (for example a library like Tweepy). Search for
'Tweepy', select the package with the matching name and click 'install'. The package should install without errors.

For Tweepy to work, you need a Twitter API key. Go to `apps.twitter.com <https://apps.twitter.com>`_ and log in with
your Twitter account or create an account. Click 'Create a new app' and fill in the form. You do not need to enter a
callback url for a standalone application like we are going to make. The system will give you an API key and an API
secret. When you have your application selected in this menu, click on 'API Keys' and generate an access token.
These four keys are used in the application. The API key and the API secret are used in the application itself, the
Access token and the Access token secret are used to identify your account with the application. In applications used by
multiple Twitter accounts, the access token and it's secret are obtained by a oauth call. We might talk about that in a
later post.

Enough talk for now, I'll move on to the example and explain what it does.

.. code-block:: python

    # coding=utf8
    # A program to search twitter for a given search query in a given area.
    import tweepy

    api_key = 'API KEY'  # replace API KEY with your api key
    api_secret = 'API SECRET'  # replace API SECRET with your api secret
    access_token = 'ACCESS TOKEN'  # replace ACCESS TOKEN with your access token
    access_token_secret = 'ACCESS TOKEN SECRET'  # replace ACCESS TOKEN SECRET with your access token secret

    search_query = '#GazaUnderAttack'
    latitude = 31.494722  # latitude of the location
    longitude = 34.451667  # longitude of the location
    search_range = 15  # range in kilometers

    search_area = '{0},{1},{2}km'.format(latitude, longitude, search_range)

    authentication = tweepy.OAuthHandler(api_key, api_secret)
    authentication.set_access_token(access_token, access_token_secret)

    api = tweepy.API(authentication)
    search = api.search(q=search_query, geocode=search_area)

    for item in search:
        print(item.text)

That might look quite complicated. We will walk through the program line by line.

.. code-block:: python

    # coding=utf8
    # A program to search twitter for a given search query in a given area.

In Python, every line starting with a '#' is known as a comment. You can enter everything you want behind this sign and
the program won't pick it up as code. This way you can document what you are writing, write instructions for yourself or
for others and make sure you understand what you just wrote in a couple of years time.
``# coding=utf8`` is a special instruction for the Python interpreter, the program that reads your code and executes it.
To make sure non-ascii characters, for example Arabic or Cyrillic characters, can be used in your program without the
interpreter falling over it, put this on the first line of your program (or the second line if you specify which
interpreter must be used, but as we didn't in this example, just put it on the first line).

.. code-block:: python

    import tweepy

Every non-comment line contains one statement. In Python, every library you would like to use in your program has to be
loaded. Even the libraries included with the basic installation of Python must be imported to use them.

.. code-block:: python

    api_key = 'API KEY'  # replace API KEY with your api key
    api_secret = 'API SECRET'  # replace API SECRET with your api secret
    access_token = 'ACCESS TOKEN'  # replace ACCESS TOKEN with your access token
    access_token_secret = 'ACCESS TOKEN SECRET'  # replace ACCESS TOKEN SECRET with your access token secret

Now it is getting more interesting: variables. A variable can hold a value, whether that is a piece of text, a number or
a more complex data structure like a list of values. In Python, all you have to do to declare a variable is to give it
a name and give it a value, like this: ``name = value``. To enter a piece of text, a so called string, put it between
quotes. You can use both single and double quotes, but not a mix of them. Single quotes are preferred by the style
guideline known as PEP8. By following this set of guidelines, even large programs are easy enough to read. PEP8 does
contain a section about variable naming as well: use whole words or even multiple words and replace the spaces by
underscores. So the most logical name for a access token secret is ``access_token_secret``.
Again, we see comments, however in a slightly different way. When added somewhere halfway a line, they start a comment
till the end of the line. Another style guideline from PEP8 says that when you add a comment to a line with a statement,
separate the statement and the comment by two (2) spaces, add a '#' to start the comment and put another space there
before starting the comment text. The same thing applies to regular comments.

.. code-block:: python

    search_query = '#GazaUnderAttack'
    latitude = 31.494722  # latitude of the location
    longitude = 34.451667  # longitude of the location
    search_range = 15  # range in kilometers

Here we have another set of variables. Note the hashtag in the search query. The next part of the line is not read as a
comment however, because it is enclosed by (single) quotes, thus being part of a string. The next variable ``latitude``,
is not a string. The decimal value is known as a floating point number, float in short. To declare a variable as a
float, simply put it there without any quotation marks. Use a dot as the decimal separator. The variable
``search_range`` is a number, known as a integer. To make things complicated in programming, 15 is not the same as 15.0,
because the first is an integer and the second one a float. For now, let's not focus on that fact and continue with the
next block.

.. code-block:: python

    search_area = '{0},{1},{2}km'.format(latitude, longitude, search_range)

This line is a matter of taste. There are more ways to glue different variables together to one string. I prefer this
way, others might prefer other ways. With *my* way, more advanced formatting is possible. Again, we declare a variable
of a string type. This time however, we also call a function to do something with this string. By using the format
function, it is possible to replace parts enclosed in brackets by specified variables. In most programming languages,
including Python, we start counting from 0 instead of 1. When calling the format function, you supply variables to
replace parts in brackets. The part marked as ``{0}`` is replaced by the first supplied variable, the part marked
``{1}`` is replaced by the second supplied variable and so on. You can even reuse variables, if for example you wrote
this: ``'{0} is the same as {0}'.format(variable)``. To call the function format on a string, add a dot to the value
like seen above, and type the name of the function, which in this case is ``format``. Add parentheses and between them
add the variables, separated with commas.

If not used to this way of writing, it looks difficult. As I said, there are different ways to do this. I'll list a
couple more:

.. code-block:: python

    search_area = str(latitude) + ',' + str(longitude) + ',' + str(search_range) + 'km'
    search_area = '31.494722,34.45166,15km'

I'll start with the second line. Although this is perfectly possible, it is not what we want. It might work out fine
for now, but every time we use a different location, we have to change this line. In the current situation, we only have
to change the variables latitude and longitude to a different value. We might even fill them dynamically...
Some people prefer the first line. To glue strings together, just sum them up. However, every part must be a string for
this to work. Both latitude and longitude are floats and with search_range being an integer, just summing them up does
not work. We have to 'cast' every non-string to a string, by calling the 'string' function. The string function takes
one argument, the variable we want to cast to a string. It returns the same variable, but with the string type. The
format function, described above, does the same thing in putting the string together. However, it automatically casts
a non-string variable you supply to it to a string.

As I said before, it is a matter of taste. Both cases work, but the format function does the checking to see which
variables must be translated to a string type.

.. code-block:: python

    authentication = tweepy.OAuthHandler(api_key, api_secret)
    authentication.set_access_token(access_token, access_token_secret)

Next, we are finally going to use the Tweepy library. We call the 'OAuthHandler' function from the tweepy library and
supply our api key and api secret. The resulting 'object' is put in the variable authentication.
On the object authentication, we call the function set_access_token and supply our access token and access token
secret. As we can see, ``authentication`` is a special kind of variable. I'll explain more about this in future posts.
For now, we just accept that this variable has it's own set of functions.

.. code-block:: python

    api = tweepy.API(authentication)
    search = api.search(q=search_query, geocode=search_area)

Until now, we did the make a lot of variables and used a couple of functions. Here, we are finally going to use them.
The first line authenticates our client by Twitter and logs in with the account used to create the application.
The resulting object is put in the variable api. Like ``authentication``, this is a special kind of variable as well.
The second line is where we were al waiting for: the actual search. We call the function search on the api object and
supply the search query and the geocode. By all functions until now, we had positional arguments; arguments that had to
be supplied in the correct order. This function uses keyword arguments. The other functions had keyword arguments as
well, but we did not use them, as keyword arguments are optional. All arguments, both positional and keyword arguments,
are explained in the `Tweepy documentation <http://tweepy.readthedocs.org/en/v2.3.0/>`_.

This function gives another object in return. This object, of the type ``tweepy.models.SearchResults``, has a couple of
features we will use in the next part: it is basically a collection of data that we can walk through.

.. code-block:: python

    for item in search:
        print(item.text)

The final part of the program. As described in the previous block, we can walk through ``search``. In Python, this
feature is called an *iterable*. By using the for statement, we can walk through an iterable object. This is done by
telling Python we want to loop through ``search`` and every time we do that, we the next piece of data out of it, until
we reach it's end. This piece of data we pull out will be put in the variable ``item``. This variable can only be used
within the block that follows. To open this block, put a colon to the end of the line. All lines within the block are
indented with four (4) spaces or a tab. Do not merge tabs and spaces, or the interpreter will fall over it. PEP8, the
style guidelines prefer the use of four spaces. Python versions 3 and above do not even accept tabs. When using PyCharm,
you do not have to worry about this. When you press tab, the editor inserts the correct amount of spaces instead of a
tab. When you stop indenting lines, the interpreter assumes the block has ended. Every time the for statement loops over
``search``, the whole block that follows is repeated. The only thing that is different every time, is the value of the
variable ``item``. The block must have at least one line. In this case, it is exactly one line.

Here, we introduce another function: the print function. What it does is very simple. It prints the value of the
supplied variable to the output. The variable ``item`` is another kind of special object. It contains a lot of values,
including ``text``. That value is the text of the tweet.

Other values of ``item`` are: ``author``, ``contributors``, ``coordinates``, ``id``, ``geo``, ``lang``, ``place``,
``user``, and a lot more. To view all functions and values an object has, use the ``dir`` function. The dir function
takes one argument: the object you would like information about. You can use it like this:

.. code-block:: python

    print(dir(item))

Run your program
----------------
I did almost forget to add this part... If you did not launch PyCharm already, run it now. Create a new project, give
it a name and select the Anaconda distribution as it's interpreter. Create a new file in the project. PyCharm will
automatically give it the extension '.py'. I called this file 'introduction_first_example.py'. Put the code from the
example in the file, and click in the menu bar on 'Run' -> 'Run introduction_first_example'. This will run your program
and display it's output in the console in the bottom of the screen.

Congratulations, you made it to here and just ran your first program. Change some values, run it again, see how it
changes the output. Use different values from ``item`` and experiment with it.

Conclusion
----------
I'll end this introduction here. In my editor, I just passed the 260 lines. I think that's enough for one post and in
the upcoming posts I will continue on this subject. If there are questions or requests, please let me know. You can
find me on Twitter as `@boisei0 <http://twitter.com/boisei0>`_.

I consider the code in this introduction and in all the upcoming posts as open source. I'm not sure of the exact license
yet, but you can consider it Public Domain. I'll put up a notice about that later.
