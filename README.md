buildout.zrsexample
====================

Example buildout with ZRS, HAProxy and Varnish

Background
----------------

Recently [ZC](http://www.zope.com) released
[ZRS](https://pypi.python.org/pypi/zc.zrs), their ZEO replication service,
under a [ZPL license](http://foundation.zope.org/agreements/ZPL_2.1.pdf).

Soon after a new release of [plone.recipe.zeoserver](https://pypi.python.org/pypi/plone.recipe.zeoserver)
was prepared with a great feature: support for zc.zrs. As noted by
[Steve McMahon](http://www.stevemcmahon.com/steves-blog/plone-adds-replication-in-micro-release)
this was a very big feature for a small release, because not only provides
a way to backup your ZEO server live, but also provides a way to create
extra ZEO servers and they can also be configured to serve content.


Example configuration with buildout
-----------------------------------------

This buildout shows an example to create such a configuration. It also provides
configuration for HAProxy and Varnish if you want to serve everything
from one server.

To get this configuration running, just execute this buildout with python 2.7 and
start the supervisor:


    $ python2.7 bootstrap.py
    $ ./bin/buildout -vv
    $ ./bin/supervisord


It will create a configuration like this one:

    -----------------------------> Instance1 (:8081) <--> ZEO1 (:8091, replicate in :8070)
                                                                         |        |
                                                                         |        |
        Varnish --> HAProxy -----> Instance2* (:8082) <--> ZEO2* (:8092)--        |
        (:8000)     (:8080)  |                                                    |
                             |                                                    |
                             |---> Instance3* (:8083) <--> ZEO3* (:8093)----------

     Instances with * are set to be read-only



### What do you get with this?


With this configuration you have a common Zope instance with a ZEO server to
do your content-editing work. This happens in the 8081 port instance.

But, for free, you also get 2 more instances, balanced using HAProxy and cached
in Varnish, configured in read-only and each one with their own ZEO instance
which gets the data from the first ZEO server. But this instances are configured
in read-only mode, so you can't write on them. If your site just has to serve
content, it can work in read-only mode and you get that the editing and content
viewing don't happen at the same time on the same server.

You will need to configure your webserver to serve the first instance in another
domain (for instance edit.yourserver.com) and server the main domain (www.yourserver.com)
through Varnish. Varnish is not mandatory, you can just connect you webserver
to HAProxy.

If you are using Nginx, you can adapt [this configuration](http://www.starzel.de/blog/securing-plone-sites-with-https-and-nginx)
prepared by [Philip Bauer](https://twitter.com/StarzelDe) which originaly was
created to put all logged-in traffic under HTTPS- You can easily adapt it to put
all your logged-in traffic to 8081 port and the not-logged one to 8000
(or 8080 if you want to avoid Varnish)

Further steps
--------------

You can easily add new instances connected to ZEO2 and ZEO3, just copy the
buildout parts, and adjust the conf/haproxy.conf.in file to add more instances
to the load-balancing process.
