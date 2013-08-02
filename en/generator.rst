=========
Generator
=========
Here, I'll take a look at the generator command, tspawn.

Generate Skeleton
-----------------

First, we must create a skeleton of the application before we can do anything else. We'll use the name blogapp again for our creation. Enter the following command from the command line. (In Windows, run from the TreeFrog Command Prompt).

.. code-block:: bash
  
  $ tspawn new blogapp

When you run, the directory tree including the application root directory at the top, configuration file (ini), and project files (pro) are generated. The directory is just one of the names you have seen before. 
The following items will be generated as a directory.

  + controllers
  + models
  + views
  + heplers
  + config   – configuration files
  + db      - database file storage (SQLite)
  + lib    
  + log      – log files
  + plugin 
  + public   – static HTML files, images and JavaScript files
  + script  
  + test    
  + tmp      – temparary directory, such as a file upload


Generate Scaffold
-----------------

The scaffold is a basic implementation that allows CRUD operations to take place. Included in the scaffold are; controller, model, source files for views, and project files (pro).  The scaffold therefore forms a good basis on which to start a full-scale development.

In order to make a scaffold in the generator command, you need to define a table in the database in advance, and to describe the database information in the configuration file.  Now, let's define a table.
See this example:

.. code-block:: mysql
  
  CREATE TABLE blog (id INTEGER PRIMARY KEY, title VARCHAR(20), body VARCHAR(200));

If you want to use SQLite for the database, you should make the db database file in the application root directory. You can set the database information in the configuration file (database.ini). The generator command refers to the information that is set in the dev section.

.. code-block:: ini
  
  [dev]
  driverType=QMYSQL
  databaseName=blogdb
  hostName=
  port=
  userName=root
  password=root
  connectOptions=

**Settings List**

+----------------+--------------------+------------------------------------------------------+
| Item           | Meaning            | Remarks                                              |
+================+====================+======================================================+
| driverType     | Driver name        | Choices are as follows:                              |
|                |                    | QDB  2  : IBM DB2                                    |
|                |                    | QIBASE : Borland InterBase Driver                    |
|                |                    | QMYSQL : MySQL Driver                                |
|                |                    | QOCI  : Oracle Call Interface Driver                 |
|                |                    | QODBC : ODBC Driver                                  |
|                |                    | QPSQL : PostgreSQL Driver                            |
|                |                    | QSQLITE : SQLite version 3 or above                  |
+----------------+--------------------+------------------------------------------------------+
| databaseName   | Database name      | In the case of SQLite a file path must be specified. |
|                |                    | Example:  db/blogdb                                  |
+----------------+--------------------+------------------------------------------------------+
| hostName       | Host name          | *localhost* in the case of blank                     |
+----------------+--------------------+------------------------------------------------------+
| port           | Port number        | The default port if blank                            |
+----------------+--------------------+------------------------------------------------------+
| userName       | User name          |                                                      |
+----------------+--------------------+------------------------------------------------------+
| password       | Password           |                                                      | 
+----------------+--------------------+------------------------------------------------------+ 
| connectOptions | Connection options | For more information see Qt documents :              |
|                |                    | QSqlDatabase::setConnectOptions()                    |
+----------------+--------------------+------------------------------------------------------+

If the database driver is not included in the Qt SDK, you will not be able to access the database. If you have not yet built, you should incorporate the driver by reference to the FAQ. Alternatively, you can download the database driver from the `download page <http://www.treefrogframework.org/download>`_ , and then install it.

After the above, when you run the generator command (after the above steps), the scaffolding will be generated. Every command should be run from the application root directory.

.. code-block:: bash
  
  $ cd blogapp
  $ tspawn scaffold blog
    driverType: QMYSQL
    databaseName: blogdb
    hostName:
    Database open successfully
      created  controllers/blogcontroller.h
      created  controllers/blogcontroller.cpp
      :

**In Brief: Define the schema in the database, and make the scaffolding with the generator command.**

**Relationship of model-name/controller-name and table name**
The generator will generate class names determined on the basis of the table name. The rules are as the following.

