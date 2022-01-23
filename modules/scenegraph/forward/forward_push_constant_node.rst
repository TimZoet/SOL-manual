Forward Push Constant Node
==========================

The :code:`sol::ForwardPushConstantNode` class defines a very simple interface that can be implemented to add push
constant data to a scenegraph. All drawable nodes will be drawn with the data from the nearest push constant nodes above
them.

Reusing some shader code from the :code:`sol.material` section:

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

Let's assume there is a matching :code:`sol::ForwardMaterial` class whose layout contains the two push constants in the
above shader code. We can then decide how to create matching :code:`sol::ForwardPushConstantNode` classes to hold the 
actual data. It is possible to put both values into a single class, or split them up. Let's go with the second option:

.. code-block:: cpp

    class ModelMatrixNode : public sol::ForwardPushConstantNode
    {
    public:
        ModelMatrixNode(sol::ForwardMaterial& material) : ForwardPushConstantNode(material)
        {
            // Enable first push constant.
            enablePushConstant(0);
        }

        [[nodiscard]] const void* getData(size_t index) const override
        {
            // Index should never not be 0 because we only enabled 1 push constant.
            assert(index == 0);

            return matrix.data();
        }

        // Probably some getters and setters for matrix.
        ...
    
    private:
        // Actual data.
        std::array<float, 16> matrix:
    };

    class ObjectColorNode : public sol::ForwardPushConstantNode
    {
    public:
        ObjectColorNode(sol::ForwardMaterial& material) : ForwardPushConstantNode(material)
        {
            // Enable second push constant.
            enablePushConstant(1);
        }

        [[nodiscard]] const void* getData(size_t index) const override
        {
            // Index should never not be 0 because we only enabled 1 push constant.
            assert(index == 0);

            return color.data();
        }

        // Probably some getters and setters for color.
        ...
    
    private:
        // Actual data.
        std::array<float, 4> color:
    };

The first push constant will be handled by the :code:`ModelMatrixNode` class. Nodes of this type will hold a 16 element
float array representing the matrix. The second push constant will be handled by the :code:`ObjectColorNode` class.
Nodes of this type will hold a 4 element float array representing the color. They both enable their respective push
constants in their constructors by calling :code:`enablePushConstant`.

Both classes should also implement the :code:`getData` method. This method will be called during traversal to retrieve
the data of each enabled push constant. If you enabled more than 1 push constant, the index parameter should be used to
determine which push constant is being requested. Note that this index parameter is relative to the enabled push
constants, not the entire list of push constants in the material layout.

Both of these nodes can now be added to the scenegraph. Any drawables below them that are drawn with the same material
(or a compatible material) should use the push constant values:

.. code-block:: cpp

    // Assuming some previously created objects.
    sol::MeshInstance&               mesh = ...;
    sol::ForwardMaterial&             mtl = ...;
    sol::ForwardMaterialInstance& mtlInst = ...;
    sol::Scenegraph            scenegraph = ...;
    sol::Node&                       root = scenegraph.getRootNode();

    // Add material, then matrix, then color, then drawable.
    auto&    mtlNode = root.addChild(std::make_unique<sol::ForwardMaterialNode>(mtlInst));
    auto& matrixNode = mtlNode.addChild(std::make_unique<ModelMatrixNode>(mtl));
    auto&  colorNode = matrixNode.addChild(std::make_unique<ObjectColorNode>(mtl));
    auto&   meshNode = colorNode.addChild(std::make_unique<sol::MeshNode>(mesh));
