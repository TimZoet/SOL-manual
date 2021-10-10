Custom Command
==============

The :code:`sol::CustomCommand` can be used to quickly create a command that needs to execute some custom code without
implementing a separate class. It takes only one parameter, namely a :code:`std::function` object that is called when
the command is executed:

.. code-block:: cpp

    auto customCmd = std::make_unique<sol::CustomCommand>([]{
        ...
    });
