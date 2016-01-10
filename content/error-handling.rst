Error handling and generators with WikiMapia
############################################

:date: 2014-08-24 22:50
:tags: python, wikimapia
:category: python
:slug: error-handling
:authors: Arlena Derksen
:summary: About working with errors and using them for good. This article will also cover the generator function and will use the PyMapia library to retrieve information from WikiMapia.

In this post, I'll cover two themes:

1. How to work with errors and use them for good
2. How to create your own function that returns a value you can walk through in a loop.

In the example covering both of these themes, we will be using `WikiMapia <http://wikimapia.org>`_.

Error handling
--------------
I've mentioned a couple of times, and you might have noticed while experimenting with the code, that unexpected
behaviour of some code parts can trigger errors or so called exceptions. In Python, we're able to catch these errors,
in special *try-except* structures. First, you start a block with the ``try`` statement. All the code enclosed in this
block will run until an error occurs. Next, in a (number of) except blocks, fill in the names of the Errors/Exceptions
you would like to catch. The code in the block following the ``except`` statement will be executed on this particular
error. For example, see the following code:

.. code-block:: python

    try:
        total = 42 / 0
    except ZeroDivisionError:
        print('You can\'t divide by zero!')

Ah, something else is new: a backslash in front of a ``'``, *escapes* it. This way, you can use it in a string without
closing it. In Python, we have the following set of characters (and a couple more) that can be created using escaping:

* ``\n`` - Line Feed (new line, Linux/Mac OS X)
* ``\r`` - Carriage return (move to the first position of the line); combine ``\r`` and ``\n`` to create a new line on
  Windows systems
* ``\'`` - escape the ``'`` in strings
* ``\"`` - escape the ``"`` in strings
* ``\\`` - escape the ``\`` in strings

To ignore when errors occur, use the ``pass`` statement. We can also call this the do-nothing-statement. It can be used
in all code blocks that I know of. You can even create empty classes, methods and functions with it. In a try-except
block, it can look like this:

.. code-block:: python

    try:
        example = {
            'a': 1,
            'b': 2,
            'c': 3
        }
        print(example['d'])
    except KeyError:
        # do nothing
        pass

Apart from getting errors from mistakes, exceptions can be caused by dynamic data. For example, if you try to append a
unicode string to a normal string, but you don't know it is a unicode string (for example, the text of a tweet), it will
trigger an exception. You can also manually trigger an exception with the ``raise`` statement. This can be useful when
creating your own exceptions and redirecting other errors to your own exception. See the following code for an example:

