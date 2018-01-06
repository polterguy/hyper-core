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

In addition to the above four standard methods, there is also an `x` method, which allows
you to securely create any type of SQL you wish. Check out the documentation for the `x` method
further down in this document. Each of the above HTTP REST services follows the following URL format.

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
_"where"_ declarations. The `select` operation requires you to use a GET HTTP request.

* __[columns]__ - Which columns you want to select, defaults to `*` (all columns).
* __[order-by]__ - Which column you want to order your select query by. No default value.
* __[order-dir]__ - Can be either `asc` or `desc`, and declares whether or not you'd like to order ascending or descending. Defaults to `asc`.
* __[offset]__ - Offset of where to start fetching items. Defaults to `0`.
* __[limit]__ - Number of items to return. Defaults to `10`. Notice, to avoid having buggy front end code exhaust the server and bandwidth resources, it will throw an exception if you try to select more than 250 items.
* __[boolean]__ - Which boolean operator to use for multiple filter criteria. Defaults to `OR`.
* __xxx__ - Becomes additional parts of your `where` clause. Basically _"anything else"_. **Notice** - These additional arguments are `OR`'ed together by default. Change the boolean operator by changing the `[boolean]` argument.

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

Notice, if you supply multiple additional query parameters, these will be `OR`'ed together by default in your select SQL.
This means that the following would select all items having either the firstname containing _"john"_, or the surname 
containing _"hansen"_.

```
/hyper-core/mysql/database/table/select?firstname=like:%john%&surname=like:%hansen%
```

If you wish to use the `and` boolean operator instead, you can explicitly override the boolean operator your MySQL
queries are constructed with, by adding a `[boolean]` argument, and set its value to `and`. For the above query, this
would resemble the following.

```
/hyper-core/mysql/database/table/select?firstname=like:%john%&surname=like:%hansen%&[boolean]=and
```

The above URL would fetch only items containing both the text _"john"_ in its firstname, and the value of _"hansen"_
in its surname. More complex queries, such as join operations, and richer SQL queries, can be constructed using
the extensibility features of the MySQL module. Which we will have a look at later.

### Insert operation

The `insert` operation is arguably simpler than the `select` operation, since it doesn't require as many arguments. It requires
you to use a `PUT` HTTP request, and expects each column to be declared as a value/key pair in your body's request.
For instance, assuming you're in JavaScript land for instance, you could insert a column into your database with
the following code.

```
var xhr = new XMLHttpRequest();
xhr.open('PUT', '/hyper-core/mysql/database/table/insert', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    eval("window.res = " + xhr.responseText);
    alert('Item was successfully inserted with the id of; ' + window.res.id);
  }
};
var name = 'John Doe';
var email = 'john@doe.com';

// Notice, assumes you have a name and email column, in your table table, in your database database.
xhr.send(body = 'name=' + encodeURIComponent (name) + '&email=' + encodeURIComponent (email);
```

The `insert` operation will return the id of your inserted item as `id`.

### Update operation

The `update` operation requires you to use a `POST` HTTP request. It requires one query parameter declaring
which item you'de like to update, and all of its new values as part of the body of your request. Assuming
you're in JavaScript land, you could update an item with something resembling the following.

```
var id = 5;
var name = 'John Doe';
var xhr = new XMLHttpRequest();
xhr.open('POST', '/hyper-core/mysql/database/table/update?id=' + id, true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    eval("window.res = " + xhr.responseText);
    alert('Item was successfully updated, affected records was; ' + window.res.affected_records);
  }
};
var body = 'name=' + name;
xhr.send(body);
```

The above will update the `name` value for your `database.table` database item having the id of 5, 
and set its new value to _"John Doe"_. **Notice**, the `update` method can only be given one HTTP query
parameter as its condition. To construct more complex update statements, you must use the extensibility
features, which we will discuss further down in this document.

### Delete operation

The `delete` operation requires you to use a `DELETE` HTTP method. It requires a single query parameter,
declaring which column and value you wish to use as your `where` clause in your final SQL. To delete an item
in our above `database.table` database, having the id of 5 for instance, could be accomplished with the following
code, assuming you're in JavaScript land.

