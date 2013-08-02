
.. _url_routing:

URL Routing
===========

The URL Routing is the mechanism that determines the action to call for the requested URL. When a request is received from a browser, the URL is checked for correspondence with the routing rules and, where applicable, the defined action is called. If no action has been specifically allocated, action by the default rule is invoked.
 
Here is a reminder of those default rules.

::
  
  /controller-name/action-name/argument1/argument2/...

Let's look at customizing the routing rules.
The routing definition is written to config/routes.cfg. For each entry, the directives, path, and an action, are written side by side on a single line.  The directive is to select match, get, or post.
In addition, a line that begins with '#' is considered a comment line.
 
Here’s an example::
  
  match  "/index"  "Merge#index"

In this case, if the browser requests '/index' either by POST method or GET method the controller will respond with the index action of *Merge* . 
  
The next case is where the get directives have been defined.

::
  
  get  "/index"  "Merge#index"

In this case, routing will be carried out only when the '/index' is requested with the GET method, (It does not pass received in action). If the request is made by the POST method, it will be rejected.
 
Similarly, if you specify a post directive, it is only valid for POST method requests. Request in GET method will be rejected.

::
  
  post  "/index"  "Merge#index"

The following is about how to pass arguments to the action. Suppose you have defined the following entries as the routing rules.  It's important to use the keyword ':params'.

::
  
  get  "/search/:params"  "Searcher#search"

In the  case of, */search/foo* , when the request is made with the GET method, a search action with one argument is called to Searcher controller. The "foo" in the argument is passed.
Similarly with /search/foo/bar. Following this request, a search action with two arguments is called. Both "foo" and "bar" are passed, as first argument and second argument.

::
  
  /search/foo    ->   Call search("foo") of SearcherController
  /search/foo/bar ->  Call search("foo", "bar") of SearcherController
