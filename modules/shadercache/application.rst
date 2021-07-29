Command Line Application
========================

There is a command line application that wraps the main functions of the shader cache module. It allows you to import 
and export files, compile shaders and inspect a cache.

Each command requires specifying a path to a :code:`--cache` and an :code:`--action`. Each action requires specifying 
one or more parameters. See the :code:`--help` for a detailed description of each action and their parameters.

Below follow some examples of the most important functions.

Import
------

Importing a vertex shader source file:

.. code-block:: sh

    > shader_cache --cache=path/to/cache.db --import --source=vtx_shader.glsl --id=vertex/name1 --shadertype=vertex

Overwriting that same object:

.. code-block:: sh

    > shader_cache --cache=path/to/cache.db --import --overwrite --source=vtx_shader2.glsl --id=vertex/name1 --shadertype=vertex

Importing a fragment shader source file without explicitly specifying its type:

.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --import --source=frag_shader.glsl --id=fragment/name1

Importing an already compiled shader:

.. code-block:: sh
    
    shader_cache --cache=path/to/cache.db --import --module=frag_shader.bin --id=fragment/name2 --shadertype=fragment

Compile
-------

Compiling a shader requires specification of a shader source and optionally a module identifier:
    
.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --compile --source=vertex/name1 --id=vertex/name1
    
If no module id is specified, the id of the shader source is used (note that if that id is already in use, an error is 
generated unless :code:`--overwrite` is specified).

Recompiling a shader module is also allowed when it has a reference to the shader source:
    
.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --compile --overwrite --module=fragment/name1

It is also possible to recompile all shader modules that have a link to a source that was modified:
    
.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --compile --stale

Alternatively, you can compile all shader sources for which no module referencing them exists:

.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --compile --uncompiled

Inspect
-------

There are various ways in which you can inspect a shader cache.

To list all objects in the cache:

.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --list

To list all shader modules that are outdated and need to be recompiled:

.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --list --stale

To list all shader sources that do not yet have a corresponding module:

.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --list --uncompiled

To list all shader modules that do not have a reference to a source object:

.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --list --unlinked

To list all shader sources whose identifier match a certain pattern:

.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --list --source=vertex/foo/*

To list all shader modules whose identifier match a certain pattern:

.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --list --module=vertex/foo/*

To view the details of a specific shader source:

.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --view --source=vertex/name1

To view the details of a specific shader module:

.. code-block:: sh
    
    > shader_cache --cache=path/to/cache.db --view --module=vertex/name1