```
var xhr = new XMLHttpRequest();
xhr.open('DELETE', '/hyper-core/mysql/database/table/delete?id=5', true);
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    eval("window.res = " + xhr.responseText);
    alert(window.res.affected_records + ' items was successfully deleted');
  }
};
xhr.send();
```

### Creating your own extension methods

Often times the above 4 basic CRUD operations simply won't cut it for you. Maybe you want to perform a join on
multiple tables for instance? Or you want to add more complex conditions than the above allows you to? For such 
times, there is the `x` method. The `x` method doesn't take a table name as its second parameter. Instead it requires
the name, to your extension method, which you'll need to supply as a Hyperlambda file, inside
of your _"/common/documents/private/hyper-core/mysql/x/"_ folder. If you have a file called for instance
_"foo.hl"_ inside of the previously mentioned folder, you can invoke this extension method using a URL resembling
the following.

```
/hyper-core/mysql/todo/foo/x
```

The above URL will evaluate your _"/common/documents/private/hyper-core/mysql/x/foo.hl"_ file.
This file will be evaluated with all the relevant information from your HTTP request, having been automatically
decorated for you. Notice, as your file is evaluated, the `todo` database will already be the open and active
database connection, but no transactions will have been associated with it. Below is an example of how your 
file invocation might be decorated.

```
method:POST
params
  some-url-encoded-parameter:foo
query
  some-query-parameter:bar
```

This allows you to easily reference any parts of the HTTP request, and its decoration, as you see fit.
Below is an example of a file that executes a `count` SQL on your `items` table,
parametrised with a query parameter, filtering the `description` of the records it should count.

```
p5.mysql.scalar:select count(*) from items where description like @description
  @description:%{0}%
    :x:/../*/query/*/description?value
return:x:/@p5.mysql.scalar?value
```

If you invoke the extension method above, with something resembling the following 
URL `/hyper-core/mysql/todo/foo/x?description=bar` - The above 
will return JSON resembling the following to your client, assuming you have two records containing the
string _"bar"_ in their description.

```json
{"result":2}
```

To return multiple records instead, you could create a _"foo.hl"_ file such as the following instead.

```
p5.mysql.select:select * from items where description like @description
  @description:%{0}%
    :x:/../*/query/*/description?value
return:x:/@p5.mysql.select/*
```

The above would return JSON resembling the following to your client.

```json
[{"id":2,"description":"bar"},{"id":3,"description":"hello"},
{"id":4,"description":"foo 2"},{"id":6,"description":"bar 3"},
{"id":7,"description":"bar 4"},{"id":9,"description":"XYS"}]
```

If you create an extension method that does an `insert`, `update`, or `delete` operation, it is considered
to be good practice to return the results of the operation to the client. For `update` and `delete`, this
would imply returning simply the value of the **[p5.mysql.delete]** or **[p5.mysql.update]** invocation.
For a **[p5.mysql.insert]** invocation, the id of the last inserted item, can be found as an **[id]** node
child of your insert event invocation.

#### Mandatory arguments

You can create an extension method that requires one or more arguments to be supplied, and if they aren't, 
it will throw an exception. To do this, you'll have to invoke the **[micro.lambda.contract.min]** event, 
passing in the root node, and a list of mandatory arguments, and their type(s), in your extension method implementation.
Below is an example of a method that requires the client to having pass in a `description` query argument as a string, 
and if he hasn't, or the description is empty, it will throw an exception.

```
/*
 * This line will throw an exception, if the client did not pass
 * in a "description" query parameter, or the "description" query parameter
 * was empty for some reasons.
 */
micro.lambda.contract.min:x:/..
  query
    description:string

/*
 * If we came this far without an exception, there was a "description" query parameter, 
 * and it was not empty.
 */
p5.mysql.select:select * from items where description like @description
  @description:%{0}%
    :x:/../*/query/*/description?value
return:x:/@p5.mysql.select/*
```

You can of course require an argument to be supplied that is convertible into for instance a `double` or
an `int` too. If you wish to do that, simply replace the above `description:string` with e.g. `foo:int`, or
whatever name/type you want your argument to be possible to convert into.

