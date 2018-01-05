# A MySQL HTTP REST based ORM library (and more)

Hyper Core is a MySQL ORM library, built as a generic HTTP REST web service.
This allows you to build your front end, towards a mature, reusable, and secure back end, 
that automatically takes care of common security issues, such as SQL insertion attacks, and
authorisation and authentication. Hyper Core is a 
a [Phosphorus Five](https://github.com/polterguy/phosphorusfive) module, and hence requires
you to use Phosphorus Five as your core.

## MySQL CRUD operations

There are four basic CRUD operations you can perform on your MySQL database(s). These are as 
follows.

* __[select]__ - Selects data from your MySQL database. Requires `GET` method.
* __[update]__ - Updates data in your database. Requires `POST` method.
* __[delete]__ - Deletes data from your database. Requires `DELETE` method.
* __[insert]__ - Inserts data into your database. Requires `PUT` method.

Each of the above HTTP REST services follows the following URL format. 

```
/hyper-core/mysql/[database]/[table]/[operation]
```

If you have a database called e.g. `todo`, with a table called `items`, and you want to
perform a `select` query towards it for instance, you can use the following URL.

```
/*
 * Optionally add QUERY parameters to create more complex SELECT queries.
 */
/hyper-core/mysql/todo/items/select
```

### Select operation

The select operation takes the following optional parameters as HTTP query parameters. Notice,
all _"special parameter types"_, such as columns, requires square brackets surrounding their names.
This is to prevent these parameters from _"clashing"_ with your generic column
_"where"_ declarations.

* __[boolean]__ - Which boolean operator to use for multiple filter criteria. Defaults to `OR`.
* __[columns]__ - Which columns you want to select, defaults to `*` (all columns).
* __[order-by]__ - Which column you want to order your select query by. No default value.
* __[order-dir]__ - Can be either `asc` or `desc`, and declares whether or not you'd like to order ascending or descending. Defaults to `asc`.
* __[offset]__ - Offset of where to start fetching items. Defaults to `0`.
* __[limit]__ - Number of items to return. Defaults to `10`. Notice, to avoid having buggy front end code exhaust the server and bandwidth resources, it will throw an exception if you try to select more than 250 items.
* __xxx__ - Becomes additional parts of your `where` clause. Basically _"anything else"_. **Notice** - These additional arguments are `AND`'ed together by default.

All parameters above are optional, and will be given _"sane defaults"_ if omitted. If you want to 
select only the `description` and `id` columns, and sort descending by `description`, from your `todo` database, and 
its `items` table - You can accomplish that with the following URL.

```
/hyper-core/mysql/todo/items/select?[columns]=description,id&[order-by]=description&[order-dir]=desc
```

The above will return the first 10 records, but only the `description` and `id` columns, and sort your results descending by
their `description` column's value. All additional parameters becomes a part of the where clause to your SQL, and must all 
contain two components, separated by `:`. Below are the explanation for these components.

* __operand__ - Operand to use for your clause. Can be `like`, `equal`, `not-equal`, `less-than`, `more-than`, `less-than-equal`, or `more-than-equal`.
* __value__ - Value to compare against.

If you'd like to retrieve only items which have a `description` containing the string `%groceries%` for instance, 
you could accomplish that with the following URL.

```
/hyper-core/mysql/todo/items/select?description=like:%groceries%
```

**Important** - Remember to URL encode your URL!

Notice, if you supply multiple additional query parameters, these will be `OR`'ed together in your select SQL.
This means that the following would select all items having either the firstname containing _"john"_, or the surname 
containing _"hansen"_. The `select` operation requires you to use a GET HTTP request.

```
/hyper-core/mysql/database/table/select?firstname=like:%john%&surname=like:%hansen%
```

If you wish to use the `and` boolean operator instead, you can explicitly override the boolean operator your MySQL
queries are constructed with, by adding a `[boolean]` argument, and set its value to `and`. For the above query, this
would resemble the following.

```
/hyper-core/mysql/database/table/select?firstname=like:%john%&surname=like:%hansen%&[boolean]=and
```

Tha above URL would fetch only items containing both the text _"john"_ in its firstname, and the value of _"hansen"_
in its surname.

### Insert operation

The `insert` operation is arguably simpler than the `select` operation, since it doesn't require as many arguments. It requires
you to use a `PUT` HTTP request though, and expects each column to be declared as a value/key pair in your body's request.
For instance, assuming you're in JavaScript land for instance, you could insert a column into your database with
the following code.

```
var xhr = new XMLHttpRequest();
xhr.open('PUT', '/hyper-core/mysql/camphora/customers/insert', true);
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

### Update operation

The `update` operation requires you to use a `POST` HTTP request. It requires one query parameter declaring
which item you'de like to update, and all of its new values as part of the body of your request. Assuming
you're in JavaScript land, you could update an item with something resembling the following.

```
var id = 5;
var name = 'John Doe';
var xhr = new XMLHttpRequest();
xhr.open('POST', '/hyper-core/mysql/camphora/customers/update?id=' + id, true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    eval("window.res = " + xhr.responseText);
    console.log('Item was successfully updated, affected records was; ' + window.res.affected_records);
    get_items ();
  }
};
var body = 'name=' + name;
xhr.send(body);
```

The above will update the `name` value for your `camphora.customers` database item with the id of 5, 
and set its new value to _"John Doe"_.

### Delete operation

The `delete` operation requires you to use a `DELETE` HTTP method. It requires a single query parameter,
declaring which column and value you wish to use as your `where` clause in your final SQL. To delete an item
in our above `camphora.customers` database, having the id of 5 for instance, could be accomplished with the following
code, assuming you're in JavaScript land.

```
var xhr = new XMLHttpRequest();
xhr.open('DELETE', '/hyper-core/mysql/camphora/customers/delete?id=5', true);
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    eval("window.res = " + xhr.responseText);
    alert(window.res.affected + ' items was successfully deleted');
  }
};
xhr.send();
```

## Authentication

Authentication is done by issuing a `POST` HTTP request towards the login module. If you're in JavaScript
land for instance, you can login with something resembling the following.

```
var xhr = new XMLHttpRequest();
xhr.open('POST', '/hyper-core/login', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    eval("window.res = " + xhr.responseText);
    alert('Result of login operation was; ' + window.res.result);
  }
};
var username = 'your_username';
var password = 'your_password';
xhr.send('username=' + encodeURIComponent (username) + '&password=' + encodeURIComponent (password);
```

If you wish to logout, you can issue a `DELETE` HTTP request towards its _"logout"_ counterpart. Example
can be found beneath.

```
var xhr = new XMLHttpRequest();
xhr.open('DELETE', '/hyper-core/logout', true);
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    eval("window.res = " + xhr.responseText);
    alert('Status; ' + window.res.status);
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
  p5.hyper-core.allow:/hyper-core/mysql/camphora/customers/select
developer
  p5.hyper-core.allow:/hyper-core/mysql/camphora/customers
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

## Installation

First you'll have to [install Phosphorus Five](https://github.com/polterguy/phosphorusfive/releases), and then
clone or download Hyper Core, and make sure you put it inside of your Phosphorus Five folder structure, 
at _"/core/p5.webapp/modules/"_.

### Creating an example database

To create an example database, you can install [Camphora Five](https://github.com/polterguy/camphora-five),
which is actually most easily installed through the _"Bazar"_ in Phosphorus Five. You can also use
existing MySQL databases. However, if you choose the latter, you must **edit your web.config**, and make sure you
set the `p5.data.prefix` setting to an empty string. The example application, which can test out at
for instance `http://127.0.0.1:8080/modules/hyper-core/samples/minimalistic-crud-example.html`, if you're
on a locahost machine, expects a database called `camphora`, and a table called `foo`. If you use
Camphora Five to create your test database, you'll need to create a Camphora Five app and name it `foo`.

Alternatively, you can also of course edit the example code, to have it use another database,
using for instance [Hyper IDE](https://github.com/polterguy/hyper-ide), which is bundled together with the main 
download of Phosphorus Five.

## Warning

This is **ALPHA** stuff, and **NOT** ready for production.

If you wish, you can see a video of me demonstrating the system [here](https://www.youtube.com/watch?v=tWAZUAKVBeg).

