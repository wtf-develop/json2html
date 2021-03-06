# json2html
Lightweight JavaScript library for single page web applications with Multilingual support. Minified framework version is less than 20K with zero dependencies.

Implemented: templating, components, network requests, translation on fly. This is all what usually necessary in most of projects. **json2html.min.js** - can work with old browsers.

# Basic сontent
- [Installation](#installation)
- [Basic example](#basic-example)
    - [Templates](#template-file)
    - [Rendering](#rendering-and-network-requests)
- [Advanced](#advanced-content)

# Installation
jQuery is NOT necessary.

Add to the end of file just before \</BODY\> tag:
```javascript
<script type="text/javascript" src="json2html.min.js"></script>
<script type="text/javascript">
    function loadingCallback() {
        buildWebUI(); // build website-ui as in "Basic example" section
    }

    //Run this immediately after loading page
    jth.setTranslationArray(translates.en); // optional: set translation map BEFORE loadTemplatesArray()
    jth.loadTemplatesArray(["html/templates.html"], loadingCallback); // loading files with templates by URL
    //...
</script>
```
You can use **"json2html"** name or **"jth"** for access to [library methods](#basic-only-3-methods). In examples below "jth" prefix was used.

# Basic example
## HTML Template file
After installation create a file like **html/templates.html**. It's a simply HTML file with 3 special placeholers inside:
- **{{variable}}** - [insert value from JSON data](#using-variable)
- **{:template:}** - [just show template](#using-template)
- **{+loop+}** - [process arrays](#using-arraytemplate)

For translation to another language you can use:
- **@str.example** - [will be replaced](#multilanguage-support) from setTranslationArray() with key ["example"]

Content of the file with templates can look like this example:
```html
NextTemplateName: header
....

NextTemplateName: page
{:header:}
<h1>@str.example</h1>
<ul>
    {+items+}
    <li>{{name}}</li>
    {+/items+}
</ul>
{:footer:}

NextTemplateName: footer
....
```
Where **NextTemplateName: page** it is spliter between different templates in one file. And **page** it's name of template for render(), inject2DOM() and $.injectJSON() functions.


## Rendering and Network requests
Library implemented FETCH with fallback to XMLHttpRequest. Example below show standard way of working with server response.
### Without jQuery:
```javascript
jth.getJSON("api/get_info.php",function (json){ //Network request
    if (isGoodResponse(json)) {
        jth.inject2DOM(json,"page","#content"); //Render and run all inline <script>
    }
});
```

### With jQuery:
```javascript
jth.getJSON("api/get_info.php",function (json){ //Network request
    if (isGoodResponse(json)) {
        $("#content").injectJSON(json,"page"); //Render and run all inline <script>
    }
});
```

### Advanced:
```javascript
jth.getJSON("api/get_info.php",function (json){ //Network request
    if (isGoodResponse(json)) {
        let element=document.getElementById('content');
        let html=jth.render(json,"page"); //Render
        element.innerHTML=html;
        jth.executeJS(element); //optional: run all inline <script>
    }
});
```

.

.

.

.

.

.

.

# Advanced content
**All information below is for advanced library usage. If you want to use only simplest features, you can skip it**.
- [Methods List](#methods-list)
    - [Basic](#basic-only-3-methods)
    - [Extentions](#additional)
    - [jQuery](#jquery)
    - [Debug](#debug)
- [Some examples and explanations](#parsing-json-into-html)
    - [**{{variable}}**](#using-variable)
    - [**{:template:}**](#using-template)
    - [**{+loop+}**](#using-arraytemplate)
- [Loading templates/components](#loading-templates)
- [Multilanguage support](#multilanguage-support)
- [Recommended code structure](#recommended-code-structure)


# Methods List
## Basic (only 3 methods)
* **loadTemplatesArray**(["url1","url2"...], function(){..})
    ```
    load multi-files templates with callback. Result in "loaded_templates" variable

    ```

* **render**(json_data, "template_name")
     ```
    return is a HTML string created from HTMLs loaded by loadTemplatesArray(...)

    ```

* **setTranslationArray**(language_array)
    ```
    set translation array with keys as last part of "@str.key_name".
    Must be generated on server side accordingly to selected language.

    ```
## Additional
* getJSON("url", function(json_data){..})
    ```
    send GET request with callback

    ```

* postJSON("url", json_data, function(json_data){..})
    ```
    send POST request  with callback

    ```

* translate(Object,["ke1","key2"..])
    ```
    Translate all strings in object with keys (optional).
    If you need to translate response from server.
    Templates are translated automatically.

    ```

* serializeHtmlForm(css_selector)
    ```
    serialize froms with unchecked checkboxes and arrays.

    ```

* inject2DOM(json_data, "template_name", css_selector)
    ```
    put created HTML code into DOM element with CSS selector
    Return: array from querySelectors function

    ```

* executeJS(DOM_element)
    ```
    Execute JS inside element once. You don't need it if you use
    only plain HTML without active JavaScript code.

    Will run automatically inside functions:
    jth.inject2DOM(...) and $.injectJSON(...).
    But NOT in jth.render(...), must be called separately. Look example above.

    ```

## jQuery
if jQuery was added to HTML page library will create 2 extentions
* $.serializeHtmlForm()
    ```
    same as serializeHtmlForm(css_selector)

    ```

* $.injectJSON(data,"template_name")    
    ```
    inject HTML to each jQuery element. Return jQuery object.
    Example:
    $('#content').injectJSON(data,'template').fadeIn(300);

    ```

## Debug
* printObject(Object, level)
    ```
    Return String with object content (Look below:{{vardump}}). Level is optional, default=1
    Also for logging in JS-console , you can set flag DEBUG=true at the top of library file.

    ```

* setDebug(boolean)
    ```
    Enable debug mode. Disable debug output to console - setDebug(false);

    ```

# Parsing JSON into HTML
This library make a lot of work for converting data from JSON to HTML.

Example JSON
```javascript
{
    "name":"Name",
    "parameters":[{
        "param1":1,
    },
    {
        "param1":3,
    }]
}
```
### Example-1
we want to show **name** from this JSON inside HTML.
```html
NextTemplateName: head
<h1>{{name}}</h1>
```
And just call **jth.render** like here
```javascript
    var html=jth.render(json,"head");
    $('#content').html(html); //insert result in page
```
### Example-2
```html
NextTemplateName: all_page
{:head:}{:table:}

NextTemplateName: head
<h1>{{name}}</h1>

NextTemplateName: table
<ul>{+parameters,table_row+}</ul>

NextTemplateName: table_row
<li>{{param1}}</li>
```
```javascript
var html=jth.render(json, "all_page");

//---- result ----
//<h1>Name</h1>
//<ul>
//    <li>1</li>
//    <li>3</li>
//</ul>
```
### Example-3
You can generate only one row from template **table_row**
```javascript
var html=jth.render(json.parameters[0],"table_row");

//---- result ----
//<li>1</li>    
```


# Using {{variable}}
Minimal manipulations with variables in JSON data. For **if=** and **replace=** symbol GRAVE ACCENT **`** required, not ' or "
```javascript
{{variable,if=`value||value2`then`TrueString`else`FalseString`}} //- show string depends of value

{{variable,ift=`value||value2||value3`then`TemplateTrue`else`TemplateFalse`}} //- show template depends of value

{{variable,ifb=`1**0*1`then`TrueString`else`FalseString`}} //- show string depends of bit mask. Check each bit to 0 and 1

{{variable,trunc=10}} //- truncate variable to 10 chars

{{variable,replace=`abc`with`def`}} //- replace all "abc" to "def" in variable

{{variable,hash32}} //- show MurmurHash3 of variable uniq for current library instance. Look: http://sites.google.com/site/murmurhash/

{{arr.length}} //- show length of variable if type is Array

{{instance_id}} //- uniq number for current template instance on page. Can be used for components to limit the scope of the Javascript. Components its the same as templates, but with code on javascript directly in the same template.

{{random}} //- display random value in range [1,100000]

{{vardump}} //- will show content of current variable

{{this.vardump}} //- same as {{vardump}} but with keyword "this"

{{parent.variable}} //- display variable one level up by templating process, --!!! NOT BY DATA OBJECT !!!--
```


# Using {:template:}
You can create simply HTML templates or Active Components with JS-code inside.
```javascript
var json={
    "name":"Name",
    "parameters":[{
        "param1":1,
    },
    {
        "param1":3,
    }]
}
```

## HTML Templates
* **Standalong Templates**
```html
--- templates.html---
NextTemplateName: display_first_only
{:table_row,parameters.0:}

NextTemplateName: table_row
<div>{{param1}}</div>
```
**parameters.0** will change current variables scope for template to **parameters[0]** This very usefull if you have same data on different levels.

There is possibility to push JSON as second parameter. Not longer than 200 symbols for placeholder.
```html
--- templates.html---
NextTemplateName: inline_json_example
{:table_row,{"param1":"Hello World!"}:}

NextTemplateName: table_row
<div>{{param1}}</div>
```

* **Inline Templates for loops**
```html
--- templates.html inline style for loops ---
NextTemplateName: table
<ul>
    {+parameters+}
    <li>{{param1}}</li>
    {+/parameters+}
</ul>

--- javascript code ---
var html=jth.render(json,"table");
```

## Components
The same as HTML Templates but with some inital JavaScript code inside.

For components better to use anonymous function and **{{instance_id}}** predefined variable.
```javascript
--- component.html ---
<div id="id{{instance_id}}">
    html content
</div>
<script type="text/javascript">
    // Wrap all components into anonymous function to avoid conflicts
    (function(uniq_id) {
        ...
    }('{{instance_id}}'))
</script>
```


# Using {+array,template+}
Here you can also use if condition (for details please read header of json2html.js).  For **if=** symbol GRAVE ACCENT **`** required, not ' or "
```javascript
{+array,template,if=`condition`+} //- show items by condition. Use "obj." prefix for properties

{+array,template,limit=100+} //- process only first 100 items

{+array,template,default=`template`+} //- show content of template no data in array

{+array,template,defstr=`string`+} //- display string if there is no data in array
```

You can combine all conditions into one
```javascript
{+array,template,if=`condition`,limit=`100`,default=`string`+}
```
Its also possible to use Inline-Style:
```html
{+array,if=`condition`,limit=`100`,default=`string`+}
<li>{{name}}</li>
{+/array+}
```
Don't pass second parameter "template" for inline style. Content of template will be generated from text inside loop tags **{+array+}content{+/array+}**

# Loading templates
You can create a HTML-template files on server and load all files with translated texts with only 2 strings of code.
```javascript

function init() {
    //before loading any templates, you need to set translation array, if you want multilanguage support
    //Inline data from server as parameter e.g. PHP, Python and others, that check Cookie "lang" before generating JSON
    //Or AJAX request to server for json, and only after that you can load templates with "jth.loadTemplatesArray"
    jth.setTranslationArray(<?php echo(json_encode(transtale_array['en'])); ?>)
    jth.loadTemplatesArray(["html/example_template.html", "html/example_text.html"], loadingCallback)
}

function loadingCallback() {
    //if all templates loaded correctly.
    //If this function was not called - some of the file is not exists
    buildWebUI();
}

init(); //Run it immediately after loading page HTML content
```

You can put few templates into one file separated by special keyword  **NextTemplateName:**

Example **few_templates.html**
```html
NextTemplateName: users_table
<table>{+users,users_table_items+}</table>


NextTemplateName: users_table_items
<tr>
    <td nowrap width=95%>
        <a href=# onClick="return edit({{id}});">{{login,trunc=`30`}} : {{info}}</a>
    </td>
    <td width=5% onClick="return remove({{id}});">
        <img src="remove_icon.png" />
    </td>
</tr>
```
So it's possible to put all templates together in one file.
FileName will be ignored and only name after **NextTemplateName:** will be taken. You can use this names for rendering data


# Multilanguage support
All templates are translated automatically while loading, if there is a keywords present in format "@str.array_key"

Key name not longer than 40 simbols, only english simbols, digits, underscore "_"

Example:
```javascript
//Set translation array. "@str.login_name" will be replaced with "User"
jth.setTranslationArray({
    login_name:"User",
    ...
});
//All templates are translated right after loading. You don't need to do anything additionally
jth.loadTemplatesArray(["templates_url"],drawUI);

function drawUI(){
    //...
}
```

Example of template with **@str.** prefix-strings
```html
NextTemplateName: users_table_item
<tr>
    <td>
        <a href=#>@str.login_name: {{uname}}</a>
    </td>
</tr>
```

If you need to translate some response, you can use function "jth.translate(json,[keys])". Keys parameter is optional.

# Recommended code structure
I suppose better to use iframes for each module. In this case you will never have a conflicts
and don't need to load all JS-libraries that needed in your WHOLE project.


* **/common**
    * /api - (common functions for server JSON API on PHP/Python)
    * /html - (templates and components for whole project)
    * /js - (here can be implemented function **isGoodResponse(json)** and others)
    * /css
    * /img
* **/module1**
    * /api - (data response from server database)
        * api_result.php
        * ...
    * /html - (templates and components in current module)
        * page_structure.html
        * search_input_component.html
        * ...
    * index.php - (control all events and process all data from server - Presenter)
* **/module2**
    * /api
    * /html
    * index.php
* .....
* **index.php** (entry point for iframes navigation)

### Schema in each module
Clients browsers became more powerful every year. It's very silly to build all
HTML pages on server side if this work can be done on client with JavaScript.
This will save server time and money.
```
   Browser                 Server
|-----------|            |-------|
|   index   |--- AJAX -->|  API  |
|~~~~~^~~~~~|            |-------|
|~~~~~|~~~~~|                |
|   HTML    |<-----JSON------|
|-----------|
```
