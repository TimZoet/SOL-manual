.. _shader-cache-import:

Import
======

Both the :code:`ShaderCacheSource` and :code:`ShaderCacheModule` objects can be imported from file, or created by 
passing in the source or binary code yourself.

Shader Sources
--------------

An instance of the :code:`ShaderCacheSource` class corresponds to a single shader source. You can either import a shader
from a file or create one by directly by passing the source code. This is done using the :code:`importSource` and 
:code:`createSource` method, respectively:

.. code-block:: cpp

    auto source0 = cache->importSource("path/to/vert.glsl",
                                       "vertex_0",
                                       VulkanShaderModule::ShaderType::Vertex,
                                       ShaderCache::OverwriteAction::Always);
    auto source1 = cache->createSource("<glsl code>",
                                       "fragment_0",
                                       VulkanShaderModule::ShaderType::Fragment,
                                       ShaderCache::OverwriteAction::Always);

In both cases, the source code is stored in the database. When importing a file, the path is stored as well so that you 
can reimport the file later on.

Also stored is the MD5 checksum of the source code. This is used when overwriting existing source objects, and to detect
whether any shader modules are out of date and need to be recompiled because the source object they reference was 
updated.

After creating an object, some properties can still be changed. Note that these changes are not saved to disk until you
call the :code:`save` method.

Shader Modules
--------------

A :code:`ShaderCacheModule` holds the binary code of a compiled shader. These objects, too, can be imported or created.
However, in practice you will most likely be creating these by compiling source objects (see also 
:ref:`shader-cache-compile`):

.. code-block:: cpp

    // Import module.
    auto module0 = cache->importModule("path/to/shader.bin",
                                       "vertex_0",
                                       VulkanShaderModule::ShaderType::Vertex,
                                       ShaderCache::OverwriteAction::Always);
    // Create module by compiling.
    auto module1 = source1->compile(source1->getIdentifier(), true);

Regardless of how the object was created, it stores raw binary data (a :code:`std::vector<std::byte>`) which can be 
turned into a :code:`VulkanShaderModule`.
