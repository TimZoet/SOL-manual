Export
======

Source and module objects can be exported individually, or in bulk.

Individual export can be done through methods of the :code:`ShaderCache` or of the individual :code:`ShaderCacheSource` 
and :code:`ShaderCacheModule`:

.. code-block:: cpp

    // Export through source object method using original path
    auto source = cache->getSource("vertex_0");
    source->exportToFile(source->getPath(), true);
    
    // Export through cache method.
    cache->exportSource("fragment_0", "path/to/file.glsl", true);

All sources and modules in a cache can be exported at once using the :code:`exportCache` method:

.. code-block:: cpp

    cache->exportCache("path/to/directory", true);

This will generate a file for each object inside of the specified directory. Each file is located in either the 
:code:`sources` or :code:`modules` subfolder. Also taken into account are groups in the identifier, e.g. a shader source
object :code:`blur/gaussian` can be found in the :code:`path/to/directory/sources/blur` directory.