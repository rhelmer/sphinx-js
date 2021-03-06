=========
sphinx-js
=========

Why
===

When you write a JavaScript library, how do you explain it to people? If it's a small project in a domain your users are familiar with, JSDoc's hierarchal list of routines might suffice. But what about for larger projects? How can you intersperse prose with your API docs without having to copy and paste things?

sphinx-js lets you use the industry-leading Sphinx documentation tool with JS projects. It provides a handful of directives, patterned after the Python-centric autodoc ones, for pulling JSDoc-formatted function and class documentation into reStructuredText pages. And, because you can keep using JSDoc in your code, you remain compatible with the rest of your JS tooling, like Google's Closure Compiler.

Setup
=====

1. Install JSDoc using npm. ``jsdoc`` must be on your ``$PATH``, so you might want to ``npm install -g jsdoc``. We're known to work with jsdoc 3.4.3.
2. Install Sphinx. (TODO: Make this more explicit for non-Python people.)
3. Make a documentation folder in your project using ``sphinx-quickstart``.
4. Add ``sphinx_js`` to ``extensions`` in the generated Sphinx conf.py.
5. Add ``js_source_path = '../somewhere/else'`` to conf.py, assuming the root
   of your JS source tree is at that path, relative to conf.py itself. The
   default is ``../``, which works well when there is a ``docs`` folder at the
   root of your project.
6. If you have special jsdoc configuration, add ``jsdoc_config_path = '../conf.json'`` (for example) to conf.py as well.

Use
===

In short, use the directives below, then build your Sphinx docs as usual by running ``make html`` in your docs directory.

autofunction
------------

Document your JS code using standard JSDoc formatting::

    /**
     * Return the ratio of the inline text length of the links in an element to
     * the inline text length of the entire element.
     *
     * @param {Node} node - Types or not: either works.
     * @throws {PartyError|Hearty} Multiple types work fine.
     * @returns {Number} Types and descriptions are both supported.
     */
    function linkDensity(node) {
        const length = node.flavors.get('paragraphish').inlineLength;
        const lengthWithoutLinks = inlineTextLength(node.element,
                                                    element => element.tagName !== 'A');
        return (length - lengthWithoutLinks) / length;
    }

Our directives work much like Sphinx's standard `autodoc
<http://www.sphinx-doc.org/en/latest/ext/autodoc.html>`_ ones. You can specify
just a function::

    .. js:autofunction:: someFunction

Or you can throw in your own explicit parameter list, if you want to note
optional parameters::

    .. js:autofunction:: someFunction(foo, bar[, baz])

You can even add additional content. If you do, it will appear just below any
extracted documentation::

    .. js:autofunction:: someFunction

        Here are some things that will appear...

        * Below
        * The
        * Extracted
        * Docs

        Enjoy!

Use `JSDoc namepath syntax <http://usejsdoc.org/about-namepaths.html>`_ to disambiguate same-named entities::

    .. js:autofunction:: SomeClass#someInstanceMethod

Behind the scenes, sphinx-js will changes those to dotted names so that...

* Sphinx's "shortening" syntax works: ``:func:`~InwardRhs.atMost``` prints as merely ``atMost()``. (For now, you should always use dots rather than other namepath separators: ``#~``.)
* Sphinx indexes more informatively, saying methods belong to their classes.

To save some keystrokes, you can set ``primary_domain = 'js'`` in conf.py and then say simply ``autofunction`` rather than ``js:autofunction``.

``js:autofunction`` has one option, ``:short-name:``, which comes in handy for chained APIs whose implementation details you want to keep out of sight. When you use it on a class method, the containing class won't be mentioned in the docs, the function will appear under its short name in indices, and cross references must use the short name as well (``:func:`someFunction```)::

    .. js:autofunction:: someClass#someFunction
       :short-name:

autoclass
---------

