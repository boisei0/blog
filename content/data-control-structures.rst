Basic data structures
#####################

:date: 2014-08-14 09:00
:tags: python, twitter, translation
:category: python
:slug: basic-data-structures
:authors: Rob Derksen
:summary: In this post I'll tell a bit more about data types and explain another data data structure. Expect more Twitter and some machine translation.

In this post I'll tell a bit more about data types and explain another data data structure, combined with the one from
the introduction. The following example is a modification of the example from the previous post. Apart from searching
and displaying tweets, it is possible to automatically translate the tweets to a language of choice. For this, we will
use the Yandex Translate API. It has not the best quality of machine translations, but it is able to do so for free,
limited to 10,000 references with a total of 1,000,000 characters a day.

Personally, I don't consider this a very useful example. I wrote the code in the evening of the downing of MH17, with a
given search area of Torez and it's surroundings. I'm able to read Cyrillic, but translating the texts when events
are following up very quickly is not something I'm capable of. This program searches for tweets within a given search
area and translates them to English. In these kind of situations, such a program can be very useful. It is however a
good example of a couple of essential data structures and the use of the *dictionary*.

Prerequisites
-------------
For this program, we will be using the same python distribution and editor from the introduction. If not set up already,
follow the instructions in the `introduction <http://hubsec.eu/blog/introduction-to-python.html>`_.
Furthermore, we will be using the Twitter API with the Yandex Translate API. For instructions on the Twitter API, see
again the introduction (link above). The Yandex API requires a separate account. We will be using this API in the future
to translate descriptions of videos when summarizing these. Go to `<http://api.yandex.com/key/form.xml?service=trnsl>`_,
create an account and request an API key for the Translate service.

Practical example
-----------------
In the following example, we'll be using the ``tweepy`` library from the previous post and the ``yandex_translate``
module. This last module is new and not included in Anaconda. In PyCharm, go to the settings -> python interpreter ->
Anaconda and click the plus sign to install a new library. Search for ``yandex.translate`` and install this package.
The following program, as stated before, fetches tweets from a given search area and translates them to English. In this
example, the search area is a 15 kilometer radius surrounding Aleppo, Syria. Most of the tweets we fetch, will be
written in Arabic.

.. code-block:: python

    # coding=utf8
    import tweepy
    from yandex_translate import YandexTranslate

    # Twitter API keys
    twitter = {
        'api_key': 'API KEY',  # replace API KEY with your key
        'api_secret': 'API SECRET',  # replace API SECRET with your api secret
        'access_token': 'ACCESS TOKEN',  # replace ACCESS TOKEN with your access token
        'access_token_secret': 'ACCESS TOKEN SECRET'  # replace ACCESS TOKEN SECRET with your access token secret
    }
    yandex_api = 'API KEY'  # replace API KEY with your yandex translation api key

    latitude = 36.229874
    longitude = 37.170181
    search_range = 15
    search_area = '{0},{1},{2}km'.format(latitude, longitude, search_range)

    auth = tweepy.OAuthHandler(twitter['api_key'], twitter['api_secret'])
    auth.set_access_token(twitter['access_token'], twitter['access_token_secret'])
    api = tweepy.API(auth)

    translate = YandexTranslate(yandex_api)

    result = api.search(geocode=search_area)
    for tweet in result:
        translation = translate.translate(tweet.text, 'en')
        if translation['code'] == 200:
            print(u'{0}: {1}'.format(tweet.author.name, translation['text'][0]))
        else:
            print('Unable to translate tweet.')

Like in the previous post, we'll walk through this program in several blocks, to explain how it works.

.. code-block:: python

    import tweepy
    from yandex_translate import YandexTranslate

Import statements, we've seen these in the introduction. The first line is a regular import, which imports the module
tweepy. To use it's code in a program, specify the name of the module, add a dot and the name of the function, module or
*class* you would like to use, for example ``tweepy.OAuthHandler``. The import statement on the seconds line is similar,
however, it only imports ``YandexTranslate`` from ``yandex_translate``. In this program we do not need other classes
from the YandexTranslate module. Instead of ``YandexTranslate.yandex_translate``, we can now use a simple
``yandex_translate`` to specify code from this class. I've used the word *class* now a couple of times. Please accept it
for now, I'll explain this concept in the next post.

.. code-block:: python

    # Twitter API keys
    twitter = {
        'api_key': 'API KEY',  # replace API KEY with your key
        'api_secret': 'API SECRET',  # replace API SECRET with your api secret
        'access_token': 'ACCESS TOKEN',  # replace ACCESS TOKEN with your access token
        'access_token_secret': 'ACCESS TOKEN SECRET'  # replace ACCESS TOKEN SECRET with your access token secret
    }
    yandex_api = 'API KEY'  # replace API KEY with your yandex translation api key