.. code-block:: python

    try:
        print('Start')
        raise ValueError('This will trigger a ValueError')
    except ValueError as error:
        print('Error occurred: {0}'.format(error))
    finally:
        print('End)

The previous code block had another thing in it: the ``finally`` statement. This statement is executed after the try
and will **always** be executed even when an error occurred.

Generators
----------
A generator is a function that can be used like an iterator in a for loop. We can already do that by returning a list,
but this is not the most efficient way to do. Furthermore, when we create a list, it builds in the memory of the
computer. When these lists become bigger and bigger, more memory will be used. It is safer to use a generator.
A generator *yields* items instead of a complete list. A good and simple example of a generator can be given using the
mathematical Fibonacci sequence. This sequence starts with 0, 1, 1, 2, 3, 5, 8, 13, etc. We have a value ``a`` and a
value ``b``; a starts at 0, b starts at 1. The first item in the sequence is a, the second one is b, the third is the
sum of the first and second element of the sequence. The fourth item is the sum of the second and third item. I think
you can understand how the rest of this sequence is formed.

Back to the generator: according to the Python glossary, part of the documentation, we can define a generator as *a
function which returns an iterator. It looks like a normal function except that it contains ``yield`` statements for
producing a series a values usable in a for-loop or that can be retrieved one at a time with the ``next()`` function.
Each yield temporarily suspends processing, remembering the location execution state (including local variables and
pending try-statements). When the generator resumes, it picks-up where it left-off (in contrast to functions which
start fresh on every invocation).* This is a more technical description, but might describe it better than I'm able to
do. We can work out the fibonacci example with the following piece of code:

.. code-block:: python

    def fibonacci():
        a = 0
        b = 1
        while True:
            yield a
            a = b
            b = b + a

To test it, you can use the following snippet:

.. code-block:: python

    for value in fibonacci():
        print(value)
        if value > 100:
            raise Exception('An easy way to stop the program.')

Practical example
-----------------
In the following example, we will make a list of all mosques in a radius of 15 kilometers from the center of Aleppo,
Syria. This list will contain the name of the mosque, the geographical location and all available photos, if any. We
will be using this information in a future post to create a database we can use when geolocating photo and video from
inside Aleppo. For this program, we need the pymapia library. I wrote this library about a month ago and released it.
You can install it via the normal way: go to settings -> python interpreter -> Anaconda -> '+' -> search for PyMapia
and install it. This post was written for version 0.1.1, the last version available when writing this post, however
I'm planning to update it within a short time. I'll try to update this post when I change things to the library.

.. code-block:: python

    from pymapia import PyMapia

    wikimapia_api_key = 'API KEY'  # Replace API KEY by your WikiMapia API Key

    lat = 36.229874  # latitude of the center as a float
    lon = 37.170181  # longitude of the center as a float
    search_radius = 15000.0  # distance in meters as a float
    category = ['Mosque']
    query = ''

    wikimapia = PyMapia(wikimapia_api_key)


    def wikimapia_generator():
        total = int(wikimapia.search_place(lat=lat, lon=lon, category=category, query=query,
                                           distance=search_radius)['found'])
        page_size = 100
        total_pages = total / page_size + 1
        page = 1
        while page <= total_pages:
            result = wikimapia.search_place(lat=lat, lon=lon, category=category, query=query, distance=search_radius,
                                            count=page_size, page=page)
            for place in result['places']:
                yield place
            page += 1

    for location in wikimapia_generator():
        print(location['title'])
        try:
            print(location['location']['gadm'][-1]['lat'], location['location']['gadm'][-1]['lon'])
        except KeyError:
            print(location['location']['lat'], location['location']['lon'])

        if location['photos'] is not list():
            for photo in location['photos']:
                print(photo['big_url'])

        print('')

Like all of the previous examples, we will walk through this program line by line.

.. code-block:: python

    from pymapia import PyMapia

The PyMapia library in it's current form contains only one class, the PyMapia class. Still, we import this one from
``pymapia``, so we can write ``PyMapia()`` instead of ``pymapia.PyMapia()`` for easier readability.

.. code-block:: python

    wikimapia_api_key = 'API KEY'  # Replace API KEY by your WikiMapia API Key

    lat = 36.229874  # latitude of the center as a float
    lon = 37.170181  # longitude of the center as a float
    search_radius = 15000.0  # distance in meters as a float
    category = ['Mosque']
    query = ''

The variable declaration: first, we declare the wikimapia API key. This key is obtained by registering at WikiMapia,
logging in and creating a key at the `key creation page <http://wikimapia.org/api/?action=create_key>`_. Next, we notate
the latitude and the longitude of the center of the area we are looking at, followed by the search radius in meters as a
float. Note that the system is not actually this accurate, but the API does not accept integers. The same applies for
the latitude and the longitude: specify them as a float, or the program might return strange results. Categories are
specified as a list with AND logic. This means that all categories specified in list must match the object. For
example, if both *Campsite* and *Park* are given, only objects which are both known as a campsite and as a park will be
returned. WikiMapia supports OR logic, so campsite or park, as well, but using a different name. I'll mention this
below. Finally, we can specify a query. The API expects that we do a search as well, however, it can be an empty search.
As we want all mosques in Aleppo returned, we will be using an empty search query.

.. code-block:: python

    wikimapia = PyMapia(wikimapia_api_key)

To create an object of the PyMapia class, call it's constructor with the wikimapia API key. We will be using this object
to perform searches on WikiMapia.

.. code-block:: python

    def wikimapia_generator():
        total = int(wikimapia.search_place(lat=lat, lon=lon, category=category, query=query,
                                           distance=search_radius)['found'])
        page_size = 100
        total_pages = total / page_size + 1
        page = 1
        while page <= total_pages:
            result = wikimapia.search_place(lat=lat, lon=lon, category=category, query=query, distance=search_radius,
                                            count=page_size, page=page)
            for place in result['places']:
                yield place
            page += 1

Next, the wikimapia_generator function. This function works as a generator, like described above. First, we need to find
the total number of places found, as wikimapia works with pages with a maximum of 100 items per page. The method
``wikimapia.search_place()`` returns a dictionary with the results of the search. One of the keys in this dictionary is
the value ``found``. This value contains the total number of places found in the search. While writing the test cases
for the pymapia library, I noticed that several of the returned values are not of the type you might expect. This found
value is a unicode string. Because we need the total as an integer, we have to cast it to this using the ``int()``
function. The maximum page size is 100 and to use the minimum amount of calls to the API, it is the best to use 100 as
the page size.

The total number of pages is defined by dividing the total number of locations by the page size and adding 1. We need to
add 1 because of the way integers work. Dividing an integer by an integer, results in an integer value. As integers are
whole numbers, without a decimal value, everything behind the decimal point is not stored. For example, ``5 / 2``
gives 2 as a result, not 2.5 as you might have expected.

Next, the *while* structure. While is similar to for, it starts a loop that keeps running while the condition is true.
Every page again, the same structure is repeated. We search with the same parameters as before, however now we add the
``count``, the number of results per page, and the ``page``, the page we're on. As I mentioned above, WikiMapia supports
OR logic. To use OR logic, use ``categories_or`` instead of ``categories``.

The variable ``result['places']`` is a list containing dictionaries of places. This method yields a place every time
it runs. We must not forget to increase the page number, otherwise the while loop will never end.

.. code-block:: python

    for location in wikimapia_generator():
        print(location['title'])
        try:
            print(location['location']['gadm'][-1]['lat'], location['location']['gadm'][-1]['lon'])
        except KeyError:
            print(location['location']['lat'], location['location']['lon'])

        if location['photos'] is not list():
            for photo in location['photos']:
                print(photo['big_url'])

        print('')

We create a for loop using our custom generator function. When compared to the previous block, ``location`` matches
``place``. First, we print the title of the location. Next, we place a try-except structure. The
``location['location']`` contains a dictionary, however if there is only one location specified, a more specific
location is put in this dictionary, so we would be able to directly use the ``lat`` and ``lon`` keys. However, in some
cases, there are more locations known, for example country -> state -> city -> neighbourhood -> street -> building. In
this case, ``location['location']`` contains a dictionary with the key ``gadm``. This key contains a list of locations.
In all cases, we want the most specific location, which is the last item of the list. In Python, we do not have to move
forwards in a list. We can also move backwards, so the index -1 is the last item of the list, -2 the one before the last
item of the list. All items in the list are dictionaries as well. These contain the keys ``lat`` and ``lon``.

In the first part of the try-except structure, we try to print a location in the long format, i.e. we assume a location
is written in the long format. If the key ``gadm`` does not exists, i.e. where a location is only available in the
short format, it does trigger a ``KeyError``. When we then catch a ``KeyError`` and print the short format otherwise, we
can make sure both cases do work without crashing the program.

Next, we check if the ``location['photos']`` is an empty list. If it is not an empty list, all hyperlinks to photos will
be printed under the location. Finally, we print a blank line to separate the locations.
A snippet of the results: ::

    Ummayad Mosque of Aleppo
    (36.1994591, 37.1569035)
    http://photos.wikimapia.org/p/00/02/39/31/59_big.jpg
    http://photos.wikimapia.org/p/00/02/39/31/60_big.jpg
    http://photos.wikimapia.org/p/00/02/39/31/61_big.jpg
    http://photos.wikimapia.org/p/00/02/39/31/63_big.jpg
    http://photos.wikimapia.org/p/00/03/08/27/20_big.jpg
    http://photos.wikimapia.org/p/00/03/08/27/22_big.jpg
    http://photos.wikimapia.org/p/00/03/08/27/24_big.jpg

    Al-Halawiyeh Madrasa (Previously Old Cathedral Of Aleppo)
    (36.1994191, 37.155995)

    Masjid _____
    (36.2045887, 37.1964521)

Conclusion
----------
In the previous post, we've used errors to make code work, instead of crash. Furthermore, we've used a generator to
retrieve a large list as separate items. In this post I wrote about using WikiMapia to retrieve information about
mosques in Aleppo. Some of these results had a photo included, but many did not. In the next post, we will use the
'old' data API from `Panoramio <http://www.panoramio.com>`_. This API has access to a lot more photos combined with the
location where it was photographed. Combining that API with this example, will give us a lot more photographic details
to use in a database, to give better results in the end when geolocating.
