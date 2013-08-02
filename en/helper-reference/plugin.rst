Plugin
======

In TreeFrog, the plug-in refers to dynamic libraries (shared library, DLL) that are added to extend the functionality.  Since TreeFrog has the Qt plug-in system, you can make a new plug-in if the standard function is insufficient.  At this point, the categories of possible plug-ins to be made are as follows.

  + Logger plug-in　-   log output
  + The session store plug-in　-  Saving and reading sessions

How to Make the Plug-in
------------------------

How to make a plug-in is exactly the same as how to create a plug-in for Qt. To demonstrate, let's create a logger plug-in. First, we'll make the working directory in the *plugin* directory

.. code-block:: bat
  
  > cd plugin 
  > mkdir sample

- You can choose anywhere for the working directory, but when thinking about the path solution, this is a good area. 
    
We’ll create a plug-in wrapper class (TLoggerPlugin in this example). We do this by inheriting the class as an interface, and then we override some of the virtual functions.

.. code-block:: c++
  
  class SamplePlugin : public TLoggerPlugin
  {
      QStringList keys() const;
      TLogger *create(const QString &key);
  };

In the keys() method, the string that can be the key for supporting the plug-in, is returned  as a list. In the create() method, the instance of the logger that corresponds to the key is created, and implemented in order to return as a pointer.

In order to register this plug-in, after including QtPlugin in the source file, put a macro such as the following at the end. The first argument can be any string such as the name of the plug-in. The second argument is a plug-in class name.

.. code-block:: c++
  
  #include <QtPlugin>
  :
  :
  Q_EXPORT_PLUGIN(samplePlugin, SamplePlugin)

Next, create a function to extend the plug-in (class). We’ll  make a logger that outputs a log here.  As above, we do this by inheriting the TLogger class as an interface from logger, and then override some virtual function.

.. code-block:: c++
  
  class SampleLogger : public TLogger
  {
  public:
      QString key() const { return "Sample"; }
      bool isMultiProcessSafe() const;
      bool open();
      void close();
      bool isOpen() const;
      void log(const TLog &log);
       :
       :
  };

The next is Project file (.pro).  Do not forget to add a "plugin" to CONFIG parameter.

.. code-block:: ini
  
  TARGET = sampleplugin
  TEMPLATE = lib
  CONFIG  += plugin
  DESTDIR = ..
  include(../appbase.pri)
  HEADERS = sampleplugin.h \
            samplelogger.h
  SOURCES = sampleplugin.cpp \
            samplelogger.cpp



- It’s important to include the appbase.pri file by using the include function.
    
After this, when you build you can make plug-ins that are dynamically loadable. Save the plug-in to the plugin directory every time without fault, because the application server (AP server) loads the plug-ins from this directory.
Please see the Qt documentation for details of the plug-in system.

Logger plug-in
--------------

FileLogger is a basic logger that outputs the log to a file. However, it may be insufficient, depending on the requirements. For example, if a log to save as a database, or  log file that you want to keep by rotation, you may want to extend the functionality using the mechanism of the plug-in.

As described above, after creating logger plug-in, place the plug-in in the directory. Furthermore, write the configuration information to the *logger.ini* file in order to be loaded into the application server. Arrange the key of the logger in loggers parameter separated by spaces. The following is an example.

.. code-block:: ini
  
  Loggers=FileLogger Sample

In this way, the plug-in will be loaded when you start the application server.

Once again, plug-in interface for the logger is a class as the next.

  + Plug-in interface: TLoggerPlugin
  + Logger interface: TLogger

**About the Logger Methods**
  In order to implement the logger, you can override the following methods in the class TLogger;

  + key()  ： returns the name of the logger.
  + open()  ： open of the log called by the plug-in immediately after loading.
  + close()  ： close the log.
  + log()  ：method to output the log. This method is called from multiple threads, make this as  thread-safe.
  + isMultiProcessSafe()  ： The thing that indicates whether it is safe for you to output a log in a multi-process. When it is safe, it returns true. I not, it returns as false.
  
About MultiProcessSafe method, when it returns false (meaning it is not safe), and the application server is running in the multiprocess mode as well, at before and after its log output, it calls open()/close() method each time (overhead is increasing).  By the way, this system has lock/unlock around this by semaphore, so that there is no conflict.  And when you return true, system will only call open() method first.

Session Store Plug-in
---------------------

The session store that is standard on TreeFrog is as follows.

  + Cookies Session : Save to cookies
  + DB session　：　Save to DB. You need to make the table only for this purpose.
  + File session : Save to file
  
If these are insufficient, you can create a plug-in that inherits the interface class.

  Plug-in interface: TSessionStorePlugin
  Session Store interface: TSessionStore

In the same manner as described above, by overriding the virtual function by inheriting these classes, you can create a plug-in.  Then in order to load this plug- in, you can set only one key at *Session.StoreType* parameter in the *application.ini* file (you can choose only one).  The default is set as cookie.


