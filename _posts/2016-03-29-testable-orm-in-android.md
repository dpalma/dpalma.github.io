---
layout: post
title: Testable ORM in Android with the Repository Pattern
category: Android
comments: yes
---

Testing code that reads and writes the app database in Android is difficult.
ORM libraries make it easy to store and retrieve data, but do not necessarily
make database code more testable. However, taking the ORM approach one step
further and introducing the Repository pattern gives a mockable interface that
allows writing unit tests for code that interacts with the app database.
<!--more-->

There are a few obstacles to unit testing database code in Android.
First, there is no good way to switch between test and production databases.
Data created by automated tests pollutes any user-created data used for manual
testing. Test code gets get much more complicated with the effort to clean up the
database after every test. This is worse when a test malfunctions or crashes.
Test data is left behind, obscuring whatever user-created data might have been present.

The Repository pattern encapsulates the create, read, update, and delete operations
for storing and retrieving data objects to and from the database. Using a
repository instance for all database interactions allows us to introduce mock
repositories for testing. With a mock repository, the logic of such code can be
tested without actually modifying the database.

The code below is a Repository interface built for the [SugarORM](http://satyan.github.io/sugar/)
library for Android. Examples of mock implementations and tests that use them can be seen [here](https://github.com/dpalma/lineup-shaker/tree/master/app/src/androidTest/java/net/danielpalma/lineupshaker).

<script src="https://gist.github.com/dpalma/69992130c0465646baebff2b35af276a.js"></script>
