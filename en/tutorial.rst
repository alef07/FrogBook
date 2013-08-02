
.. _tutorial:

========
Tutorial
========

Let's make a TreeFrog Application. In doing so we will be taking a tour around the whole process, so let’s start!
We'll try to make a simple blog system which can list, view, and add/edit/delete text.

Generate the Application Skeleton
---------------------------------

First we will need to make a skeleton (various settings files and a directory tree). We’ll use the name “blogapp.” Run the following from the command line. (In Windows, please run from the TreeFrog Command Prompt).

.. code-block:: bash

  $ tspawn new blogapp
    created  blogapp
    created  blogapp/controllers
    created  blogapp/models
    created  blogapp/models/queries
    created  blogapp/models/sqlobjects
    created  blogapp/views


Create a table
--------------

Next we need to make a table in the database. We’ll make the field title and content (body). Here, are examples in MySQL and SQLite’ 
 
Example in MySQL:
Set to UTF-8 character set. You can also specify this when generating the database (do ensure that it is being set correctly, see FAQ ). You can specify the configuration file for the database, as described below. Also, make the path through into MySQL using the command line tool.

.. code-block:: bash
  
  $ mysql -u root -p
  Enter password:

  mysql> CREATE DATABASE blogdb DEFAULT CHARACTER SET utf8;
  Query OK, 1 row affected (0.01 sec)

  mysql> USE blogdb;
  Database changed

  mysql> CREATE TABLE blog (id INTEGER AUTO_INCREMENT PRIMARY KEY, title VARCHAR(20), body VARCHAR(200), created_at TIMESTAMP DEFAULT 0, updated_at TIMESTAMP DEFAULT 0, lock_revision INTEGER) DEFAULT CHARSET=utf8;

  Query OK, 0 rows affected (0.02 sec)

  mysql> DESC blog;
  +---------------+--------------+------+-----+---------------------+------
  | Field         | Type         | Null | Key | Default             | Extra
  +---------------+--------------+------+-----+---------------------+------
  | id            | int(11)      | NO   | PRI | NULL                | auto_     
  | title         | varchar(20)  | YES  |     | NULL                |      
  | body          | varchar(200) | YES  |     | NULL                |      
  | created_at    | timestamp    | NO   |     | 0000-00-00 00:00:00 | 
  | updated_at    | timestamp    | NO   |     | 0000-00-00 00:00:00 |      
  | lock_revision | int(11)      | YES  |     | NULL                |      
  +---------------+--------------+------+-----+---------------------+------
  6 rows in set (0.01 sec)

  mysql> quit
  Bye

Example in SQLite:
We are going to put the database files in the db directory.

.. code-block:: bash

  $ cd blogapp
  $ sqlite db/blogdb
    SQLite version 3.6.12
    sqlite> CREATE TABLE blog (id INTEGER PRIMARY KEY, title VARCHAR(20), body VARCHAR(200), created_at TIMESTAMP, updated_at TIMESTAMP, lock_revision INTEGER);
    sqlite> .quit

A blog table is made with the fields:  id, title, body, created_at, updated_at, and lock_revision.

With the fields updated_at and created_at, TreeFrog will automatically insert the date and time of creation and of each update. The lock_revision field, which is intended for use with optimistic locking, needs to be created as an integer type.
 
Optimistic Locking
------------------

The optimistic locking is used to store data while verifying that information is not locked by being updated by another user. Since there is no actual write lock, you can expect processing to be a little faster. 
See the section on O/R mapping for more information.

Set the Database Information
----------------------------

Use config/database.ini to set information about the database.
Open the file in the editor, enter the appropriate values for your environment to each item in the [dev] section, and then click Save.
 
Example in MySQL:

.. code-block:: ini

  [dev]
  DriverType=QMYSQL
  DatabaseName=blogdb
  HostName=
  Port=
  UserName=root
  Password=root
  ConnectOptions=

Example in SQLite:

