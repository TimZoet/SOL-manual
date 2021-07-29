sol.shadercache
===============

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    shadercache/import
    shadercache/export
    shadercache/compilation
    shadercache/retrieval
    shadercache/application

The :code:`sol.shadercache` module contains functionality to import, store, retrieve and compile shader source and 
binary files. These files are managed in a single Sqlite database file, a.k.a. the cache.

ShaderCache
-----------

The :code:`ShaderCache` class is the interface to a shader cache file. Opening and creating shader cache files is done 
through static methods of the class:

.. code-block:: cpp

    auto [cache, created] = ShaderCache::openOrCreate("path/to/cache.db");

A note on runtime, in-memory caching of objects. This is disabled by default. When you e.g. retrieve a source object but
let all references to it go out of scope, the object is deallocated entirely. A new retrieval will have to perform disk 
access again. The :code:`setCaching` method can be used to change this behaviour. You should of course be wary of memory
consumption not getting out of hand:

.. code-block:: cpp

    cache->setCaching(true);

ShaderCacheSource and ShaderCacheModule
---------------------------------------

Two kinds of objects are stored in the cache: source objects (:code:`ShaderCacheSource`) and module objects 
(:code:`ShaderCacheModule`). Source objects hold uncompiled GLSL source code. Module objects hold the compiled binary 
code. See :ref:`shader-cache-import` for details on how to create objects.

All objects need to be given a unique identifier. The identifier is used for :ref:`shader-cache-retrieval` of objects. 
An identifier should only consist of alphanumeric characters and underscores. Additionally, you can use single forward 
slashes to effectively place shaders in subfolders. For completeness, this is the regular expression that is used to 
validate an identifier: :code:`[a-zA-Z0-9_]+(\/[a-zA-Z0-9_]+)*`.

