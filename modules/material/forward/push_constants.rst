Push Constants
==============

Defining push constants is simple. If you have the following shaders:

.. code-block:: c

    // Vertex shader:

    layout(push_constant) uniform ModelMatrix
    {
        layout(offset = 0) mat4 model;
    } modelMatrix;

    ...

    // Fragment shader:

    layout(push_constant) uniform ObjectColor
    {
        layout(offset = 16) vec4 color;
    } objectColor;

    ...

All you need to do is add matching definitions to the material layout. No additional configuration required (or even
possible):

.. code-block:: cpp

    MyMaterial::MyMaterial(...)
    {
        auto& pc0  = layout.addPushConstant();
        pc0.offset = 0;
        pc0.size   = sizeof(float) * 16;
        pc0.stages = VK_SHADER_STAGE_VERTEX_BIT;

        auto& pc1  = layout.addPushConstant();
        pc1.offset = sizeof(float) * 16;
        pc1.size   = sizeof(float) * 4;
        pc1.stages = VK_SHADER_STAGE_FRAGMENT_BIT;

        layout.finalize();
    }