.. code-block:: ini

  [dev]
  DriverType=QSQLITE
  DatabaseName=db/blogdb
  HostName=
  Port=
  UserName=
  Password=
  ConnectOptions=

Once you have correctly set these details, it's time to display the table to access the DB.

.. code-block:: bash

  $ cd blogapp 
  $ tspawn --show-tables
    DriverType:   QSQLITE
    DatabaseName: db\blogdb 
    HostName:
    Database opened successfully
    -----
    Available tables:
    blog

If all is well, it will display like this.

If a required SQL driver is not included in the Qt SDK, the following error message will show.
"QSqlDatabase: QMYSQL driver not loaded"

If you receive this message, download the SQL database driver from the `download page <http://www.treefrogframework.org/download>`_ , and install it.

You can check which SQL drivers are installed with the following command:

.. code-block:: bash
  
  $ tspawn --show-drivers
    Available database drivers for Qt:
    QSQLITE
    QMYSQL3
    QMYSQL
    QODBC3
    QODBC

The pre-built SQL driver can be used for SQLite, although the SQLite driver can also be used with a little effort.

Specifying a Template System
----------------------------

In TreeFrog Framework, we can specify either Otama or ERB as a template system. We will set the TemplateSystem parameter in the development.ini file.

.. code-block:: ini
  
  TemplateSystem=ERB
  #or
  TemplateSystem=Otama

Automatic Generation of Code Made from the Table
------------------------------------------------

From the command line, run the command generator (tspawn) which will generate the underlying code. The example below shows production of controller, model, and view. The table name is specified as the command argument.

.. code-block:: bash

    $ cd blogapp
    $ tspawn scaffold blog
      created  controllers/blogcontroller.h
      created  controllers/blogcontroller.cpp
      created  controllers/controllers.pro
      created  models/sqlobjects/blogobject.h
      created  models/blog.h
      created  models/blog.cpp
      created  models/models.pro
      created  views/blog/index.html

Using the tspawn options, either model only, or controller, can be generated. 
 
Build the Source Code
---------------------

To start the build process, run the following command only once; it will generate a Makefile.

.. code-block:: bash

  $ qmake -r "CONFIG+=debug"

*A WARNING* message will be displayed, but there is no problem. Next, run the make command to compile the controller, model, view, and helper.

.. code-block:: bash
  
  $ make     (On Windows run 'mingw32-make' command instead)

If the build succeeds, four shared libraries (controller, model, view, helper) will be created in the lib directory. By default, the library is generated in debug mode; however, you can regenerate the Makefile, using the following command, to create a library in release mode.

Creating a Makefile in release mode:

.. code-block:: bash
  
  $ qmake -r "CONFIG+=release"

To Start the Application Server
-------------------------------

Change to the root directory of the application before starting the application server (AP server). The server will start to process what it regards as the application root directory to the directory where the command is run. To stop the server, press Ctrl+c.

.. code-block:: bash
   
  $ treefrog -e dev

In Windows, start by using treefrogd.exe. 

.. code-block:: bat
  
  > treefrogd.exe -e dev

In Windows, start by using treefroged.exe when you build web applications in debug mode, and start by using treefrog.exe when you want to build a web application in release mode. *Release and debug modes should not be mixed, as the result will not work properly*.

If you want it to run in the background, use the option -d together with any other required options.

.. code-block:: bash
  
  $ treefrog -d -e dev

The command option '-e'  appears in the above examples. When this is followed by a section name you have specified in database.ini, it can be used to change the database settings. If no section name is specified it is assumed that the command refers to a product (when the project is being made, the following three are defined). 

+---------+------------------------------------------------+
| Section | Description                                    |
+=========+================================================+
| dev     | For generator, development                     |
+---------+------------------------------------------------+
| test    | For test                                       |
+---------+------------------------------------------------+
| product | For official version, production version       |
+---------+------------------------------------------------+

'-e' comes from the initials letter of “environment”.

Stop command:

.. code-block:: bash
  
  $ treefrog -k stop

Abort command (forced termination):

.. code-block:: bash
  
  $ treefrog -k abort

Restart command:

.. code-block:: bash
  
  $ treefrog -k restart

If the firewall is in place, make sure that the correct port is open (the default is port 8800).

Browser Access
--------------

We will now use your web browser to access http://localhost:8800/Blog. A list screen, such as the following should be displayed.

Initially, there is nothing registered.

.. image:: images/ListingBlog.png
   
When two items are registered the options to show, edit, and remove become visible. As you can see, there is no problem in displaying Japanese text.

.. image:: images/ListingBlog2.png

TreeFrog is equipped with a call method mechanism (Routing system) for the appropriate controller from the requested URL to the action (as well as other frameworks).
Developed source code can work on other platforms, if it is re-built.

To see a sample Web application. `Go here <http://treefrogframework.org:8800/Blog/index/>`.
You can play with this and it will respond at the same speed as the average desktop application.

Source Code of Controller
-------------------------

Let's take a look at the contents of the controller which is generated.
First, the header file. There are several charm codes, but these are required for sending by URL.

The purpose of public slots, is to declare the actions (methods) you want to send. Actions corresponding to the CRUD are defined. Incidentally, the slots keyword is a feature of the Qt extension. Please see the Qt documentation for details.

.. code-block:: c++
  
  class T_CONTROLLER_EXPORT BlogController : public ApplicationController
  {
      Q_OBJECT
  public:
      BlogController() { }
      BlogController(const BlogController &other);
              
  public slots:
      void index();                     // Lists all
      void show(const QString &pk);     // Shows one entry
      void entry();                     // Registration screen display
      void create();                    // New registration
      void edit(const QString &pk);     // Displays editing screen
      void save(const QString &pk);     // Updates (save)
      void remove(const QString &pk);   // Deletes one entry
  };
  
  T_DECLARE_CONTROLLER(BlogController, blogcontroller)     // Charm

Next, let’s look at the source file. At about 100 lines, it’s a bit long, but please bear with me.

.. code-block:: c++
  
  #include "blogcontroller.h"
  #include "blog.h"

  BlogController::BlogController(const BlogController &)
      : ApplicationController()
  { }

  void BlogController::index()
  {
      QList<Blog> blogList = Blog::getAll();  // Get a list of all Blog objects
      texport(blogList);             // Pass the value to a view
      render();                      // Render the view (template)
  }

  void BlogController::show(const QString &pk)
  {
      Blog blog = Blog::get(pk.toInt()) ;  // Gets Blog model by primary key
      texport(blog);
      render();
  }
  
  void BlogController::entry()
  {
      render();
  }

  void BlogController::create()
  {
      if (httpRequest().method() != Tf::Post) {   // Checks that the method is POST
          return;
      }
                                                      
      Blog blog = Blog::create( httpRequest().parameters("blog") );  // Creates the object from POST information
      if (!blog.isNull()) {
          QString notice = "Created successfully.";
          tflash(notice);                           // Sets the flash message
          redirect(urla("show", blog.id()));        // redirect to show action
      } else {
          QString error = "Failed to create.";     // Object creation failure
          texport(error);                         
          render("entry");
      }
  }

  void BlogController::edit(const QString &pk)
  {
      Blog blog = Blog::get(pk.toInt());  // Get a Blog object for editing
      if (!blog.isNull()) {
          texport(blog);
          session().insert("blog_lockRevision", blog.lockRevision());  // Save the lock-revision to session
          render();

      } else {
          redirect(urla("entry")); 
      }
  }

  void BlogController::save(const QString &pk)
  {
      if (httpRequest().method() != Tf::Post) {
          return;
      }
      
      QString error;
      int rev = session().value("blog_lockRevision").toInt();  // Gets the lock revision
      Blog blog = Blog::get(pk.toInt(), rev);  // Get a Blog object for update
  
      if (blog.isNull()) {
          error = "Original data not found. It may have been updated/removed by another transaction.";
          tflash(error);
          redirect(urla("edit", pk));
          return;
      }
      
      blog.setProperties( httpRequest().parameters("blog") );  // Sets the requested data
      
      if (blog.save()) {                   // Saves the object
          QString notice = "Updated successfully.";
          tflash(notice);
      } else {
          error = "Failed to update.";
          tflash(error);
      }
      
          redirect(urla("show", pk));      // Redirects to show action 
  }

  void BlogController::remove(const QString &pk)
  {
      if (httpRequest().method() != Tf::Post) {
          return;
      }
      
      Blog blog = Blog::get(pk.toInt());   // Gets a Blog object
      blog.remove();                       // Removes it
      redirect(urla("index"));
  }
  // Don't remove below this line
  T_REGISTER_CONTROLLER(blogcontroller)       // Charm

