## A generic back end for JSON/REST based front ends

Hyper Core is an attempt at creating a generic back end, built around JSON, allowing you to easily consume
pre-built JSON based web services, from your own JavaScript projects. It is recently
initiated, but contains for now, a _"database"_ module, which allows you to perform
all basic CRUD operations, on any MySQL based database, as long as it has at the very least
an _"id"_ primary key column.

The idea with the project, is to create a generic back end, which will serve most
of your purposes, by exposing parts of Phosphorus Five to JSON web services, such
that you can focus exclusively on the front end, integrating Phosphorus Five with
any front end framework you wish. This would allow you to create your front end in 
anything you wish, such as for instance Sencha, Angular, React, jQuery, etc. While
having Phosphorus Five automatically take care of things such as security, etc.

Since the module is built around URLs, and you can grant or deny access to URLs
in Phosphorus Five, according to role(s) - This allows you to easily control who are 
allowed to do what in the back end, using the integrated authorisation and authentication
features of Phosphorus Five to control this.

To see an example of how to use it, on a development machine, you can access the 
url `http://127.0.0.1:8080/modules/hyper-core/samples/minimalistic-crud-example.html`, if you
are on a localhost development machine for instance. This is an example
of how to perform all CRUD operations on a database. Notice, this example requires
that you have MySQL setup and configured correctly in your web.config, in addition
to that you actually have some database, and some table within it. The example expects
a database called _"p5_camphora"_, with a table named _"foo"_, and a column named _"name"_ in 
your _"foo"_ table - In addition to an AUTO_INCREMENT column called _'id'_, which is the table's
primary id, preferably also with some records in your _"foo"_ table - But this can easily 
be changed. The easiest way to create such a table, is to use Camphora Five, and create a
CRUD app named _"foo"_, with a _"name"_ field within it.

Notice, the database needs to be prefixed with your database prefix, which is defined in
your web.config, and may vary from installation to installation. This is the part that
says _"p5\_"_ above, in the database name.

