Image Manipulation
==================

There are various libraries that can make image processing.  If you need to do complicated image processing you may need to use `OpenCV <http://opencv.org/>`_ , but here I would like to explain the Qt library. It’s easy to use. Since Qt is the base library of the TreeFrog Framework, it is ready to provide a GUI tool kit including many useful functions for image processing.

At first, for image processing, it is necessary to activate the QtGUI module of the TreeFrog framework.
Let’s recompile the framework.

In the case of Linux / Mac OS X:

.. code-block:: bash
  
  $ ./configure --enable-gui-mod
  $ cd src
  $ make 
  $ sudo make install
  $ cd ../tools
  $ make
  $ sudo make install

In the case of  Windows:

.. code-block:: bat
  
  > configure --enable-debug --enable-gui-mod
  > cd src
  > mingw32-make install
  > cd ..\tools
  > mingw32-make instal
  > cd ..
  > configure --enable-gui-mod
  > cd src
  > mingw32-make install
  > cd ..\tools
  > mingw32-make install

Next, the setting is also required for the Web application side. Edit the project file (.pro), add "gui" in the QT variable.

.. code-block:: ini
  
  :
  QT += network  sql  gui
  :

In this setting, the app is built as a GUI application. For Linux, particularly, X Window System is required to implement the environment.
If you cannot meet this requirement, it is recommended that you use OpenCV as your image processing library.

Resize the image
----------------

The following code is an example of how to save a JPEG image by converting to the QVGA size, while keeping the aspect ratio

.. code-block:: c++
  
  QImage img = QImage("src.jpg").scaled(320, 240, Qt::KeepAspectRatio);
  img.save("qvga.jpg");

- In reality, use the absolute path as a the file path.

Using this QImage class, you can convert while ignoring aspect ratio.  Also you can convert to a different image format.  Please see `Qt Document <http://qt-project.org/doc/qt-4.8/>`_ for detail.

Rotation of the Image
---------------------

The following code is an example of rotating an image clockwise through 90 degrees.

.. code-block:: c++
  
  QImage img("src.jpg");
  QImage rotatedImg = img.transformed(QMatrix().rotate(90.0));
  rotatedImg.save("rotated.jpg");

It’s very simple.

Image Synthesis
---------------

Let’s superimpose two images. You can align the small image to the coordinates of the upper right corner in the big picture.

.. code-block:: c++
  
  QImage background("back.jpg");
  QPainter painter(&background);
  painter.drawImage(0, 0, QImage("small.jpg"));
  background.save("composition.jpg");

You can then prepare a painter to the original image, and draw a different picture there.

