
.. _erb:

====
ERB
====
ERB was originally a library for embedding Ruby script into text documents. As is well known, it has been adopted as (one of the) template engines such as Rails, you can embed the code in HTML between the tags, <% … %>.

In the same way TreeFrog Framework, uses the tags <% … %> , to embed C++ code. For convenience, I'll refer to this implementation as ERB as well.
 
First, I'll make sure that the items in the configuration file *development.ini* are as follows. Unless you’ve changed from the default, they should be.

.. code-block:: ini
  
  TemplateSystem=ERB

Then, when you generate a view in the command generator, the ERB template format will be generated. File name of the template that is generated must follow the rule that it should all be in lower case.

::
  
  views/Controller-name/Action-name.erb    (In lower case)

If you want to add a new template, please follow the same naming convention.
 
Relationship of View and Action
-------------------------------

I would like to review the relationship between view and action. As an example, when the bar action of Foo controller calls the render() method without arguments, the content of the following template is output.

::
  
  views/foo/bar.erb

If you call the render () method with an argument, the contents of the appropriate template is output.
 
Template files that you like can be added. Once you have added a new one, run once the following command once in the view directory. It’s all that’s needed in order for the new template to added to the makefile entry of build.

.. code-block:: bash
  
  $ cd views 
  $ make qmake

Remember to reflect the new template that you’ve added in the shared library.

**In brief: After you add a template file use "make qmake".**
  
To Output a String
------------------

We are going to the output the string "Hello world". There are two ways to do this. First, we can use the following method.

::
  
  <% eh("Hello world"); %>

This eh() method prints the string using the HTML escape for cross-site scripting protection. If you do not want to use escape processing, use the echo() method instead.

Another way is to use the notation <%= … %>. This produces exactly the same results as using the eh() method does.

::
  
  <%= "Hello world" %>

Again if you do not want to use escape, you can use <%== … %> which gives exactly the same results as using the echo() method.

::
  
  <%== "<p>Hello world</p>"  %>

**Note:**
When you write <%= … %>  in the original (eRuby) specification it outputs a string WITHOUT HTML escape. In TreeFrog, the thinking is that by using the shorter code for indicating HTML escape, safety is increased, based on the possibility that extra bits of code can easily be omitted in error. It seems to be becoming the mainstream in recent years.

Display of Default Value
------------------------

If a variable is an empty string, let's display the default value as an alternative.
By writing as follows, if the variable str is empty, then the string "none" is displayed.

::
  
  <%= str %|% "none" %> 

To Use the Object Passed from the Controller
--------------------------------------------

In order to display the object that is exported from the controller by texport() method,  first declare type(class) and variable names using the tfetch macro or T_FETCH macro. We’ll refer to this operation as 'fetch'.

::
  
  <% tfetch(Blog, blog); %>
  <%= blog.title %>       ← output tha value of blog.title 

The variable (object) that has been fetched, can be accessed later in the same file. In other words, it becomes a local variable.
 
You might ask if fetch processing has to be processed each time of using, but you could see that the fetch processing does not have to be run every time you want to use the variable.
  
The fetched object is a local variable in a function. Therefore, if we fetched the same object twice, because the variables are the same we would have defined twice, giving an error at compile time. Of course, it would not be a compile-time error if we divided the block, but it probably would not make much sense to do so.

   **In brief: Use the fetch case only once.**

If you export objects of the type int and QString type, then you can only output in the tehex function. Fetch processing eliminates the need for it to be this way.

::
  
   <% tehex(foo); %>

Understand that tehex() is, in effect, a combination of  the eh() method and fetch. There is a difference in defining between fetching and local variables, as you can see in here, function variable (foo in this example) outputs the value without defining 
  
If you do not want to escape the HTML processing, use the techoex() function, which is a combination of echo() and fetch() methods. In the same way, the variable is not defined.
  
In addition, there is another way to export the output objects. You can use the code <=$ .. %>. Note that there must be no space between '$' (dollar) and '=' (equal). 
This produces exactly the same result.

::
  
  <%=$ foo %>

Codes have been considerably simplified. 
This means that you can replace the tehex() method with the "=$" code. 
Similarly, <% techoex(..); %> can be rewritten in the notation <==$  .. %>.

To sum up, to export an object of int type or QString type, I think it’s better to output using the notation <=$ .. %>, unless you want to output just once (in which case, use the fetch process ).

    **In brief: Use  <=$ .. %>  to export objects that do not output only once.**

How to Write Comments
---------------------

The following is an example of how to write a comment. Nothing is output as HTML. However, its contents will remain in the C++ code.

::
  
  <%# comment area %>

After the C++ code is placed in the *views/_src* directory, it is compiled. Look here if you want to refer to the C++ code.

Include Files
-------------

If you use a class, such as a model, in the ERB template, you will need to include the header file in the same way as with C++. Note that it is not automatically included.
Include these next.

::
  
  <%#include "blog.h" %>

Note that there must be no space between ‘#’ and ‘include’. In this example, the blog.h file is to be included.

Remember that the template is converted to C++ code as it is, so don’t forget to include the template file.

Loop
----

Let's write a loop to use on a list. For example, take the list blogList which is made up of objects called Blog, it looks like this. (If it is exporting an object, do the fetch processing in advance.)

::
  
  <% QListIterator<Blog> i(blogList);
  while ( i.hasNext() ) {
      const Blog &b = i.next();  %>
      ...
  <% } %>

You can use the foreach statement from Qt comes to make the coding shorter.

::
  
  <% foreach (Blog b, blogList) { %>
  ...
  <% } %>

This looks more like C++.

Creating an <a> Tag
-------------------

