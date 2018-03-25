## A generic HTTP REST backend

Hyper Core is a generic HTTP REST backend that allows you to evaluate server side Hyperlambda,
as a consequence of invoking some HTTP REST request. Hyper Core is mostly for developers who already knows
how to create and maintain HTML, CSS, and JavaScript based applications - And probably an easier way to create
apps, unless this applies to you, is to use Hyperlambda's _"Managed Ajax"_ approach.

**Important** - Some parts of this documentation, requires you to having followed our _"AngularJS tutorial"_,
since it relies upon the same database schema as that tutorial creates. It might be beneficial for
you to first go through that tutorial in the help files for Hyper IDE, before you continue reading this.
If you haven't followed this tutorial yet, you can watch the following video to see how to create this app.

https://www.youtube.com/watch?v=Zr5f7oweed8

### URL overview

Being a generic HTTP REST based backend, the URLs you can create with Hyper Core is obviously of importance.
There are 4 basic URLs you can construct, which are listed below.

* `/hyper-core/x/[function]` - Invokes some _"[function]"_ Hyperlambda file created by you, without any MySQL bindings
* `/hyper-core/mysql/[database]/[function]/[x]` - Invokes some _"[function]"_ Hyperlambda file created by you, with the _"[database]"_ connection already open
* `/hyper-core/mysql/[database]/[table]/[operation]` - Selects, updates, inserts or deletes from the specified MySQL _"[table]"_ in the specified _"[database]"_
* `/hyper-core/auth` - Authentication services

**Notice**, Hyper Core can also be used towards other databases, such as MongoDB or MS SQL. However,
if you choose this route, you'll have to use the `x` methods, since there are no existing
wrappers around any other databases than MySQL at the moment. You will also have to create your own
C#/F#/VB.NET/etc Active Events to access such alternative databases. You can watch the video below
for an introduction on how to create your own C# Active Events.

https://www.youtube.com/watch?v=sUeRdmzRwbs
