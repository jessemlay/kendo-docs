---
title: Intro to Building Apps with Kendo UI And PHP
slug: tutorial-intro-building-apps-with-kendo-ui-and-php
tags: Tutorial
publish: true
---

# Tutorial: Intro to Building Apps with Kendo UI And PHP

This tutorial will walk you through setting up your PHP development environment, connecting to a database, and interacting with your data using Kendo UI.

### Setting Up To Develop In PHP

The development environment that will be used for this consists of a MacBook Pro running Lion.  Install MySQL, PHP and Apache by using [MAMP][3]
(Macintosh Apache MySQL PHP).  Once you do this, you will have a neat
environment setup for running PHP applications with a MySQL database.  MAMP
will run by default on port 8888 for the webserver, and the root folder will
be **htdocs** inside of the **Applications/MAMP** directory.

The next step is to get an IDE.  It’s all text in the end, but a good PHP IDE
is a must.  The Eclipse PHP development environment is recommended for this.  You can pick up
their community tools contribution to Eclipse free [here][5].

Once you download the IDE, you are all setup to start building PHP
applications.

### We Need Some Data

For this tutorial we will be using the Northwind database.

Northwind comes in Microsoft Access and SQL Server flavors, but someone was
nice enough to port it to just about every other database type out there.  You
can download the .sql script [here][7] and run it in to MySQL to create the
database.

### Project Setup

Below is a screenshot of how the project looks in Eclipse.  Normally, you would
put your publicly accessible files in a “public” folder and restrict access to
ones that are server only.  For the sake of simplicity, all the files are
public.

[![1][8]][9]


Let’s look a bit at the structure.

The css files for Kendo UI and images are in the **css** folder.  Additionally, the Metro theme for Kendo UI is being used. The Kendo UI JavaScript files and
jQuery are in a **js** folder.  The startup page for the site is **index.php**.
A **data** folder has been manually added.  In this folder is where all of the data access code will go.  Each of these files will return a JSON formatted
string of data from the MySQL database.

The next thing to do is to add in the CSS and JavaScript files to the head of
the** index.php **page.


	<!DOCTYPE html> 
		<html> 
			<head> 
				<link href="css/kendo.metro.min.css" rel="stylesheet"> 
				<link href="css/kendo.common.min.css" rel="stylesheet">
				<script src="js/jquery.min.js"></script>
				<script src="js/kendo.all.min.js"></script> </head>


So far this is probably all very familiar if you have done any PHP development
in the past at all.

### Create The Grid

Next, we are just going to add an empty **div** and then turn that div into a
grid by selecting it with a jQuery selector and calling the **kendoGrid** function.
For more information about the basics of working with the grid, see the
“[Getting Started With The Kendo UI Grid][10]” Screencast, or the
[Walkthrough][11].

