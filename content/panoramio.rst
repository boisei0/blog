Panoramio: how to use it's data API to gather photos from a given area
######################################################################

:date: 2014-08-26 15:15
:tags: python, panoramio
:category: python
:slug: pynoramio
:authors: Arlena Derksen
:summary: A quick overview of the Pynoramio library: a library written to use with the Data API of Panoramio.

I encourage you all to create your own, better version of Pynoramio. Pynoramio is a module/library to talk with the
Panoramio data API. I created it while writing this post and I think it can be optimized. This post will be a bit
different from the ones I wrote before. I'll move to the code straight away. This post won't contain as much theory
as the previous posts, but is mainly a description of how to read modules, talk to websites and read API documentation.

The full code of the Pynoramio module is available on
`gist.github.com <https://gist.github.com/boisei0/03669ef6f086b7ff09b3>`_. I'll put it here as well, however I'm pretty
sure the line lengths will not work correctly on most screens.

Pynoramio
---------
.. code-block:: python

    import requests


    class PynoramioException(Exception):
        pass


    class Pynoramio:
        def __init__(self):
            self.base_url = 'http://www.panoramio.com/map/get_panoramas?order=popularity'

        def _request(self, lat_min, lon_min, lat_max, lon_max, start, end, picture_size=None, set_=None,
                     map_filter=None):
            if not isinstance(lat_min, float):
                raise PynoramioException(
                    '{0}._request requires the lat_min parameter to be a float.'.format(self.__class__.__name__))
            if not isinstance(lon_min, float):
                raise PynoramioException(
                    '{0}._request requires the lon_min parameter to be a float.'.format(self.__class__.__name__))
            if not isinstance(lat_max, float):
                raise PynoramioException(
                    '{0}._request requires the lat_max parameter to be a float.'.format(self.__class__.__name__))
            if not isinstance(lon_max, float):
                raise PynoramioException(
                    '{0}._request requires the lon_max parameter to be a float.'.format(self.__class__.__name__))

            if not isinstance(start, int):
                raise PynoramioException(
                    '{0}._request requires the start parameter to be an int.'.format(self.__class__.__name__))
            if not isinstance(end, int):
                raise PynoramioException(
                    '{0}._request requires the end parameter to be an int.'.format(self.__class__.__name__))

            url = self.base_url + '&minx={0}&miny={1}&maxx={2}&maxy={3}&from={4}&to={5}'.format(lon_min, lat_min,
                                                                                                lon_max, lat_max,
                                                                                                start, end)

            if picture_size is not None and isinstance(picture_size, basestring) \
                    and picture_size in ['original', 'medium', 'small', 'thumbnail', 'square', 'mini_square']:
                url += '&size={0}'.format(picture_size)

            if set_ is not None and (isinstance(set_, basestring) and set_ in ['public', 'full']) \
                    or (isinstance(set_, int)):
                url += '&set={0}'.format(set_)
            else:
                url += '&set=public'

            if map_filter is not None and isinstance(map_filter, bool) and not map_filter:
                url += '&map_filter=false'

            r = requests.get(url)
            try:
                return r.json()
            except ValueError:
                # add your debugging lines here, for example, print(r.url)
                raise PynoramioException(
                    'An invalid or malformed url was passed to {0}._request'.format(self.__class__.__name__))

        def get_from_area(self, lat_min, lon_min, lat_max, lon_max, picture_size=None, set_=None, map_filter=None):
            page_size = 100
            page = 0

            result = self._request(lat_min, lon_min, lat_max, lon_max, page * page_size, (page + 1) * page_size,
                                   picture_size, set_, map_filter)

            total_photos = result['count']
            if total_photos < page_size:
                return result

            page += 1

            pages = (total_photos / page_size) + 1
            while page < pages:
                new_result = self._request(lat_min, lon_min, lat_max, lon_max, page * page_size, (page + 1) * page_size,
                                           picture_size, set_, map_filter)

                result['photos'].extend(new_result['photos'])

                page += 1

            return result

        def get_all_pictures_cursor(self, lat_min, lon_min, lat_max, lon_max, picture_size=None, set_=None,
                                    map_filter=None):
            page_size = 100
            page = 0

            result = self._request(lat_min, lon_min, lat_max, lon_max, page * page_size, (page + 1) * page_size,
                                   picture_size, set_, map_filter)

            total_photos = result['count']

            for photo in result['photos']:
                yield photo

            if total_photos < page_size:
                raise StopIteration()

            page += 1

            pages = (total_photos / page_size) + 1
            while page < pages:
                result = self._request(lat_min, lon_min, lat_max, lon_max, page * page_size, (page + 1) * page_size,
                                       picture_size, set_, map_filter)

                for photo in result['photos']:
                    yield photo

                page += 1

            raise StopIteration()

    __all__ = ['Pynoramio', 'PynoramioException']

This module was written to use with the 'long lost' Panoramio Data API. In June 2008, Google first wrote about this data
API on their
`Geo Developers Blog <http://www.googlegeodevelopers.blogspot.nl/2008/06/panoramio-api-markermanager-instant.html>`_.
The link on that page for the API documentation now links to the Widget API documentation. The documentation for the
Data API is still available, however the page is not linked to anymore. This documentation is available at
`http://www.panoramio.com/api/data/api.html <http://www.panoramio.com/api/data/api.html>`_.

