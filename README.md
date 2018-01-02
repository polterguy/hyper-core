# A generic REST web service

Hyper Core is a generic back end for your web apps, built as a generic REST HTTP 
web service, allowing you to build your front end, towards a mature and secure back end, 
automatically taking care of common security issues, such as SQL insertion attacks - While
at the same time getting to reuse pre-built generic operations, such as CRUD operations,
towards your MySQL database(s). It is a fairly new project, and should be considered BETA, 
but contains the following main modules at the time being. Hyper Core is built is
a [Phosphorus Five](https://github.com/polterguy/phosphorusfive) module, and hence obviously
dependent upon Phosphorus Five to function.

## MySQL CRUD operations

There are four basic CRUD operations you can perform on your MySQL database. These are as 
follows.

* __[select]__ - Selects data from your MySQL database. Requires `GET` method.
* __[update]__ - Updates data in your database. Requires `POST` method.
* __[delete]__ - Deletes data from your database. Requires `DELETE` method.
* __[insert]__ - Inserts data into your database. Requires `PUT` method.

Since each of the above operations have their own unique URL, you can use
the integrated authentication and authorisation features of [Phosphorus Five](https://github.com/polterguy/phosphorusfive),
through for instance its [Peeples](https://github.com/polterguy/peeples) module, 
which allows you to allow or deny access to URLs, according to your users' role.
Each of the above HTTP REST services follows the following URL format. 

```
/hyper-core/database/[database]/[table]/[operation]
```

If you have a database called e.g. `my-cool-database`, with a table called `customers`, and you want to
perform a `select` query towards it for instance, you can accomplish that with the following URL.

```
/hyper-core/database/my-cool-database/customers/select
```

Assuming you have not changed the default installation folder of your Hyper Core module.
Each of the four above operations, can take additional parameters, depending upon which operation
you want to perform.

### Select operation

The select operation takes the following optional parameters as HTTP GET parameters.

* __[columns]__ - Which columns you want to select, defaults to "\*" (all columns).
* __[order-by]__ - Which column you want to order your select query by.
* __[order-dir]__ - Can be either 'asc' or 'desc', and declares whether or not you'd like to order ascending or descending.
* __[offset]__ - Offset of where to start fetching items. Defaults to 0.
* __[limit]__ - Number of items to return. Defaults to 10.

All parameters are optional. If you want to select only name and email columns, and sort descending by name,
from your _"my-cool-database"_ and its _"customers"_ table, you can accomplish that with the following URL.

```
/hyper-core/database/my-cool-database/customers/select?[columns]=name,email&[sort-by]=name&[sort-dir]=desc
```

The above will return the first 10 records, but only the name and email columns.

All additional parameters becomes a part of the where clause to your SQL, and must all contain three components.

* __type__ - Type of column.
* __operand__ - Operand to use for your clause.
* __value__ - Value to compare against.

If you'd like to retrieve only items which have a name containing the string `%hansen%` for instance, you can accomplish
that with the following URL `/hyper-core/database/my-cool-database/customers/select?name=string:like:%hansen%`. Remember
to URL encode your URL though.