#### About Hyperlambda

The above extension methods are actually created using Hyperlambda. Although for most practical concerns, you
don't really have to worry about this, since for the most parts you will be creating very simple extension methods,
simply invoking a MySQL active event - Hyperlambda is actually a fully fledged programming language, which
is Turing complete, and allows you to do much more advanced stuff, than what we are doing above.

For most practical concerns, this won't be an issue, since you'll probably simply invoke a couple of MySQL
Active Events in your code, and return its results - Realise that you can still do some pretty amazing stuff
with Hyperlambda, using more of its feature set.

Let's create a thought experiment, imagining that we would for instance like to translate the description of 
our items into Norwegian, before we return them to the client. This is in fact easily accomplished using Hyperlambda, 
by invoking an Active Event from Micro, which allows us to invoke Google Translate. Below is an example of doing 
just that.

```
/*
 * Running our SQL select query.
 */
p5.mysql.select:select * from items where description like @description
  @description:%{0}%
    :x:/../*/query/*/description?value

/*
 * Iterating through each item, and invoking Google Translate for its "description",
 * updating the above item with the results returned from Google Translate.
 *
 * WARNING, this will take a LOT of time, if you have a huge result set
 * returned from your above SQL.
 * It is simply an **EXAMPLE** of how to use Hyperlambda, and not something you should
 * use in production code, due to that it creates one HTTP request, towards Google Translate,
 * for each item returned from our above SQL.
 */
for-each:x:/@p5.mysql.select/*
  micro.google.translate:x:/@_dp/#/*/description?value
    dest-lang:nb-NO
  set:x:/@_dp/#/*/description?value
    src:x:/@micro.google.translate?value

/*
 * Returning our items, now with the description in Norwegian instead of English.
 */
return:x:/@p5.mysql.select/*
```

The above will actually translate all your descriptions into Norwegian, before returning them to the client.
Obviously, if you want to implement something like the above, you should definitely cache the results somehow 
from Google Translate, to avoid multiple roundtrips for the same items again. However, to keep our example
small and simple, we didn't implement that in our above example.

#### Creating C#/VB.NET/F# extension methods

You can also create extension methods as CLR methods if you want to. This is easily accomplished by creating 
a new library type of .Net/Mono project, and add a reference to the project called _"p5.core"_, for then
to create a static C# method, in one of your classes, and mark it with the attribute `ActiveEvent`.
Below is an example of some C# code that would do just that for you.

```csharp
using System;
using p5.core;

class Foo
{
    [ActiveEvent (Name = "foo-bar.my-event")]
    public static void foo_bar_my_event (ApplicationContext context, ActiveEventArgs e)
    {
        e.Args.Value = 42;
    }
}
```

Then you'll need to reference the assembly containing the above class into your main _"p5.webapp"_ project,
for then to edit your _"web.config"_ file, to include your assembly as a plugin, by adding another `add` entry
into your `assemblies` configuration. Below is an example of how to accomplish this, assuming your assembly
is called `foo`.

```xml
<add assembly="foo" />
```

If you do the above, you can actually invoke your C# method from an extension method, by for instance changing
our above _"foo.hl"_ file to something resembling the following.

```
foo-bar.my-event
return:x:/@foo-bar.my-event?value
```

For the record, assuming you modified our above _"foo.hl"_ file, the URL to invoke your C# method would become
the following `/hyper-core/mysql/todo/foo/x`, and it would return the following JSON.

```json
{"result":42}
```

You can of course mix all concepts discussed here under the extension methods as you see fit, and have any amount
of Hyperlambda, SQL, and/or C# perfectly mixed together, as you see fit, creating any types of extension methods
you want to create.

## Authentication