We provide a ``js:autoclass`` directive which documents a class with the concatenation of its class comment and its constructor comment. It shares all the features of ``js:autofunction`` and even takes the same ``:short-name:`` flag, which can come in handy for inner classes. The easiest way to use it is to invoke the ``:members:`` option, which automatically documents all your class's public methods and attributes::

    .. js:autoclass:: SomeEs6Class(constructor, args, if, you[, wish])
       :members:

You can add private members by saying... ::

    .. js:autoclass:: SomeEs6Class
       :members:
       :private-members:

Privacy is determined by JSDoc ``@private`` tags.

Exclude certain members by name with ``:exclude-members:``::

    .. js:autoclass:: SomeEs6Class
       :members:
       :exclude-members: Foo, bar, baz

Or explicitly list the members you want. We will respect your ordering. ::

    .. js:autoclass:: SomeEs6Class
       :members: Qux, qum

Finally, if you want full control, pull your class members in one at a time by embedding ``js:autofunction`` or ``js:autoattribute``::

    .. js:autoclass:: SomeEs6Class

       .. js:autofunction:: SomeEs6Class#someMethod

       Additional content can go here and appears below the in-code comments,
       allowing you to intersperse long prose passages and examples that you
       don't want in your code.

autoattribute
-------------

This is useful for documenting public properties::

    class Fnode {
        constructor(element) {
            /**
             * The raw DOM element this wrapper describes
             */
            this.element = element;
        }
    }

And then, in the docs... ::

    .. autoclass:: Fnode

       .. autoattribute:: Fnode#element

This is also the way to document ES6-style getters and setters, as it omits the trailing ``()`` of a function. The assumed practice is the usual JSDoc one: document only one of your getter/setter pair::

    class Bing {
        /** The bong of the bing */
        get bong() {
            return this._bong;
        }

        set bong(newBong) {
            this._bong = newBong * 2;
        }
    }

And then, in the docs... ::

   .. autoattribute:: Bing#bong

Example
=======

A good example using most of sphinx-js's functionality is the Fathom documentation. A particularly juicy page is https://mozilla.github.io/fathom/ruleset.html. Click the "View page source" link to see the raw directives.

Fathom also carries a Travis CI configuration and a deployment script for building docs with sphinx-js and publishing them to GitHub Pages. Feel free to borrow them. (ReadTheDocs, which is otherwise the canonical hosting platform for Sphinx docs, doesn't work because it won't run JSDoc for us, nor will it accept uploads of docs built externally.)

Caveats
=======

* We don't understand the inline JSDoc constructs like ``{@link foo}``; you have to use Sphinx-style equivalents for now, like ``:js:func:`foo``` (or simply ``:func:`foo``` if you have set ``primary_domain = 'js'`` in conf.py.
* So far, we understand and convert only the JSDoc block tags ``@param``, ``@returns``, ``@throws``, and their synonyms. Other ones will go *poof* into the ether.

Tests
=====

Run ``python setup.py test``. Run ``tox`` to test across Python versions.

Version History
===============

Unreleased
  * Fix KeyError when trying to gather the constructor params of a plain old
    object labeled as a ``@class``.

1.5.2
  * Fix crasher while warning that a specified longname isn't found.

1.5.1
  * Sort ``:members:`` alphabetically when an order is not explicitly specified.

1.5
  * Add ``:members:`` option to ``autoclass``.
  * Add ``:private-members:`` and ``:exclude-members:`` options to go with it.
  * Significantly refactor to allow directive classes to talk to each other.

1.4
  * Add ``jsdoc_config_path`` option.

1.3.1
  * Tolerate @args and other info field lines that are wrapped in the source code.
  * Cite the file and line of the source comment in Sphinx-emitted warnings and errors.

1.3
  * Add ``autoattribute`` directive.

1.2
  * Always do full rebuilds; don't leave pages stale when JS code has changed but the RSTs have not.
  * Make Python-3-compatible.
  * Add basic ``autoclass`` directive.

1.1
  * Add ``:short-name:`` option.

1.0
  * Initial release, with just ``js:autofunction``