This looks a bit like the variable declaration from the introduction; ``yandex_api`` might look like the *normal* way to
declare a variable. The variable ``twitter`` is of a special type: it is a *dictionary*, or dict for short. You can see
a dictionary as a list with named indices. A dictionary is a special type in different ways: not only can it store
multiple values, but the values can be of different types. It can even store other dictionaries. It is declared as a
comma separated list of key-value pairs between accolades. To declare a key-value pair, specify the name of the key as a
string, add a colon and specify the value, for example like this: ``'name of the key': 'value'``. Remember, the value
does not need to be a string. It can be any type you would like it to be. See the following block for a more advanced
example:

.. code-block:: python

    example = {
        'example_key': 'A string',
        'an_integer_value': 42,
        'dictionary_in_a_dictionary': {
            'another_example_key': 1.5,
            'final_example_key': 'another string'
        }
    }

Dictionaries are especially useful when bundling variables with a similar use, for example a set of API keys for
Twitter. When called, ``example`` returns the complete dictionary. To use only ``an_integer_value`` from this dict,
call it like this: ``example['an_integer_value']``. To access ``another_example_key``, call
``example['dictionary_in_a_dictionary']['another_example_key']``. You can add as much depth to these structures as you
wish.

.. code-block:: python

    auth = tweepy.OAuthHandler(twitter['api_key'], twitter['api_secret'])
    auth.set_access_token(twitter['access_token'], twitter['access_token_secret'])
    api = tweepy.API(auth)

    translate = YandexTranslate(yandex_api)

I skipped a couple of lines, as these are almost identical to the code from the introduction. This part is also pretty
straightforward. As described in the previous block, the values from ``twitter`` can be accessed in the same way as the
values from ``example``. In the last line, we define another variable by calling the ``YandexTranslate`` class with the
translation api key we defined above. I'll explain this in the next post, as this may be quite difficult when never used
before.

.. code-block:: python

    result = api.search(geocode=search_area)
    for tweet in result:
        translation = translate.translate(tweet.text, 'en')
        if translation['code'] == 200:
            print(u'{0}: {1}'.format(tweet.author.name, translation['text'][0]))
        else:
            print('Unable to translate tweet.')

The final part: we do a search on twitter again, however without a search query. By searching like this, we get all
tweets posted from a given search area. This is similar to using the 'near' parameter in the Twitter advanced search.
To translate a snippet of text, pass it as the first argument to the ``translate`` function of the ``translate`` object.
The second argument is the language to translate the text to, in this case English.

Next up, a new structure: the if-else check. If a given condition is true, then the first code block is executed. If the
condition is false, it is not executed. An else block, if given, will only be executed if the condition is false.
The condition is writen out like this: ``variable_1 == variable_2``. Instead of ``==``, other checks can be used: ``!=``
stands for not equal to, ``<=`` stands for smaller than or equals to, ``>=`` stands for greater than or equals to,
``<`` stands for smaller than, ``>`` stands for greater than. Furthermore, ``in`` can be used to check if a given
variable is part of a data structure like a dict, a list, or even part of a string.

The variable ``translation`` is a dictionary. It's key ``code`` describes if Yandex succeeded in translating the text.
Only when the code is 200, a (partial) translation is returned. The Yandex API supports translating multiple texts at
once, however this module does not support that. The translation(s), using the key ``text``, are returned as a list.
Lists do not support naming indices, so the key is just it's position in the list. In most programming languages, Python
included, counting starts from 0 upwards, so the first element from the list is found at position 0. Combining this
information, all translations from ``YandexTranslate`` are returned as ``translation['text'][0]``.

In Python 2.7, a string with a "u" in front of it, is a unicode string. By default, strings only support characters in
the ASCII charset. ASCII is the American Standard Code for Information Interchange and has room for 128 different
characters. A unicode string has a lot more different characters available. When in doubt if a character is usable,
use a unicode string.

Conclusion
----------
In this blogpost, we've seen an example of combining the Twitter API with the Yandex translation API to translate
tweets from a given area. To do so, we've used the for-loop and the if-else structure. Furthermore, we've worked with
dictionaries and lists. This is essential for future programs.

In the next blogpost, I'll be talking about a more difficult subject: object oriented programming, the use of classes
and functions. For this, I'll be using some basic examples about furniture and demonstrate the subject with a program
featuring the Twitter streaming API to fetch tweets real time.