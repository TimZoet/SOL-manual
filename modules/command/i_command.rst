ICommand
========

The :code:`sol::ICommand` class defined the interface which all commands must implement in order to be compatible with
the :code:`sol::CommandQueue`. It currently has only 2 methods.

:code:`finalize` will be called whenever the same method on the command queue is called. In this method, you should
verify that the command is correctly configured and throw an exception if it is not.

:code:`operator()` should contain the code that is to be executed when this command is invoked.
