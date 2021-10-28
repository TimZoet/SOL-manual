Forward Material Instance
=========================

Once you have a fully implemented forward material class, you must create a class that derives from 
:code:`sol::ForwardMaterialInstance`. This class must implement several methods.

Of course you must specify which material a material instance is an, erm..., instance of. This is done by passing a 
reference to some forward material object to the base class's constructor.

Also needed is the index of the descriptor set the material instance represents. If there is only one descriptor set in
the material layout, you'll of course only need a single class that just returns 0 from :code:`getSetIndex`. For 
multiple descriptor sets it is allowed to create either separate classes, or deal with that inside of a single class. 
Just as with the material class, material instance classes are very flexible in how they can be implemented. At runtime,
whenever you want to instantiate a forward material, any mesh or other drawable cannot be rendered without a material 
instance for each descriptor set.

For uniform buffers, samplers and push constants a number of methods must be implemented as well. See the sections 
below.

.. code-block:: cpp

    class MyMaterialInstance : public sol::ForwardMaterialInstance
    {
    public:
        // Constructor must pass material to base class.
        MyMaterialInstance(MyMaterial& myMaterial);

        // Must return the descriptor set index this material instance represents.
        [[nodiscard]] uint32_t getSetIndex() const override;

        /*
         * Uniform buffer data. See following section.
         */

        [[nodiscard]] const void* getUniformBufferData(size_t binding) const override;

        [[nodiscard]] bool isUniformBufferStale(size_t binding) const override;
    
    private:
        // Member variables containing uniform data, textures, etc.
        ...
    };

Uniform Buffers
---------------

Regardless of whether or not the material layout actually contains uniform buffer bindings you must implement several
methods. This is mainly done to prevent you from forgetting to do so when it is needed. If not needed, just return 
:code:`nullptr`, :code:`false` or anything else. The methods shouldn't get called.

The :code:`getUniformBufferData` method must of course return a pointer to the actual data that is to be written to the 
uniform buffers. The pointer should refer to the data for the requested :code:`binding`. The size of the data should be
equal to however many bytes were defined in the material layout.

If the material layout contains uniform buffer bindings whose update detection method is configured as 
:code:`sol::ForwardMaterialLayout::UpdateDetection::Manual`, the :code:`isUniformBufferStale` method should return 
whether or not the data changed for the requested binding. If this method returns :code:`true`, 
:code:`getUniformBufferData` will be called. For bindings with a different update detection method this function is 
skipped and the data is retrieved immediately. For more information on this feature, refer to :doc:`uniform_buffers`.

Samplers
--------

.. note::
    Not yet implemented.

Push Constants
--------------

Push constants are not set through material instances, but through separate nodes. See 
:doc:`../../scenegraph/nodes/forward_push_constant_node`.
