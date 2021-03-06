=================
Tornado Testlayer
=================

This is a simple testlayer that can be used to test tornado web applications
using the `zope testrunner`_.

First lets create a simple sample application::

    >>> from tornado.web import Application, RequestHandler
    >>> class HelloWorldHandler(RequestHandler):
    ...     def get(self):
    ...         return self.write('Hello World')

Importing the layer::

    >>> from tornado_testing.layer import TornadoLayer

Create the layer using the sample application created earlier::

    >>> my_app = Application(handlers=[(r'/', HelloWorldHandler)])
    >>> tornado_layer = TornadoLayer(my_app, port=None)
    >>> type(tornado_layer.port)
    <class 'int'>

.. note::

    If the port is None the next free port will be used.

Set up the layer::

    >>> tornado_layer.setUp()

Now the layer should be running and therefore tornado will serve requests::

    >>> from urllib.request import urlopen
    >>> url = 'http://localhost:{}/'.format(tornado_layer.port)
    >>> f = urlopen(url)
    >>> f.read()
    b'Hello World'

    >>> f.close()
    >>> tornado_layer.tearDown()


Patches
=======

The testlayer supports patches which can be used to mock out any modules that
are used inside a tornado application.

Here is an example of a tornado application that uses the ``os`` modules
``listdir`` method::

    >>> import json
    >>> class ListDirHandler(RequestHandler):
    ...     def get(self):
    ...         import os
    ...         return self.write(json.dumps(os.listdir('.')))

    >>> my_app = Application(handlers=[(r'/', ListDirHandler)], debug=True)

.. note::

    Usually the import also works outside the method on module level. But since
    this example is executable using doctest it had to be put inside the method
    as otherwise the patch wouldn't work.

Now define the fake ``listdir`` method::

    >>> def fake_listdir(path):
    ...     return ['fake_file']

Create the monkey patch::

    >>> from unittest.mock import patch
    >>> listdir_patch = patch('os.listdir', fake_listdir)

Create the test layer using the patch::

    >>> tornado_layer = TornadoLayer(my_app, port=None, patches=[listdir_patch])
    >>> tornado_layer.setUp()

    >>> url = 'http://localhost:{}/'.format(tornado_layer.port)
    >>> f = urlopen(url)
    >>> f.read()
    b'["fake_file"]'

    >>> f.close()
    >>> tornado_layer.tearDown()


.. _`zope testrunner`: https://pypi.python.org/pypi/zope.testrunner
