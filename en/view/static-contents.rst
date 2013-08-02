===============
Static Contents
===============

Place the static contents that is accessible from the browser in the *public* directory. Only save published files here.

For example, it assumes that an HTML file is in  public/sample.html. 
When you access http://URL:PORT/sample.html from the browser, if the application server (AP server) is running, its content will be displayed.

After creating the skeleton of the application by using the generator command, the following subdirectories will be created.

+----------------+------------------------------+---------------+
| Directory      | File Type                    |  URL path     |
+================+==============================+===============+
| public/images/ | Image files                  | /images/...   |
+----------------+------------------------------+---------------+
| public/js/     | Javascript files             | /js/...       |
+----------------+------------------------------+---------------+
| public/css/    | Cascading Style Sheets (CSS) | /css/...      |
+----------------+------------------------------+---------------+

You can make subdirectories freely within the public directory.

Internet media type (MIME type)
-------------------------------

When the web server returns static content, the rule is to set the internet media type (MIME type) in the response’s content-type header field.  These are the strings such as "text/html" and ”image/jpg”. Using this information, the browser can determine the format in which the data has been sent. 

TreeFrog Framework returns the media type by using the file extension and referring to the defined content in the *config/initializers/internet_media_types.ini* file. In that file, by line, file extensions and internet media types are defined, connected with "=". It is as in the following table.

.. code-block:: ini
  
  pdf=application/pdf
  js=application/javascript
  zip=application/zip

If the Internet media types don’t cover your needs, you can add other types in this file. After doing so, you should restart the AP server to reflect the definition information that you added.

Error display
-------------

The application server is always required to return some response even if some error or exception occurs. In these cases, the status codes for the error responses are defined in `RFC <http://www.ietf.org/rfc/rfc2616.txt>`_ .
In this framework, the contents of the following files will be returned as the response when an error or exception occurs.

+--------------------------+-----------------+
| Cause                    | Static file     |
+==========================+=================+
| Not Found                | public/404.html |
+--------------------------+-----------------+
| Request Entity Too Large | public/413.html |
+--------------------------+-----------------+
| Internal Server Error    | public/500.html |
+--------------------------+-----------------+

By editing these static files, you can change what to display.
 
 
By calling the function as follows from the action, you will be able to return the static file to indicate the error. In this way, when 401 is set as the status code of the response, the contents of *public/401.html* would be returned.

.. code-block:: c++
  
  renderErrorResponse(401);

Further, as another method of displaying an error screen to the browser, it is redirecting to the URL in question.

.. code-block:: c++
  
  redirect(QUrl("/401.html"));

Send a file
-----------

If you want to send a file from the controller, use the sendFile() method. As the first argument, specify the file path, and as the second argument, specify the content type.  The file sent is not required to be in the *public* directory.

.. code-block:: c++
  
  sendFile("filepath.jpg", "image/jpeg");


 If you send a file in this function, then the file download process is implemented on the Web browser side.  A dialog is displayed, asking if the user wants to open or to save the file.
 This function sends the file as HTTP response, it is the same treatment as the render() method. Therefore, the controller can no longer output the template by the render() method.  

 By the way, as for the file path here, if you specify absolute path, it would ensure that it would work. If you use the Tf::app()->webRootPath() function, you can obtain the absolute path to the application directory route, so you can easily create the absolute path to the file. In order to use this function, please include TWebApplication in the header file. 

.. code-block:: c++
  
  #include <TWebApplication>

Send data
---------

To send the data in the memory, instead of a file, you can use the sendData() method as follows. You can omit the access process to reduce the overhead compared to the sendFile() method. 

.. code-block:: c++
  
  QByteArray data;
  data = ...
  sendFile(data, "text/plain");

Similarly, it means the file download operation is executed on the Web browser side, after that you cannot call render() method (it would not work if you did).
