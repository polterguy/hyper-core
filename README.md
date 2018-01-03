# A generic REST web service

Hyper Core is a back end for your web apps, built as a generic HTTP REST 
web service. This allows you to build your front end, towards a mature and secure back end, 
that automatically takes care of common security issues, such as SQL insertion attacks, and
authorisation and authentication - While at the same time getting to reuse pre-built generic 
operations, such as CRUD operations, towards your MySQL database(s). It is a fairly new project, 
and should be considered BETA. Hyper Core is built is 
a [Phosphorus Five](https://github.com/polterguy/phosphorusfive) module, and hence obviously 
depends upon Phosphorus Five to function.

## MySQL CRUD operations

There are four basic CRUD operations you can perform on your MySQL database(s). These are as 
follows.

* __[select]__ - Selects data from your MySQL database. Requires `GET` method.
* __[update]__ - Updates data in your database. Requires `POST` method.
* __[delete]__ - Deletes data from your database. Requires `DELETE` method.
* __[insert]__ - Inserts data into your database. Requires `PUT` method.

Since each of the above operations have their own unique URL, and each database
and table also gets its own unique virtual URL, this implies that you can use
the integrated authentication and authorisation features of Phosphorus Five,
through for instance its [Peeples](https://github.com/polterguy/peeples) module, 
which allows you to allow or deny access to URLs, according to your users' role.
Each of the above HTTP REST services follows the following URL format. 

```
/hyper-core/database/[database]/[table]/[operation]
```

If you have a database called e.g. `camphora`, with a table called `customers`, and you want to
perform a `select` query towards it for instance, you can accomplish that with the following URL.

```
/*
 * Optionally add QUERY parameters to create more complex SELECT queries.
 */
/hyper-core/database/camphora/customers/select
```

### Authorisation objects

Hyper Core builds on top of the extendible [p5.auth project](https://github.com/polterguy/phosphorusfive/tree/master/plugins/extras/p5.auth),
to grant or deny access to database operations, for specific databases and tables, according to a user's role,
and your access control objects. To allow for instance all users, including _"guest"_ visitors, to for instance 
evaluate `select` operations towards your above `camphora` database, and its `customers` table, but deny everybody 
except your _"developer"_ users to perform all other operations on the same database and table - You could create 
an authorisation object that looks like the following.

```
*
  p5.module.allow:/modules/hyper-core
*
  p5.hyper-core.allow:/hyper-core/database/camphora/customers/select
developer
  p5.hyper-core.allow:/hyper-core/database/camphora/customers
```

The first record above, gives all users access to the module in general, which is necessary
to have the core URL resolver in Phosphorus Five to even resolve the URL,
and pass the request control onwards to the _"hyper-core"_ module. The second record, grants 
all users access to do `select` operations on the `camphora` database, but only its `customers` table.
The third record above, gives your `developer` users access to all operations on the
`camphora` database, but only its `customers` table.

All in all, the above three access control objects, gives your developer users complete control
over the `camphora.customers` database, while random visitors have only `select` rights on the
same database.

The default access control, unless explicitly overridden in your access control object, is 
to **deny all operations on all databases and all tables**. So you'll need to explicitly grant
access for a role to be able to have that role do anything towards any database in your system.

### Select operation

The select operation takes the following optional parameters as HTTP GET parameters.

* __[columns]__ - Which columns you want to select, defaults to "\*" (all columns).
* __[order-by]__ - Which column you want to order your select query by. No default value.
* __[order-dir]__ - Can be either _'asc'_ or _'desc'_, and declares whether or not you'd like to order ascending or descending. Defaults to _'asc'_.
* __[offset]__ - Offset of where to start fetching items. Defaults to `0`.
* __[limit]__ - Number of items to return. Defaults to `10`. Notice, for security reasons, it will throw an exception if you try to select more than 100 items.
* __xxx__ - Becomes additional parts of your `where` clause.

All parameters above are optional, and will be given _"sane defaults"_ if omitted. If you want to 
select only name and email columns, and sort descending by name, from your `camphora` database, and 
its `customers` table, you can accomplish that with the following URL.

```
/hyper-core/database/camphora/customers/select?[columns]=name,email&[sort-by]=name&[sort-dir]=desc
```

The above will return the first 10 records, but only the name and email columns, and sort your results descending by
their `name`column's value. All additional parameters becomes a part of the where clause to your SQL, and must all 
contain three components, separated by `:`. Below are the explanation for these components.

* __type__ - Type of column. Can be `string`, `date`, `bool`, etc.
* __operand__ - Operand to use for your clause. Can be `like`, `=`, `>=`, etc.
* __value__ - Value to compare against.

If you'd like to retrieve only items which have a `name` containing the string `%hansen%` for instance, 
you could accomplish that with the following URL.

```
/hyper-core/database/camphora/customers/select?name=string:like:%hansen%
```

**Important** - Remember to URL encode your URL!


