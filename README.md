# A generic REST web service

Hyper Core is a back end for your web apps, built as a generic HTTP REST 
web service. This allows you to build your front end, towards a mature, reusable, and secure back end, 
that automatically takes care of common security issues, such as SQL insertion attacks, and
authorisation and authentication - While at the same time getting to reuse pre-built generic 
operations, such as CRUD operations, towards your MySQL database(s). It is a fairly new project, 
and should be considered **ALPHA**. Hyper Core is built is 
a [Phosphorus Five](https://github.com/polterguy/phosphorusfive) module, and hence obviously 
depends upon Phosphorus Five to function.

The idea if the project, is to provide a generic and extendible REST back end, arguably solving
_"all"_ your needs for a back end, allowing you to use any front end you wish. However, at the
time being, the only _"modules"_ it contains, is a MySQL REST service, allowing you to perform
all basic CRUD operations on your MySQL database(s) - In addition to an authentication module,
allowing you to login and authenticate you towards the back end.

## MySQL CRUD operations

There are four basic CRUD operations you can perform on your MySQL database(s). These are as 
follows.

* __[select]__ - Selects data from your MySQL database. Requires `GET` method.
* __[update]__ - Updates data in your database. Requires `POST` method.
* __[delete]__ - Deletes data from your database. Requires `DELETE` method.
* __[insert]__ - Inserts data into your database. Requires `PUT` method.

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

### Select operation

The select operation takes the following optional parameters as HTTP query parameters. Notice,
all _"special parameter types"_, such as columns, requires square brackets surrounding their names.
This is a conscious choice, to eliminate that these parameters are _"clashing"_ with your generic column
_"where"_ declarations.

* __[columns]__ - Which columns you want to select, defaults to "\*" (all columns).
* __[order-by]__ - Which column you want to order your select query by. No default value.
* __[order-dir]__ - Can be either _'asc'_ or _'desc'_, and declares whether or not you'd like to order ascending or descending. Defaults to _'asc'_.
* __[offset]__ - Offset of where to start fetching items. Defaults to `0`.
* __[limit]__ - Number of items to return. Defaults to `10`. Notice, to avoid having buggy front end code exhaust the server and bandwidth resources, it will throw an exception if you try to select more than 1,000 items.
* __xxx__ - Becomes additional parts of your `where` clause. Basically _"anything else"_. **Notice** - These additional arguments are `OR`'ed together.

All parameters above are optional, and will be given _"sane defaults"_ if omitted. If you want to 
select only name and id columns, and sort descending by name, from your `camphora` database, and 
its `customers` table - You can accomplish that with the following URL.

```
/hyper-core/database/camphora/customers/select?[columns]=name,id&[order-by]=name&[order-dir]=desc
```

The above will return the first 10 records, but only the name and email columns, and sort your results descending by
their `name`column's value. All additional parameters becomes a part of the where clause to your SQL, and must all 
contain two components, separated by `:`. Below are the explanation for these components.

* __operand__ - Operand to use for your clause. Can be `like`, `=`, `>=`, etc.
* __value__ - Value to compare against.

If you'd like to retrieve only items which have a `name` containing the string `%hansen%` for instance, 
you could accomplish that with the following URL.

```
/hyper-core/database/camphora/customers/select?name=like:%hansen%
```

**Important** - Remember to URL encode your URL!

Notice, if you supply multiple additional query parameters, these will be `OR`'ed together in your select SQL.
This means that the following would select all items having either the firstname containing _"john"_, or the surname 
containing _"hansen"_. The `select` operation requires you to use a GET HTTP request.

```
/hyper-core/database/camphora/customers/select?firstname=like:%john%&surname=like:%hansen%
```

## Insert operation

The `insert` operation is arguably simpler than the `select` operation, since it doesn't require as many arguments. It requires
you to use a `PUT` HTTP request though, and expects each column to be declared as a value/key pair in your body's request.
For instance, assuming you're in JavaScript land for instance, you could insert a column into your database with
the following code.

```
var xhr = new XMLHttpRequest();
xhr.open('PUT', '/hyper-core/database/camphora/customers/insert', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    eval("window.res = " + xhr.responseText);
    alert('Item was successfully inserted with the id of; ' + window.res.id);
  }
};
var name = 'John Doe';
var email = 'john@doe.com';

// Notice, assumes you have a name and email column, in your customer table, in your camphora database.
xhr.send(body = 'name=' + encodeURIComponent (name) + '&email=' + encodeURIComponent (email);
```

The `insert` operation will return the id of your inserted item, whatever that happens to be, or whatever the
name of your id column happens to be.

## Delete operation

The `delete` operation requires you to use a `DELETE` HTTP method. It requires a single query parameter,
declaring which column and value you wish to use as your `where` clause in your final SQL. To delete an item
in our above `camphora.customers` database, having the id of 5 for instance, could be accomplished with the following
code, assuming you're in JavaScript land.

```
var xhr = new XMLHttpRequest();
xhr.open('PUT', '/hyper-core/database/camphora/customers/delete?id=5', true);
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    eval("window.res = " + xhr.responseText);
    alert(window.res.affected + ' items was successfully deleted');
  }
};
xhr.send();
```

## Authorisation objects

Hyper Core builds on top of the extendible [p5.auth project](https://github.com/polterguy/phosphorusfive/tree/master/plugins/extras/p5.auth).
Each operation, database, and table within your database, get its own unique URL.
Since Phosphorus Five allows you to grant or deny access to URLs according to which role your currently
logged in user belongs to - This gives you a highly fine grained control over who are allowed to do what
in regards to your SQL operations. To allow for instance all users, including _"guest"_ visitors, to for instance 
evaluate `select` operations towards your above `camphora` database, and its `customers` table - But deny everybody 
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

