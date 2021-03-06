ogdf-python: Automagic Python Bindings for the Open Graph Drawing Framework
===========================================================================

`Original repository <https://github.com/N-Coder/ogdf-python>`_ (GitHub) -
`Bugtracker and issues <https://github.com/N-Coder/ogdf-python>`_ (GitHub) -
`PyPi package <https://pypi.python.org/pypi/ogdf-python>`_ (PyPi ``ogdf-python``) -
`Documentation <https://ogdf-python.readthedocs.io>`_ (Read The Docs).

`Official OGDF website <https://ogdf.net>`_ (ogdf.net) -
`Public OGDF repository <https://github.com/ogdf/ogdf>`_ (GitHub) -
`Internal OGDF repository <https://git.tcs.uos.de/ogdf-devs/OGDF>`_ (GitLab) -
`OGDF Documentation <https://ogdf.github.io/docs/ogdf/>`_ (GitHub / Doxygen) -
`cppyy Documentation <https://cppyy.readthedocs.io>`_ (Read The Docs).

.. |(TM)| unicode:: U+2122

``ogdf-python`` uses the `black magic <http://www.camillescott.org/2019/04/11/cmake-cppyy/>`_
of the awesome `cppyy <https://bitbucket.org/wlav/cppyy/src/master/>`_ library to automagically generate python bindings
for the C++ Open Graph Drawing Framework (OGDF).
It is available for Python>=3.6 and is Apache2 licensed.
There are no binding definitions files, no stuff that needs extra compiling, it just works\ |(TM)|, believe me.
Templates, namespaces, cross-language callbacks and inheritance, pythonic iterators and generators, it's all there.
If you want to learn more about the magic behind the curtains, read `this article <http://www.camillescott.org/2019/04/11/cmake-cppyy/>`_.


Quickstart
----------

First, install the package ``ogdf-python`` package.
Please note that building ``cppyy`` from sources may take a while.
If you didn't install OGDF globally on your system,
either set the ``OGDF_INSTALL_DIR`` to the prefix you configured in ``cmake``,
or set ``OGDF_BUILD_DIR`` to the subdirectory of your copy of the OGDF repo where your
`out-of-source build <https://ogdf.github.io/docs/ogdf/md_doc_build.html#autotoc_md4>`_ lives.

.. code-block:: bash

    $ pip install ogdf-python
    $ OGDF_BUILD_DIR=~/ogdf/build-debug python3

Alternatively, ``ogdf-python`` works very well with Jupyter:

.. code-block:: python

    %env OGDF_BUILD_DIR=~/ogdf/build-debug

    from ogdf_python import ogdf, cppinclude
    cppinclude("ogdf/basic/graph_generators/randomized.h")
    cppinclude("ogdf/layered/SugiyamaLayout.h")

    G = ogdf.Graph()
    ogdf.setSeed(1)
    ogdf.randomPlanarTriconnectedGraph(G, 20, 40)
    GA = ogdf.GraphAttributes(G, ogdf.GraphAttributes.all)

    for n in G.nodes:
        GA.label[n] = "N%s" % n.index()

    SL = ogdf.SugiyamaLayout()
    SL.call(GA)
    GA

.. image:: docs/examples/sugiyama-simple.svg
    :target: docs/examples/sugiyama-simple.ipynb
    :alt: SugiyamaLayouted Graph
    :height: 300 px

Read the pitfalls section and check out `docs/examples/pitfalls.ipynb <docs/examples/pitfalls.ipynb>`_
for the more advanced Sugiyama example from the OGDF docs.
There is also a bigger example in `docs/examples/ogdf-includes.ipynb <docs/examples/ogdf-includes.ipynb>`_.
If anything is unclear, check out the python help ``help(ogdf.Graph)`` and read the corresponding OGDF documentation.

..
    Documentation
    -------------
    TODO use ``help(ogdf.Graph)`` and sphinx autodoc

..
    Load Custom Code
    ----------------
    TODO

Pitfalls
--------

See also `docs/examples/pitfalls.ipynb <docs/examples/pitfalls.ipynb>`_ for full examples.

OGDF sometimes takes ownership of objects (usually when they are passed as modules),
which may conflict with the automatic cppyy garbage collection.
Set ``__python_owns__ = False`` on those objects to tell cppyy that those objects
don't need to be garbage collected, but will be cleaned up from the C++ side.

.. code-block:: python

    SL = ogdf.SugiyamaLayout()
    ohl = ogdf.OptimalHierarchyLayout()
    ohl.__python_owns__ = False
    SL.setLayout(ohl)

When you overwrite a python variable pointing to a C++ object (and it is the only
python variable pointing to that object), the C++ will usually be immediately deleted.
This might be a problem if another C++ objects depends ob that old object, e.g.
a ``GraphAttributes`` instance depending on a ``Graph`` instance.
Now the other C++ object has a pointer to a delted and now invalid location,
which will usually cause issues down the road (e.g. when the dependant object is
deleted and wants to deregister from its no longer alive parent).
This overwriting might easily happen if you run a Jupyter cell multiple times.
Please ensure that you always overwrite or delete C++ dependent variables in
the reverse order of their initialization.

.. code-block:: python

    CGA = CG = G = None
    G = ogdf.Graph()
    CG = ogdf.ClusterGraph(G)
    CGA = ogdf.ClusterGraphAttributes(CG, ogdf.ClusterGraphAttributes.all)