Lock revision is used to realize the optimistic locking. See "model", later in this chapter, for more information.
  
As you can see, you can use the texport method to pass data to the view (template). The argument for this texport method is a QVariant object. QVariant can be any type, so int, QString, QList, and QHash can pass any object. For details on QVariant, please refer to the Qt documentation.

View Mechanism
--------------

Two template systems have been incorporated into TreeFrog so far. These are, the proprietary system (called Otama), and ERB. As is familiar from Rails, ERB is used for embedding into HTML.

The default view that is automatically generated by the generator is an ERB file. So, let's take a look at the contents of index.erb. As you can see, the C++ code is surrounded by <% … %>.  When the render method is called from the index action, the content of index.erb is returned as the response.

.. code-block:: html
  
  <!DOCTYPE HTML>
  <%#include "blog.h" %>
  <html>
  <head>
    <meta http-equiv="content-type" content="text/html;charset=UTF-8" />
      <title><%= controller()->name() + ": " + controller()->activeAction() %></title>
  </head>
  <body>
  <h1>Listing Blog</h1>
  <%== linkTo("New entry", urla("entry")) %><br />
  <br />
  <table border="1" cellpadding="5" style="border: 1px #d0d0d0 solid; border-collapse: collapse;">
      <tr>
          <th>ID</th>
          <th>Title</th>
          <th>Body</th>
      </tr>
      <% tfetch(QList<Blog>, blogList); %>
      <% for (QListIterator<Blog> it(blogList); it.hasNext(); ) {
          const Blog &i = it.next(); %>
          <tr>
              <td><%= i.id() %></td>
              <td><%= i.title() %></td>
              <td><%= i.body() %></td>
              <td>
                  <%== linkTo("Show", urla("show", i.id())) %>
                  <%== linkTo("Edit", urla("edit", i.id())) %>
                  <%== linkTo("Remove", urla("remove", i.id()), Tf::Post, "confirm('Are you sure?')") %>
              </td>
          </tr>
      <% } %>
  </table>

*Next, let's look at the Otama template system.*

Otama is a template system that completely separates the presentation logic from the templates. The template is written in HTML and a "mark" element is inserted as the start tag of the section to be rewritten dynamically. The presentation logic file, written in C++ code, provides the logic in relation to the "mark".

The following example is a file, index.html, that is generated by the generator when it is specified in the Otama template system. This can include the file data but you will see, if you open it in your browser as it is, because it uses HTML5, the design does not collapse at all without the data.

.. code-block:: html
  
  <!DOCTYPE HTML>
  <html>
  <head>
    <meta http-equiv="content-type" content="text/html;charset=UTF-8" />
      <title data-tf="@head_title"></title>
    </head>
    <body>
      <h1>Listing Blog</h1>
      <a href="#" data-tf="@link_to_entry">New entry</a><br />
      <br />
      <table border="1" cellpadding="5" style="border: 1px #d0d0d0 solid; border-collapse: collapse;">
        <tr>
            <th>ID</th>
            <th>Title</th>
            <th>Body</th>
            <th></th>
        </tr>
        <tr data-tf="@for">               ← mark '@for'
            <td data-tf="@id"></td>
            <td data-tf="@title"></td>
            <td data-tf="@body"></td>
            <td>
                <a data-tf="@linkToShow">Show</a>
                <a data-tf="@linkToEdit">Edit</a>
                <a data-tf="@linkToRemove">Remove</a>
            </td>
        </tr>
      </table>
    </body>
  </html>

