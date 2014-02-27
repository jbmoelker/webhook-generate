webhook-generate
================

Grunt Static File Generator

Grunt Commands
----------------------

The following grunt commands are supported:

  * clean - Cleans the build directory
  * buildTemplates - Builds just the templates directory into the build directory
  * buildPages - Builds just the pages directory into the build directory
  * scaffolding - Generates scaffolding for a passed in content type (i.e. grunt scaffolding:Article will generate scaffolding for the Article type)
  * watch - Watches changes in template files and rebuilds site when files are changed
  * watchFirebase - Watches changes in firebase data and rebuilds site when data is changed
  * build - Cleans build directory and builds site
  * init - Initializes site with credential information

File Structure
----------------------

Templates are split into two folders: pages and templates.

The pages folder is intended to hold templates that will directly be built into the build directory. This is useful for things like front door, contact pages, and other one off pages. For example /pages/index.html will become the root of the built site (www.site.com or www.site.com/index.html), whereas /pages/contact.html will become www.site.com/contact.html.

The templates folder is intended to hold files that use content types from firebase, and the generator only will recognize two types of files from this directory 'list.html' and 'individual.html' files.

Both of these files can be generated by scaffolding. The 'list.html' file should contain the template for the door to a content type, and will be rendered into the index of a content type (Article/list.html becomes /article/index.html on the built site). The 'individual.html' template should contain the template for an individual instance of a content object, a page will be rendered for each object of that content type at a location based off its id (Article/individual.html becomes /article/<id>/index.html when rendered).

Finally in the template directory is a special directory called 'partials' that is ignored by the build system. This directory is intended to house any partials that need to be included in other templates.

All website hosting will render the index.html of a directory as that directory if queried. e.g. www.site.com will look for /index.html and www.site.com/contact/ will look for www.site.com/contact/index.html


Template Basics
-----------------------

In addition to the standard template filters and tags provided by Swig, we have included several helper functions and filters to aid in development. They are documented below.

Template Variables:
  * CURRENT_URL - Contains the current url of the template e.g {{ CURRENT_URL }}

Template Functions:
  
  * `get(contentType1, contentType2, contentType3, ...)` - Gets data for a content type (or multiple mixed content types) e.g. {% set data = get('type1') %}
  * `getTypes` - Returns a list of all content types that are not one offs. e.g. {% set types = getTypes() %}
  * `paginate(data, perPage, urlPrefix)` - Tells the templating system that this data needs to be paginated, and will cause the engine to render this page into several pages. perPage is the number of items per page. urlPrefix is the prefix you want to use for the page url (default is /page-<pageNum>/). ONLY ONE SET OF DATA CAN BE PAGINATED PER TEMPLATE. e.g {% set pageData = paginate(data, 10) %}
  * `getCurPage` - Returns the current page in a paginated template. e.g. {% set curPage = getCurPage() %}
  * `getMaxPage` - Returns the max number of pages in a paginated template, only works after paginate. e.g. {% set maxPage = getMaxPage() %}
  * `getPageUrl(pageNum)` - Returns the full url for a page num. e.g. {% set curUrl = getPageUrl(curPage) %}
  * `url(object)` - Returns the full url to an object (or type from getTypes). e.g. {{ url(object) }}

Template Filters:

  * `upper` - Makes a string entirely upper case e.g. {{ object.name|upper }}
  * `slice(offset, limit)` - Slices the array/object returning a set of data from offset up to limit items e.g. {% set data = getData('articles')|slice(0, 10) %}
  * `sort(property)` - Sorts an array by the given property e.g. {% set data = getData('articles')|sort('name') %}
  * `reverse` - Reverses an array or object of keys e.g. {% set data = getData('articles')|sort('name')|reverse %}
  * `imageSize(width, height)` - Takes an image url, returns url of image with resize information on it e.g. {{ object.image|imageSize(50, 50) }}
  * `imageCrop(width, height)` - Takes an image url, returns url of image with crop information on it e.g. {{ object.image|imageCrop(50, 50) }}
  * `size` - Returns the number of elements in a list e.g. {{ data|size }}


Examples
-------------------------

Paginating a list of articles by publish date, 10 per page:

```
{% set articles = get('article')|sort('publishDate')|reverse %}
{% set articles = paginate(articles, 10) %}

{% for article in articles %}
  {{ article.title }}
{% endfor %}
```

Getting the first 10 objects in a mixed set of articles, videos, and podcasts:

```
{% set content = get('article', 'video', 'podcast')|sort('publishDate')|reverse|slice(0, 10) %}
{% for object in content %}
  {{ content.title }}
{% endfor %}
```