
.. _making_original_helper:

Making Original Helper
======================

When you create a helper, it can be accessible from the model/controller/view. Please try to consider making a helper whenever similar code appears several times.

How to make the helper
----------------------

You can implement the class as you like in the *helpers directory*, remember to put the charm of T_HELPER_EXPORT.  Other than that you can do anything.

.. code-block:: c++
  
  #include <TGlobal>
  class T_HELPER_EXPORT SampleHelper
  {
      // Write as you like.
  };

Add the header and the source file to project files, *helpers.pro*, then run make.

How to Use the Helper
---------------------

Use the same as the normal class by including header file.  There is no particular difference compared to the way the normal class is used.
