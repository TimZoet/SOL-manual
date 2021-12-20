Shader Module
=============

The :code:`sol::VulkanShaderModule` class manages the lifetime of a :code:`VkShaderModule`. It can be created from 
shader source code that is then compiled, or directly from precompiled binary code:

.. code-block:: cpp
    
    // Compile vertex shader. Have compiler infer type from source code.
    sol::VulkanShaderModule::Settings settings;
    settings.device   = device;
    settings.type     = sol::VulkanShaderModule::ShaderType::Automatic;
    settings.filename = "vertex.glsl"; // Filename is optional. Can be useful when inspecting compilation errors.
    settings.source   = "..."; // Load source from somewhere.
    auto vertShader   = sol::VulkanShaderModule::create(settings);

    // Create fragment shader from precompiled binary code.
    settings.device   = device;
    settings.type     = sol::VulkanShaderModule::ShaderType::Fragment;
    settings.binary   = ...; // Load binary from somewhere.
    auto fragShader   = sol::VulkanShaderModule::create(settings);

Compiler
--------

By default, a new :code:`shaderc::Compiler` is constructed every time you create a new module from source. For
performance reasons it is best to pass in an already initialized compiler when creating more than one module:

.. code-block:: cpp
    
    shaderc::Compiler compiler;

    sol::VulkanShaderModule::Settings settings;
    settings.device   = device;
    settings.type     = sol::VulkanShaderModule::ShaderType::Automatic;
    settings.compiler = &compiler;

    for (const auto& source : sourcesToCompile)
    {
        settings.source = source;
        auto shader = sol::VulkanShaderModule::create(settings);
    }
