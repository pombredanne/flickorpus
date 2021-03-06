flickorpus
==========

flickorpus collects an image and tag corpus from flickr.
For different tag queries, it finds the N most relevant images, and retrieves
the tags for those images.
It never actually downloads images themselves, it merely has sufficient
metadata to concoct the correct URL.

Written by Joseph Turian
    <lastname at gmail dot com>

Currently available at:
    http://github.com/turian/flickorpus/tree

LICENSE
-------
MIT license. See LICENSE.txt

If you collect good data with this program, we encourage you to pay
it forward and share with others.


REQUIREMENTS
------------

* Python

* Beej's Python Flickr API: <http://flickrapi.sourceforge.net/>

* ZODB: <http://wiki.zope.org/ZODB/FrontPage>

* Our python common library, in your PYTHONPATH:
    git clone git://github.com/turian/common.git
If you download 'common' into directory DIR, you have to put DIR in
your PYTHONPATH:
    export PYTHONPATH=$PYTHONPATH:$DIR


GETTING STARTED
---------------

* Get the source code, in a location present in your PYTHONPATH:
    hg clone http://pylearn.org/hg/flickorpus

* Install Beej's Python Flickr API:
    easy_install flickapr

* Copy keys.py.tmpl to keys.py, and enter your flickr API keys.

* ZODB:
    * Install ZODB. For example:
        easy_install ZODB3
        mkdir db/

    * If you want only one local ZODB instance (simpler):
        * Copy flickr-local.conf to flickr.conf
    * Or, if you want to allow multiple clients to save data to your
    ZODB instance (more flexible):
        * Get a ZODB server running, e.g. copy zeo.conf.tmpl to zeo.conf,
        (optionally) edit zeo.conf and zeo.sh, and run
            ./zeo.sh
        If you like, you can edit zeo.sh to explicitly specify the hostname.

        * Copy flickr.conf.tmpl to flickr.conf, and edit the HOSTNAME to match the
        host on which the ZEO server was launched.

* Run ./searchspider.py to begin accumulating a corpus.
If you do not see any progress within a few seconds, and you are using the
client/server ZODB approach (above), try using the local ZODB instance.

* Run ./dumpdatabase.py to dump the database in a JSON format.

SOME DETAILS
------------

ZODB gives the program persistent state, so you can start and stop
execution as you please. Work will continue where you left off.

Flickr API queries are cached in ZODB. Note that search results may get
stale over time.

Flickr typically throttles you to two requests per second.

It wouldn't be too hard to extend this program to operate concurrently.
I have not done so, though. Email me for more details.

./searchspider.py
    The search spider begins with the seed queries in myflickr.seed_queries.

    For each query, it searches for the 1000 (= myflickr.total) most relevant
    results with this tag. This requires 2 = 1000/500 (= myflickr.total /
    myflickr.queries_per_page) API requests.
    For each of the 1000 results, it retrieves their metadata. In particular,
    it retrieves every tag for the image. If an image ID was returned previously
    in another query, then its metadata is already cached and does not need to
    be retrieved. Otherwise, an API request is issued.

    After all seed queries have been pulled, the spider goes over the entire
    database of stored image metadata. It determines the most common tag that
    has not already been queried and that is not present in
    myflickr.blacklist_queries. It then queries this tag.

    Lather, rinse repeat.

./getresults.py tag
    Search for the 1000 (= myflickr.total) most relevant results with
    this tag, and output the 75x75 images in HTML.

./tags.py
    For every image in the ZODB, retrieve its tags and output them.

BUGS
----

Doomie found a bug in the database initialization:
"So I managed to quickly hack a fix to the problem by simply initializing the
dictionary of objects in the DB. Wonder if there's a cleverer fix for that :)
"

COMMENTS, QUESTIONS, ETC.
-------------------------

Please email the author.