::
  
  Table name        Model name   Controller name      Sql object name
  blog_entry   →    BlogEntry    BlogEntryController  BlogEntryObject

Notice that when the underscore is removed, the next character is capitalized. You may completely ignore any distinction between singular and plural word forms.

Generator Sub-commands
----------------------

Here are the usage rules for the *tspawn* command.

.. code-block:: bash
  
  $ tspawn -h
    usage: tspawn <subcommand> [args]
    
    Available subcommands:
      new (n)  <application-name>
      scaffold (s)  <model-name>
      controller (c)  <controller-name>
      model (m)  <table-name>
      sqlobject (o)  <table-name>

If you specify “controller”, “model”, or “sqlobject“ as a sub-command, you can generate ONLY “controller”, ONLY “model”, or ONLY “SqlObject”.

**Column**

TreeFrog has no migration feature or other mechanism for making changes to and differential management of the DB schema. I think this is unimportant for the following reasons:

1. If I had made a Migration function, users would face the extra learning cost.
2. Those that are knowledgeable about SQL can enjoy the full functionality of dB operations.
3. In TreeFrog, it is possible to regenerate only the ORM object classes when changing table. 
   → (Unfortunately there might be some possibilities for affecting something to the model class…)
4. I consider that there is little merit to framework-side differential management of SQL commands.
 
Agreed?

Naming Conventions
------------------

TreeFrog has a class naming and file naming convention.  With the generator, class or file names are generated under the following terms and conditions.

**Convention for Naming of Controllers**

The class name of the controller is "table name + Controller". Always begin with an upper-case letter, do not use the underscore ('_') to separate words but capitalize the first letter after where the separator would be.
The following class names are good examples:

  + BlogController
  + EntryCommentController

These files are stored in the controller’s directory. File names in that case will be something that you want to be in all lowercase; the class name plus the relevant extension (.cpp or .h).
  
**Conventions for Naming Models**

In the same manner as with the controller, model names should always begin with a capital letter, erase the underscore ('_') to separate words but capitalize the first letter after where the separator would be. For example, class names such as the following.

  + Blog
  + EntryComment

These files are stored in the models directory. As well as the controller, the file name will be something that you want all in lowercase. The model name is used, plus the file extension (.cpp or .h).
It does not convert the singular-plural form of words, as Rails does.


**View Naming Conventions**

Template files are generated with the file name "action name + extension" all in lower case, in the 'views/controller name" directory. The extension used depends on the template system.
Also, when you build the view, and then output the source file in views/_src directory they are converted to C++ code templates. When these are compiled, a shared library view is created.


CRUD
----

CRUD covers the four major functions found in a Web application. The name comes from taking the initial letters of "Create (generate)," "Read (Read)", "Update (update)", and "Delete (Delete)".
When you create a scaffolding, the generator command generates the naming code in the next.

**CRUD Correspondence Table**

+---+--------+-------------------+----------+--------+
|   | Action | Model             | ORM      | SQL    |
+===+========+===================+==========+========+
| C | create | create() [static] | create() | INSERT |
|   |        | create()          |          |        |
+---+--------+-------------------+----------+--------+
| R | index  | get() [static]    | find()   | SELECT |
|   | show   | getAll() [static] |          |        |
+---+--------+-------------------+----------+--------+
| U | save   | save()            | update() | UPDATE |
|   |        | update()          |          |        |
+---+--------+-------------------+----------+--------+
| D | remove | remove()          | remove() | DELETE |
+---+--------+-------------------+----------+--------+

About the T_CONTROLLER_EXPORT Macro
-----------------------------------

The controller class that you created in the generator, will have added a macro called T_CONTROLLER_EXPORT.

In Windows, the controller is a single DLL file, but in order to be available from outside the classes and functions of these, we need to define it with a keyword __declspec called (dllexport). The T_CONTROLLER_EXPORT macro is then replaced with this keyword. 
However, nothing is defined in the T_CONTROLLER_EXPORT in Linux and Mac OS X installations, because a keyword is unnecessary.

.. code-block:: c++
  
  #define T_CONTROLLER_EXPORT

In this way, the same source code is used, and you are thus able to support multiple platforms.