There are also authentication end points, to login and logout a user, towards the 
[p5.auth](https://github.com/polterguy/phosphorusfive/tree/master/plugins/extras/p5.auth) authentication/authorisation
module in Phosphorus Five. This allows you to easily authenticate your users towards
the user and role based auth system in Phosphorus Five. The main URL for this module is `auth`.

### Logging in

Authentication is done by issuing a `POST` HTTP `login` request towards the `auth` module. If you're in JavaScript 
land for instance, you can login with something resembling the following.

```
var xhr = new XMLHttpRequest();
xhr.open('POST', '/hyper-core/auth/login', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    eval("window.res = " + xhr.responseText);
    alert('You belong to the role; ' + window.res.role);
  }
};
var username = encodeURIComponent ('your_username');
var password = encodeURIComponent ('your_password');
var persist = 'true';
xhr.send('username=' + username + '&password=' + password + '&persist=' + persist;
```

If you set the `persist` query parameter to true, a persistent cookie will be created on the client, making sure
the client will be remembered according to the persistent authorisation cookie rules of Phosphorus Five. The default
value for persisting a user's credential cookie using persistent logins are `false`, implying that once the user closes
his browser, or ends his session somehow, he will have to login again. The persistent authentication cookie
rule of Phosphorus Five can be edited in your _"web.config"_ file, by changing the `p5.auth.credential-cookie-valid` key's
value.

### Logging out

If you wish to logout, you can issue a `DELETE` HTTP request towards its _"logout"_ counterpart. This will destroy
the authentication cookie on the client. An example can be found beneath.

```
var xhr = new XMLHttpRequest();
xhr.open('DELETE', '/hyper-core/auth/logout', true);
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    eval("window.res = " + xhr.responseText);
    alert('Status; ' + window.res.status);
  }
};
xhr.send();
```

### whoami, figuring out you're status

Since the default authentication of Phosphorus Five allows a client to persist his logged in status in a cookie on disc,
it is sometimes necessary to query towards the back end, to find out the authentication status of the client. This
can be done by issuing an HTTP `GET` request towards the `whoami` module. This will return the username and the role
of the client. An example can be found below.

```
/hyper-core/auth/whoami
```

**Notice**, the authentication cookie is not accessible on the client, since this would open up for a whole range
of different types of attacks, such as _"session highjacking"_, etc. Hence the only way to get information from the
client about the status of your client, is by issuing a `whoami` HTTP request towards the server.

### Authorisation objects

Hyper Core builds on top of the extendible [p5.auth project](https://github.com/polterguy/phosphorusfive/tree/master/plugins/extras/p5.auth).
Each operation, database, and table within your database, get its own unique URL.
Since Phosphorus Five allows you to grant or deny access to URLs according to which role your currently
logged in user belongs to - This gives you a highly fine grained control over who are allowed to do what
in regards to your SQL operations. To allow for instance all users, including _"guest"_ visitors, to for instance 
evaluate `select` operations towards your above `todo` database, and its `items` table - But deny everybody 
except your _"developer"_ users to perform all other operations on the same database and table - You could create 
an authorisation object that looks like the following.

```
*
  p5.module.allow:/modules/hyper-core
*
  p5.hyper-core.allow:/hyper-core/mysql/todo/items/select
developer
  p5.hyper-core.allow:/hyper-core/mysql/todo/items
```

The first record above, gives all users access to the module in general, which is necessary
to have the core URL resolver in Phosphorus Five even resolve the URL,
and pass the request control onwards to the _"hyper-core"_ module. The second record, grants 
all users access to do `select` operations on the `todo` database, but only its `items` table.
The third record above, gives your `developer` users access to all operations on the
`todo` database, but only its `items` table.

All in all, the above three access control objects, gives your developer users complete control
over the `todo.items` database table, while random visitors have only `select` rights on the
same database table.

The default access control, unless explicitly overridden in your access control object, is 
to **deny all operations on all databases and all tables**. So you'll need to explicitly grant
access for a role to be able to have that role do anything towards any database in your system.

## Installation

Hyper Core has not yet been released, which means that if you want to test it, you'll have to
first first of all [install Phosphorus Five](https://github.com/polterguy/phosphorusfive/releases), and then
clone or download Hyper Core, and make sure you put it inside of your Phosphorus Five folder structure, 
at _"/core/p5.webapp/modules/"_.

## Warning

This is **ALPHA** stuff, and **NOT** ready for production yet!

If you wish, you can see a video of me demonstrating the system [here](https://www.youtube.com/watch?v=tWAZUAKVBeg).

