Uniform Package Metadata Representation
=======================================

Motivation & Goals
------------------

Make the life of a package user simpler by having all the package metadata for
common and known use cases specified in a uniform and intuitive manner
irrespective of the tool it is used by.  In addition to having a uniform syntax
and semantics the metadata representation should be:

* concise i.e.remove repetition
* organized in a modular way
* standard (having well defined semantics)
* extensible (allow tool specific extensions where necessary)

The desire is to encourage:

* all existing tools to adopt the same format
* any new tools should reuse and extend the existing format rather than using their own

Existing tools and formats
--------------------------

Some examples of tools that work on some sort of package metadata using
different config specification formats:

* cabal (package.cabal)
* stack (stack.yaml)
* iridium (iridium.yaml)
* cabal-debian (Debianize.hs)

We can imagine more use cases e.g. test matrices.

Uniform Metadata Representation
-------------------------------

Assuming we a use a fictitious format called ``info``, we can imagine
representing various types of package metadata described in the previous section
systematically in a uniform format, using individual files inside a ``pkg``
directory. Something like this:

+--------------+------------------------------------------------------------------------+
| pkg.info     | a. General package information (name, version, author, location etc.)  |
|              | b. Build control information (flags, sources, deps version bounds)     |
|              |                                                                        |
|              | (cf. .cabal)                                                           |
+--------------+------------------------------------------------------------------------+
| dist.info    | Distribution policies (pvp checks, sanity checks, license checks etc)  |
|              | (cf. iridium.yaml)                                                     |
+--------------+------------------------------------------------------------------------+
| test.info    | Package maintainer's reproducible build matrix for testing the package |
+--------------+------------------------------------------------------------------------+
| project.info | Reproducible build spec for a tree of packages (cf. stack.yaml)        |
+--------------+------------------------------------------------------------------------+
| debian.info  | debian packaging (cf. Debianize.hs)                                    |
+--------------+------------------------------------------------------------------------+

There should be an independent library to parse '.info'. This library can be used
by all the tools to read and write the config.

The .info format could be one of ``.cabal`` or ``.yaml`` or ``.hs``. Currently
all three are used by different tools.