A custom attribute called 'data-tf' is used to turn on the "mark". This is a Custom Data Attribute as defined in HTML5. A string beginning with "@" is used as the value for the "mark".

Next, let's look at the index.otm corresponding to the presentation logic.
The mark, which links to the associated logic, is declared in the above template, and continues in effect until a blank line is encountered. The logic is contained in the C++ part of the code.

Operators (such as == ~ =) are also used. The operators control different behaviors  (for more information see the following chapters).

.. code-block:: c++
  
  #include "blog.h"  ← This is as it is C++ code to include the blog.h
  @head_title ~= controller()->controllerName() + ": " + controller()->actionName()

  @for :
  tfetch(QList<Blog>, blogList);  /* Declaration to use the data passed from the controller */
  for (QListIterator<Blog> it(blogList); it.hasNext(); ) {
      const Blog &i = it.next();        /* reference to Blog object */
      %%      /* usually, for loop statements, to repeat the child and elements  */
  
  @id ~= i.id()   /* assigns the results of i.id()  to the content of the element marked with @id */

  @title ~= i.title()

  @body ~= i.body()

  @linkToShow :== linkTo("Show", urla("show", i.id()))  /* replaces the element and child elements with the results of linkTo() */

  @linkToEdit :== linkTo("Edit", urla("edit", i.id()))

  @linkToRemove :== linkTo("Remove", urla("remove", i.id()), Tf::Post, "confirm('Are you sure?')")

  @linkToEntry :== linkTo("New entry", urla("entry"))

The Otama operators, (and their combinations) are fairly simple:
~  (tilde) sets the content of marked elements to the result of the right-hand side,
=  output the HTML escape, therefore ~= sets the content of the element to the results of the right-hand side then HTML-escape, if you don’t want to escape HTML, you can use  ~==.

: (colon) replaces the result of the right-hand child elements and the elements that are marked, therefore :== replaces the element without HTML escape.
 

*Passing Data from the Controller to the View*
If you want to use the textport data in view, the controller (object), must be declared in the tfetch (macro) method. For the argument, specify the variable name and type of the variable. Because it is the same state as immediately before the specified variable is texported, it can be used in exactly the same way as a normal variable. In the presentation logic above, it is used as the actual variable.
 
Here is an example in use ::
  
  Controller side :
    int hoge;
    hoge = ...
    texport(hoge);
      
  View side :
    tfetch(int, hoge);

The Otama system, generates the C++ code based on the presentation file and the template file. Internally, tmake is responsible for processing it. After that the code is compiled, with the shared library as one view, so, the operation is very fast.
 

**HTML Glossary:**
An HTML element consists of three components, a start tag, the content, end an tag. For example, in the typical HTML element,
"<p>Hello</p>",  <p> is the start tag, Hello is the content, and </p> is the end tag.
 
Model and ORM
-------------

In TreeFrog, because it is based on relationships, the model will contain an ORM object, (however, you may want to create such a model with two or more ORM objects); the relationship being has-a. In this respect TreeFrog differs from other frameworks, since it uses "ORM = Object Model" by default.  You can not change this for specific cases. In TreeFrog, ORM object(s) is wrapped by Model object.

An O/R mapper named SqlObject is included by default in TreeFrog. Since C++ is a statically typed language, type declaration is required. Let's take a look at the SqlObject file generated by blogobject.h.

There is a charm code half, but the field in the table is declared as a public member variable. It is close to the actual structure, but can only be used by CRUD or an equivalent method, (create, findFirst, update, remove). These methods are defined in the TSqlORMapper class and in the TSqlObject class.

.. code-block:: c++
  
  class T_MODEL_EXPORT BlogObject : public TSqlObject, public QSharedData
  {
    public:
      int id;
      QString title;
      QString body;
      QDateTime created_at;
      QDateTime updated_at;
      int lock_revision;
                          　
      enum PropertyIndex {
        Id = 0,
        Title,
        Body,
        CreatedAt,
        UpdatedAt,
        LockRevision,
        
      };
    
      int primaryKeyIndex() const { return Id; }
      /*** Don't modify below this line ***/
    
      Q_OBJECT                             // Below is the charm macro
    
      Q_PROPERTY(int id READ getid WRITE setid)
    
      T_DEFINE_PROPERTY(int, id)
    
      Q_PROPERTY(QString title READ gettitle WRITE settitle)
    
      T_DEFINE_PROPERTY(QString, title)
    
      Q_PROPERTY(QString body READ getbody WRITE setbody)
    
      T_DEFINE_PROPERTY(QString, body)
    
      Q_PROPERTY(QDateTime created_at READ getcreated_at WRITE setcreated_at)
    
      T_DEFINE_PROPERTY(QDateTime, created_at)
    
      Q_PROPERTY(QDateTime updated_at READ getupdated_at WRITE setupdated_at)
    
      T_DEFINE_PROPERTY(QDateTime, updated_at)
    
      Q_PROPERTY(int lock_revision READ getlock_revision WRITE setlock_revision)
    
      T_DEFINE_PROPERTY(int, lock_revision)
    
    };

There are methods to query and update the primary key in the TreeFrog’s O/R mapper, but the primary key SqlObject can have only one return primaryKeyIndex () method. Therefore, any table with multiple primary keys should be corrected to return one only. It is also possible to issue more complex queries by using the TCriteria class condition. Please see following chapters for details.

Next, let's look at the model.
The setter/getter for each property and static method of generation/acquisition of the object are defined.  The parent class TAbstractModel defines the methods to save and to remove, because of this, the Blog class is equipped with the CRUD methods (create, get, save, remove) .

.. code-block:: c++
  
  class T_MODEL_EXPORT Blog : public TAbstractModel
  {
    public:
      Blog();
      Blog(const Blog &other);
      Blog(const BlogObject &object);  // constructor made from ORM object
      ~Blog();
     
      int id() const;    // The following lines are the setter/getter 
      void setId(int id);
      QString title() const;
      void setTitle(const QString &title);
      QString body() const;
      void setBody(const QString &body);
      QDateTime createdAt() const;
      QDateTime updatedAt() const;
      int lockRevision() const;
      
      static Blog create(int id, const QString &title, const QString &body);  // object creation
      static Blog create(const QHash<QString, QString> &values);    // object creation from Hash
      static Blog get(int id);        // Gets object specified by ID
      static Blog get(int id, int lockRevision);
      static QList<Blog> getAll();      // Gets all model objects
      
    private:
      QSharedDataPointer<BlogObject> d;   // a pointer to the ORM object
      TSqlObject *data();
      const TSqlObject *data() const;
      
  };
  Q_DECLARE_METATYPE(Blog)        // Charm, from here
  Q_DECLARE_METATYPE(QList<Blog>)

Despite the fact that the number of code steps automatically generated by the generator is not high, all the basic functions are covered.

Of course, automatically generated code is not perfect. Real life applications it may need to be more complex. The code may not be sufficient as generated, so some reworking may be necessary. Nevertheless, the generator will save a little time and effort in writing code.

In the background, the code as described above also functions to provide; CSRF measures with cookie tampering check, optimistic locking, and token authentication against SQL Injection. If you are interested, please look into the source.

Video Demo – Sample blog Application Creation
---------------------------------------------

`Create Sample Blog App Demo on TreeFrog Framework <http://www.youtube.com/watch?feature=player_embedded&v=M_ZUPZzi9V8>`_