I did not know of the existence of the Data API. I found it 'by accident' while trying to find out how the system works.
At first, I thought it was an internal API, to communicate between the server and the interface. I'm very glad it is in
fact not an internal system, because now I'm able to write about it. If it was the internal API, it would violate
Google's Terms of Service to use it. It would allow me however to write about it, but the reader was not allowed to use
it. I even contacted a legal expert to make sure I would not violate the Terms of Service by writing this post. Now that
the API turns out to be an actual Panoramio API, I can write about this subject without problems.

The data API
------------
As far as I know, there are no existing Python libraries that communicate with the Panoramio Data API. If you would like
to use this API, you'll have to write your own communication system. To communicate with web pages, we will use the
``requests`` library. According to the documentation, you only have to do a *GET* to a link in the following format:
``http://www.panoramio.com/map/get_panoramas.php?set=public&from=0&to=20&minx=-180&miny=-90&maxx=180&maxy=90&size=medium&mapfilter=true``.
In the documentation, they describe that the result data is formatted using *JSON*. JSON is a format very similar to the
dictionaries in Python. The requests library is able to *parse* JSON formatted text to a dictionary. To do a request on
Panoramio, import the requests module, create your link, also known as Uniform Resource Locator (URL) and do a
get-request on the given url. On the resulting object, call the json() function. In code:

.. code-block:: python

    import requests

    url = 'http://www.panoramio.com/map/get_panoramas.php?set=public&from=0&to=20' +
          '&minx=-180&miny=-90&maxx=180&maxy=90&size=medium&mapfilter=true'
    response = requests.get(url)
    panoramio_dictionary = response.json()

Back to Pynoramio
-----------------
.. code-block:: python

    class PynoramioException(Exception):
        pass

For this library, we first create a custom exception, which we will use later to catch other errors and re-raise them
as this new, custom exception. To create a custom exception, just make a new class and let it override exception. It can
be empty.

The constructor of the class Pynoramio contains only one declaration. The base url variable is the start of each link
that is the same for every possible request. We will use that variable when building together a url for the request. The
``_request`` method is a so called private method. It should only be used within the class. In Python, you can recognize
a private method because the name starts with one underscore.

.. code-block:: python

    if not isinstance(lat_min, float):
        raise PynoramioException(
            '{0}._request requires the lat_min parameter to be a float.'.format(self.__class__.__name__))

The previous code block is repeated several times in the ``_request`` method with only a small variation. First, it
checks if the specified parameter, in this case ``lat_min`` is of the type ``float``. If not, it will raise our custom
PynoramioException with the error message that the ``_request`` method requires the specific parameter to be of the type
float. The variable ``self.__class__.__name__`` is a special kind of variable. It contains the name of the class, in
this case ``Pynoramio``. When the url is complete, a request will be send to the page to retrieve it. This is enclosed
in a try-except structure. The method ``json()`` will trigger an error when the data is not in the correct format. In
most cases it is a website trying to display an error message instead of the json. It will raise a ``ValueError``
exception.

The ``get_from_area`` method is used to retrieve the full dataset of a search query. To decrease the load on the server,
we use the maximum page size of 100 items per request. We start with pulling the first page in. From the result, we
check the amount of found results. If it is lesser than the pagesize, we return the results. Otherwise, we start a loop
until all pages are pulled in. The ``photos`` key of the dictionary contains a list of dictionaries. To append an item
to a list, we can use the ``append(item)`` function on the original list, however in this case we would like to extend
the list with another list. To do so, use the ``extend(other_list)`` on the original list. When the end of the loop is
reached, return the result.

The ``get_all_pictures_cursor`` method is very similar to the ``get_from_area`` method, however it works as a generator.
Instead of returning the full dataset, it yields the individual photo dictionaries. When the end of the dataset is
reached, the ``StopIteration`` exception is raised. This is a special exception, to stop the generator.

Example usage
-------------
The following example can be used to test the Pynoramio module. Import the module, specify a location, initialize the
Pynoramio class and loop through the results, printing general information about each photo. A more extensive example
will be covered in the next post.

.. code-block:: python

    from pynoramio import Pynoramio

    location = [
        37.635040,  # Bottom left longitude
        47.917262,  # Bottom left latitude
        37.964630,  # Top right longitude
        48.126684  # Top right latitude
    ]

    data_api = Pynoramio()

    for photo in data_api.get_all_pictures_cursor(location[1], location[0], location[3], location[2]):
        print(photo['photo_title'])
        print(photo['latitude'], photo['longitude'])
        print(photo['upload_date'])
        print('')

Conclusion
----------
This was quite a bit of information on Panoramio and the Pynoramio library. I don't expect that you are now able to
create your own library of a subject or system. However, you might now be able to read it and find information you need
when, for example with the Twitter Streaming API and Tweepy, the documentation on a part of the library is non-existent.
In the next post, I'll give an introduction to the subject of databases. We will then combine the information from the
`WikiMapia post <http://hubsec.eu/blog/error-handling.html>`_ with this post to create a database with information about
memorable places in a given area with their coordinates and photos. I'm not sure which location I will use in the
example, **but you can use this on every location you would like to**.
