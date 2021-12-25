Forward Renderer
================

Configuration
-------------

...

Clearing
^^^^^^^^

There are various settings for clearing the framebuffer color, depth and stencil values when beginning the render pass.
These can be configured using the various :code:`setClear...` methods. For the clear color, the format of the
framebuffer is relevant. Refer to the :code:`Vulkan` documentation for details:

.. code-block:: cpp

    auto renderer = std::make_unique<sol::ForwardRenderer>();
    ...
    renderer->setClearColorFormat(sol::ForwardRenderer::ClearColorFormat::Uint);
    renderer->setClearColorUint({255, 0, 0, 255});
    renderer->setClearDepth(1.0f);
    renderer->setClearStencil(0);
