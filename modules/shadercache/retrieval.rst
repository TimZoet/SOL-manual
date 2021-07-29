.. _shader-cache-retrieval:

Retrieval
=========

Naturally, objects can be retrieved from the cache directly using their identifier:

.. code-block:: cpp

    auto source = cache->getSource("vertex_0");
    auto module = cache->getModule("fragment_0");

It is also possible to retrieve objects using an identifier pattern. These patterns consist of all the characters of an 
identifier, plus wildcards (:code:`*`). The list that is returned will contain all matching objects (if any):

.. code-block:: cpp

    // Find all sources whose identifier starts with "vertex".
    auto sources = cache->findSources("vertex*");
    // Find all sources grouped under "blur".
    sources = cache->findSources("blur/*");
    // Find all sources whose identifier contains "color".
    sources = cache->findSources("*color*");
    // Or just get all sources.
    sources = cache->findSources("*");

There are a number of utility methods to retrieve objects based on their state. For example, there is the 
:code:`getUncompiledSources` method. As the name implies, this will return all source objects for which no module 
exists:

.. code-block:: cpp

    // Compile all uncompiled sources.
    for (auto source : cache->getUncompiledSources())
    {
        source->compile();
    }
