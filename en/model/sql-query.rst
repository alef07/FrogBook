
.. _model_sql_query:

SQL Query
=========

SqlObject function is sufficient when you need to read a simple record, but when the processing is complex such as when relating to multiple tables, you may want to publish an SQL query directly. In this framework, it is possible to generate a query safely by using a placeholder.

The following is an example of issuing a query using placeholders for ODBC format.

.. code-block:: c++
  
  TSqlQuery query("INSERT INTO blog (id, title, body) VALUES (?, ?, ?)");
  query.addBind(100).addBind(tr("Hello")).addBind(tr("Hello world"));
  query.exec();  // Query execution

Here is an example using a named placeholder.

.. code-block:: c++
  
  TSqlQuery query("INSERT INTO blog (id, title, body) VALUES (:id, :title, :body)");
  query.bind(":id", 100).bind(":title", tr("Hello")).bind(":body", tr("Hello world"));
  query.exec();  // Query execution

How to retrieve the data from the query result is the same as the QSqlQuery class of Qt.

.. code-block:: c++
  
  TSqlQuery query("SELECT id, title FROM blog");
  query.exec();  // Query execution
  while (query.next()) {
      int id = query.value(0).toInt(); //  Convert the field first to int type
      QString str = query.value(1).toString(); // Convert the second field to QString type
      // do something 
  }

The same method can be used for the TSqlQuery class because it inherits the `QSqlQuery class <http://doc.qt.digia.com/4.8-snapshot/qsqlquery.html>`_ of Qt.  For more information, please refer to the Qt documentation.

**In brief: Queries can be generated using placeholders in all cases.**

In fact, you can see in the :ref:`query log <helper_reference_logging>` any query that has been executed.

Reading a Query From a File
---------------------------

Because it's necessary to compile after every time you write or modify a query statement in the source code, you might find it a bit of a hassle during application development. To alleviate this, there is a mechanism for writing only query statements to a separate file to be loaded at runtime.

The file is placed in the *sql* directory (however, the directory can be changed through *application.ini* ). The *insert_blog.sql* is temporary, I'll describe the contents below.

.. code-block:: mysql
  
  INSERT INTO blog (id, title, body) VALUES (?, ?, ?)

The following is the source code. We will read the *insert_blog.sql* file with the load() method.

.. code-block:: c++
  
  TSqlQuery query;
  query.load("insert_blog.sql")
  query.addBind(100).addBind(tr("Hello")).addBind(tr("Hello world"));
  query.exec();  // Query execution

The cache works inside the load() method (but only when thread module is applied in :ref:`MPM <performance>`). The query is read from the file only on the first occasion, after that it is used from the cache memory, so it then works at high speed. 
After the file has been updated, we need to restart the server in order to read the query statement.

.. code-block:: bash
  
  $ treefrog -k abort ;   treefrog -d  -e dev

Or like the following.

.. code-block:: bash
  
  $ treefrog -k restart

Get an ORM object from the Result of a Query
--------------------------------------------

In the above method, it is necessary to retrieve the value of every field from the results of the query; however, single records can be extracted as ORM objects in the following manner.
 
Run the query using the TSqlQueryMapper object. Then extract the ORM object from the results using an iterator. It’s important to specify the 'blog. \*' In the SELECT statement, so as to select and target all fields.

.. code-block:: c++
  
  TSqlQueryORMapper<BlogObject> mapper;
  mapper.prepare("SELECT blog.* FROM blog WHERE ...”);
  mapper.exec();  // Query execution
  TSqlQueryORMapperIterator<BlogObject> it(mapper);
  while (it.hasNext()) {
      BlogObject obj = it.next();
      // do something
      :
  }")

If you need to extract only one, you can get the results using the findFirst() method.  
