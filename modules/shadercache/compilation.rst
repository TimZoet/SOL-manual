.. _shader-cache-compile:

Compilation
===========

While you can import shader modules directly, it's much more likely that you want to create them by compiling shader 
source code. Doing this is pretty straightforward. Just call the :code:`compile` method on a source object:

.. code-block:: cpp
    
    auto source = cache->getSource("vertex_0");
    auto module = source->compile("vertex_0", false);

The module object that is created is linked to the source object. It also stores the checksum of the source code at the
time of compilation. If the source code is then changed, you can compare the checksums to see if the module needs to be
recompiled.

The identifier of the module object does not need to be the same as that of the source object. Furthermore, you can 
compile the same source multiple times using different identifiers. Each individual module is linked to the source.

After compilation, you can retrieve source objects and module objects from each other:

.. code-block:: cpp
    
    // Returns source object (or nullptr if not linked).
    auto source = module->getSource();
    // Returns 0 or more modules.
    auto modules = source->getModules();

To create an actual :code:`VulkanShaderModule`, you can either construct your own using that class's methods, or just 
use the utility method the module object provides:

.. code-block:: cpp
    
    // Get a device from somewhere.
    VulkanDevicePtr device = ...;
    // Get module from cache...
    auto cacheModule = cache->getModule("fragment_0");
    // ...and create VulkanShaderModule from it.
    auto shaderModule = cacheModule->createVulkanShaderModule(*device);