To create an <a> tag, use the linkTo() method.

::
  
  <%== linkTo("Back", QUrl("/Blog/index")) %>
                ↓
  <a href="/Blog/index">Back</a>

In the linkTo() method, other arguments can be specified; see the `API reference <http://www.treefrogframework.org/api-reference>`_ for more information.

You could also use the url() method to specify a URL. Specify the controller name, with the url as the first argument, and the action name as the second argument.

::
  
  <%== linkTo("Back", url("Blog", "index")) %>
                ↓
  <a href="/Blog/index/">Back</a>

If the template is on the same controller, you use the urla() method and specify only the action name.

::
  
  <%== linkTo("Back", urla("index")) %>
                ↓
  <a href="/Blog/index/">Back</a>

Using JavaScript, the link and confirmation dialog can be written as follows.

::
  
  <%== linkTo(tr("Delete"), urla("remove", 1), Tf::Post, "confirm('Are you sure?')") %>
                        ↓
  <a href="/Blog/remove/1/" onclick="if (confirm('Are you sure?')) { 
            var f = document.createElement('form');
            :  (omission)
             f.submit();
  } return false;">Delete</a>

Now let’s add an attribute to the tag, using the THtmlAttribute class.

::
  
  <%== linkTo("Back", urla("index"), Tf::Get, "", THtmlAttribute("class", "menu")) %>
                   ↓
  <a href="/Blog/index/" class="menu">Back</a>

You can use the short  a() method code to generate the same THtmlAttribute output.

::
  
  <%== linkTo("Back", urla("index"), Tf::Get, "", a("class", "menu")) %>

If there is more than one attribute, use '|' operator.

By the way, there is the anchor() method, that aliases the linkTo() method; they can be used in exactly the same way.

In addition, many other methods are available, see the `API Document <http://www.treefrogframework.org/api-reference>`_ .

Form
----

We are going to post the data to the server from a browser form. In the following example, we use the formTag() method to post data to create action on the same controller.

::
  
  <%== formTag(urla("create"), Tf::Post) %>
  ...
  ...
  </form>

You may think that you can do the same thing by describing the form tag, rather than the formTag() method, but when you do use the method it means that the framework can do CSRF measures. From a security point of view, we should do so. To enable this CSRF measures, please set the items as follows in the configuration file application.ini.

.. code-block:: ini
  
  EnableCsrfProtectionModule=true

This framework makes it illegal to guard post data. However, following various repeated tests during development, I’m be sure that you can disable the protection temprarily.
  
For more information on CSRF measures, see also the chapter :ref:`security <security>`.

Layout
------

The layout is a template on which to outline the design commonly used in the site. It is not possible to place an HTML element freely in the layout, except in the header part, menu section, and footer section.

There are four ways to interact with the layout when the controller requests view, as follows;

1. Set the layout for each action.
2. Set the layout for each controller.
3. Use the default layout.
4. Do not use layout.

Only one layout is used when drawing the view, but a different layout can be used if the above list gives it a higher priority. So, for example, this means that rather than "layout that is set for each controller," being used, the rule "layout is set for each action" takes precedence.
   
Let’s take the example of a very simple layout (as per the following). It is saved with the extension .erb. The location of the layout is the view/layouts directory.

.. code-block:: html
  
  <!DOCTYPE HTML>
  <html>
  <head>
  <meta http-equiv="content-type" content="text/html;charset=UTF-8" />
      <title>Blog Title</title>
  </head>
  ...
  <%== yield() %>
  ...
  </html>

The important part here is the line; <%== yield(); %>. This line outputs the contents of the template. In other words, when the render() method is called, the content of the template will be merged into the layout.
 
By using a layout like this, you can put together a common design, such as headers and footers, for the site. Changing the design of the site then becomes much easier, because all you need to change is the layout.

  **In brief: Layout is the overall outline for the site design.**

Now, I will describe each method to set the layout.

1. Set the layout for each action
   You can set the layout name as the second argument of the render() method when it is called in the action. 
   This example layout *simplelayout.erb* is used here.

.. code-block:: c++
  
  render("show", "simplelayout");

Now, the contents of the show template, merged into the layout of simplelayout, will be returned as a response.

2. Set the layout for each controller
   In the constructor of the controller, call the setLayout() method, and specify the layout name as an argument.

.. code-block:: c++
  
  setLayout("basiclayout");  // basiclayout.erb the layout used

3. Use the default layout
The file *application.erb* is the default layout file. If you do not specify a particular layout, this default layout is used.

4. Do not use layout
If none of the above three conditions are met, no layout is used. In addition, if you want to specify that a layout is not used, ca set that does not use a layout, use an action to call the function setLayoutDisabled(true).

Partial Template
----------------

If you viewing a website on the net, you will often notice an area of the page which is constant, that is it shows the same content on multiple pages. Perhaps it’s an advertising area, or some kind of toolbar.

To work on a Web application in cases like this, besides the methods discussed above for including such an area in the layout, there is also a way to share by cutting the area as a "partial".

First, cut out the part you want to use on multiple pages, then save it to the views/partial directory as a template, with the .erb extension. We can then draw it with the renderPartial() method.

::
  
  <%== renderPartial("content") %>

The contents of *content.erb* are embedded into the original template and then output. In the partial, you can output the value of export objects as well as the original templates.

From the fact that both are types of merging, the layout and partial template should seem very familiar. What can you do is to use them in a different way.

My own opinion is that it is good to define the layout for sections such as footer and header that are is always present on the page, and to use a partial template to display the parts that often but not necessarily always go on the page.

  **In brief: Cut out the parts that appear frequently as a partial template.**