This is what the whole page looks like so far.  The grid has no columns and no
data.  The next step is to create the .php file which will return the data
from the MySQL database.

	<!DOCTYPE html> 
	<html> 
		<head> 
			<link href="css/kendo.metro.min.css"rel="stylesheet"> 
			<link href="css/kendo.common.min.css" rel="stylesheet">
			<script src="js/jquery.min.js"></script>
	        <script src="js/kendo.all.min.js"></script>
	    </head>
	    <body>
			<div id="grid"></div>
	        <script>
	            $(function() {
	                $("#grid").kendoGrid();
	        </script> 
		</body> 
	</html>

### Connect To The Database

We are going to create a file in the **data** folder called **employees.php**.
This file will connect to the MySQL instance and return all of the employees.
To do this, we need to connect to the database instance using the MySQL PHP
commands.


	1.  <?php
    2.
	3.  $link = mysql_pconnect("localhost", "root", "root") or die("Unable To Connect To Database Server");
	4.  mysql_select_db("northwind") or die("Unable To Connect To Northwind");
	5.
    6.  $arr = array();
	7.
	8.  $rs = mysql_query("SELECT EmployeeID, LastName, FirstName FROM Employees");
	9.
	10.  while($obj = mysql_fetch_object($rs)) {
	11.
	12.		$arr[] = $obj;
	13.
	14. }
	15.
	16. echo "{\"data\":" .json_encode($arr). "}";
	17.
    18. ?>


**Line 3** connects to the MySQL server.  The first parameter is the server
name, the second is the username and the third is the password.

**Line 4** selects the Northwind database.

**Line 8** defines a SQL query and associates the results with the **rs**
variable.

**Lines 10 through 14** iterate over the **rs** variable, which holds a
collection of records and stuffs each record into an array that was defined on
line 5.

**Line 16** returns back the results of the array using the **json_encode**
PHP function which turns the array into formatted JSON.  Additionally, I have
prefixed the array of data with a top level **data** element.  It’s typically
a good idea to not have an array as your top level JSON element.

You could now hit that PHP page directly from your browser and see the
results.  Your URL might be something similar to the following depending on how your development environment is setup.
[http://localhost:8888/KendoUI/data/employees.php][12]

[![2][13]][14]

### Wire Up The Grid

We define a data source for the grid and bind that data source to this URL
which returns the JSON data.  The **transport** defines how the grid will
interact with remote endpoints, and the **read** method defines which URL will
provide the data when the grid is loaded.  Additionally, we are defining columns
for the **LastName** and **FirstName**.


	<body> 
		
		<div id="grid"></div> 
	
		<script>
			
			$(function() {
				$("#grid").kendoGrid({
                    dataSource: {
                        transport: {
                            read: "data/employees.php"
                        },
                        schema: {
                            data: "data"
                        }
                    },
                    columns: [{ field: "FirstName" }, { field: "LastName" }]
                });
            });

        </script> 
	
	</body>

Note that since we are not actually defining anything in the column but the
field, we could have just specified the columns as:

	columns: [ “FirstName”, “LastName” ]

Also notice that we only have to reference **data/employees.php** as the URL to
read because the path is relative to the current application page being served
up.  I also set some **schema** information basically telling the grid that
the repeating information that it needs to display is found in the top level
**data** element.  Let’s save and have a look at it in the browser.

[![3][15]][16]

## There’s No Data….

This was done on purpose to demonstrate a rather important detail.  If you look
in the Developer Tools on Chrome you will see an error saying that “length”
cannot be read off of undefined.

[![4][17]][18]

That’s a good indication that something is wrong with your JSON data.  Either
it isn’t formatted correctly, or you have some other issue.  In this case the
data is formatted correctly, the problem is that when the PHP file gets read,
it’s getting read as HTML and not as JSON.

How do you know this?

Have a look in the “Network Activity” tab in Chrome Developer Tools.  You can
examine the content type there.

[![5][19]][20]

Actually, it is JSON but the server is reporting that it’s HTML.  How do we
fix this?  We just need to specify in our PHP file right before we echo back
that the type is JSON.  We do this by adding a header.  The PHP file now looks
like this…


	<?php

    $link = mysql_pconnect("localhost", "root", "root") or die("Unable To
Connect To Database Server");

    mysql_select_db("northwind") or die("Unable To Connect To Northwind");

    $arr = array();

    $rs = mysql_query("SELECT EmployeeID, LastName, FirstName FROM
Employees");

    while($obj = mysql_fetch_object($rs)) {

        $arr[] = $obj;

    }

    // add the header line to specify that the content type is JSON
    header("Content-type: application/json");

    echo "{\"data\":" .json_encode($arr). "}";

    ?>

Now we can refresh the page and there will be data in the grid.

[![6][21]][22]

### More Complex Data

That’s all well and good, but that data is really simple.  Let’s make it a bit
more complex.  Each of these employees is in sales.  They all have territories
that they are responsible for.  Ideally, we need to be able to display 1 to
many territories per each employee.

With Kendo UI, you can define a detail template for each item in the grid.  In
other words, we are going to nest a grid within a grid.

**The Detail Template**

The detail template is very simple.  It’s just an empty **div**.  We could do
much more complex things here like adding in tab controls like we did in the
detail template example on the [demo’s page][23].


	<script type="text/x-kendo-template" id="template">

        <div class="subgrid"></div>

	</script>



For more information on Kendo UI Templates, refer to the [documentation][24]
and [examples][25].  Now that we have a template defined, we need to specify
in the grid that we do in fact want a detail template and a function to call
each time a row is expanded to check the details.

Our main grid function now looks like this.  The last two lines define the
template, and to call the **detailInit()** function when a grid row is
expanded.


    $("#grid").kendoGrid({
        dataSource: {
            transport: {
                read: "data/employees.php"
            },
            schema: {
               data: "data"
            }
        },
        columns: [{ field: "FirstName" }, { field: "LastName" }],
        detailTemplate: kendo.template($("#template").html()),
        detailInit: detailInit
    });


The **detailInit** function is fairly straightforward.


    function detailInit(e) {
        // get a reference to the current row being initialized 
		var detailRow = e.detailRow;

        // create a subgrid for the current detail row, getting territory data for this employee

        detailRow.find(".subgrid").kendoGrid({
            dataSource: {
                transport: {
                     read: "data/territories.php"
                },
                schema: {
                	data: "data"
                },

                serverFiltering: true,
                filter: { field: "EmployeeID", operator: "eq", value:e.data.EmployeeID }
           },

           columns: [{ title: "Territories", field: "TerritoryDescription" }],

        });
	}


It takes in an argument of **e** which will contain data about the row we
expanded in the grid, and the data that was in that row.

The **subgrid** has a class of **.subgrid**. The .**find()** is a jQuery
method to search the children of the current row and find the grid.  Again,
having a full template here is unnecessary since all we have is a simple
**div**, but it should give you a better idea of where you would start if you
wanted to have a robust details section.

The only other thing of note in the **subgrid** declaration is that we are
going to pass the EmployeeID on which to filter the territories by using
filtering.  Filtering will pass a filter object with the request to the
**data/territories.php** file.

### Create The Territories File

We need a new file for querying territories.  Make one exactly like the
**employees.php** file and call it **territories.php**.  You can copy the code
inside verbatim.  We do need to swap out the SQL.  It’s going to have to join
several tables together to get from the Territories table back to the
Employees Table.

We can get the **EmployeeID** off the **filter** object which Kendo UI sends
to the server as part of the request.  Filter objects are complex in their
structure because they can contain extensive filtering instructions.  The
filter we defined above will pass an object filter that is an array of
filters.  Each of these filters has a field to filter on, an operator (eq,
greaterthan, ect.) and a value.  We are only interested in the one value we
are passing and we know the operator is always equals.


       1:  <?php
       2:  // DISCLAIMER: It is better to use prepared statements in PHP.
           // This provides protection against sql injection.
           // [http://php.net/manual/en/pdo.prepared-statements.php][26]
       3:  // get the employee id off the request. escape it to protect against sql injection
       4:  $employeeID = mysql_real_escape_string($_REQUEST["filter"]["filters"][0]["value"]);
       5:
       6:  $link = mysql_pconnect("localhost", "root", "root") or die("Unable To Connect To Database Server");
       7:  mysql_select_db("northwind") or die("Unable To Connect To Northwind");
       8:
       9:  $arr = array();
      10:   $rs = mysql_query("SELECT TRIM(t.TerritoryDescription) AS TerritoryDescription
      11:                      FROM Territories t
      12:                      INNER JOIN EmployeeTerritories et ON t.TerritoryID = et.TerritoryID
      13:                      INNER JOIN Employees e ON et.EmployeeID = e.EmployeeID
      14:                      WHERE e.EmployeeID = " .$employeeID);
      15:
      16:  while($obj = mysql_fetch_object($rs)) {
      17:      $arr[] = $obj;
      18:  }
      19:
      20:  // add the header line to specify that the content type is JSON
      21:   header("Content-type: application/json");
      22:
      23:   echo "{\"data\":" .json_encode($arr). "}";
      24:
      25:  ?>


Line 4 pulls the filter data of the PHP **$_REQUEST** object.  This is not
necessarily the best way to read the parameter, but for the sake of simplicity
and the ability to see how the filter object is actually structured, I have
done it this way.

Now whenever a row is expanded, a query will be made to the**
territories.php** file which will provide the territories that are associated
with each employee.

[![7][27]][28]

And expanded…


[![8][29]][30]


### Wrap Up

We walked through a lot of content here, but let’s review.  We accomplished
the following things…

  1. Installed PHP, MySQL

  2. Created A PHP Application

  3. Wrote Endpoints For Returning Data

  4. Wired Up A Kendo UI Grid For Displaying Parent-Child Data

This has been an example of how to get rolling with PHP and [Kendo UI][32].

   [2]: http://en.wikipedia.org/wiki/Polyglot

   [3]: http://www.mamp.info/en/index.html

   [4]: http://framework.zend.com/

   [5]: http://www.zend.com/en/community/pdt

   [6]: http://www.microsoft.com/web/webmatrix/

   [7]: http://code.google.com/p/northwindextended/downloads/detail?name=North
wind.MySQL5.sql

   [8]: https://github.com/telerik/kendo-docs/blob/master/tutorials/images/php1_1.png (1)

   [9]: https://github.com/telerik/kendo-docs/blob/master/tutorials/images/php1_2.png

   [10]: http://www.kendoui.com/videos.aspx

   [11]: http://www.kendoui.com/documentation/ui-widgets/grid/walkthrough.aspx

   [12]: http://localhost:8888/KendoUI/data/employees.php

   [13]: https://github.com/telerik/kendo-docs/blob/master/tutorials/images/php1_3.png (2)

   [14]: https://github.com/telerik/kendo-docs/blob/master/tutorials/images/php1_4.png

   [15]: https://github.com/telerik/kendo-docs/blob/master/tutorials/images/php1_5.png (3)

   [16]: https://github.com/telerik/kendo-docs/blob/master/tutorials/images/php1_6.png

   [17]: https://github.com/telerik/kendo-docs/blob/master/tutorials/images/php1_7.png (4)

   [18]: https://github.com/telerik/kendo-docs/blob/master/tutorials/images/php1_8.png

   [19]: http://www.kendoui.com/Libraries/MetaBlogLib/Windows-Live-Writer-
ff819b019713_9A19-5_thumb_1.sflb.ashx (5)

   [20]: http://www.kendoui.com/Libraries/MetaBlogLib/Windows-Live-Writer-
ff819b019713_9A19-5_4.sflb.ashx

   [21]: http://www.kendoui.com/Libraries/MetaBlogLib/Windows-Live-Writer-
ff819b019713_9A19-6_thumb.sflb.ashx (6)

   [22]: http://www.kendoui.com/Libraries/MetaBlogLib/Windows-Live-Writer-
ff819b019713_9A19-6_2.sflb.ashx

   [23]: http://demos.kendoui.com/web/grid/detailtemplate.html

   [24]:
http://www.kendoui.com/documentation/framework/templates/overview.aspx

   [25]: http://demos.kendoui.com/web/templates/index.html

   [26]: http://php.net/manual/en/pdo.prepared-statements.php

   [27]: http://www.kendoui.com/Libraries/MetaBlogLib/Windows-Live-Writer-
ff819b019713_9A19-7_thumb_2.sflb.ashx (7)

   [28]: http://www.kendoui.com/Libraries/MetaBlogLib/Windows-Live-Writer-
ff819b019713_9A19-7_6.sflb.ashx

   [29]: http://www.kendoui.com/Libraries/MetaBlogLib/Windows-Live-Writer-
ff819b019713_9A19-8_thumb_1.sflb.ashx (8)

   [30]: http://www.kendoui.com/Libraries/MetaBlogLib/Windows-Live-Writer-
ff819b019713_9A19-8_4.sflb.ashx

   [31]: /Blogs_Libraries/Source_Code/KendoUI.sflb.ashx

   [32]: http://www.kendoui.com/get-kendo-ui.aspx
