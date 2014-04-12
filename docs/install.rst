Installation
============

The server is a `Node.js`_ application providing a rendez-vous point
for all displays to subscribe using a Websocket protocol as well as an
administration page to manage all displays, group them and tell them
which URL to display.

Server and receiver
-------------------

To install it, you need to execute the following step:

  1. Grab the latest `tarball for Dashkiosk`_ from GitHub.

  2. Install `Node.js`_ and optionally `npm`_. Currently, *Dashkiosk*
     only works with *Node.js* 0.10.x which is the current stable
     version. If the version available in your distribution is not
     up-to-date, have a look at `how to install Node.js via the
     package manager`_ before trying to build from the sources.

  3. Install *bower* and *grunt*, two package managers for Javascript
     with the following command::

         $ npm install -g bower grunt-cli

  4. Unpack *Dashkiosk* in the directory of your choice and go into
     that directory.

  5. Install the appropriate dependencies with the following commands::

         $ npm install
         $ bower install

  6. Build the final version of *Dashkiosk* with the following command::

         $ grunt

  7. Upon success, you will get a `dist` directory that you can put on
     some server. It includes both the receiver and the server
     part. If you want to use the Android application, you still need
     to build it and install it. See :doc:`android`.

Branding
--------

As is, *Dashkiosk* is branded for `Deezer`_. If you do not like that,
there are two images that you should modify before building the final
version with ``grunt``.

  - The spinning vinyl is located in ``app/images/loading.svg``. You
    should be able to replace it by anything that will give a cool
    effect while spinning. In this case, please submit it back.

  - The *Deezer* logo appearing both in the administration interface
    and in the default dashboard is located in
    ``app/images/stamp.svg``. Put your favorite logo instead.

Database
--------

*Dashkiosk* stores its data inside some database. By default, it uses
SQLite. If you prefer to use another database, this is quite easy. We
will use `PostgreSQL`_ but this should be easy to transpose to another
database supported by `Sequelize.js`_, the ORM used in
*Dashkiosk*. The databases currently supported are:

 - MySQL,
 - MariaDB,
 - SQLite, and
 - PostgreSQL.

Here are the steps:

  1. Create a dedicated user inside your RDBMS. For *PostgreSQL*, this
     is done as the ``postgres`` user with the following command::

           $ createuser -P dashkiosk
           Enter password for new role: 
           Enter it again: 
           Shall the new role be a superuser? (y/n) n
           Shall the new role be allowed to create databases? (y/n) n
           Shall the new role be allowed to create more new roles? (y/n) n

  2. Create an empty database. For *PostgreSQL*, this is also done as
     the ``postgres`` user::

           $ createuser -O dashkiosk dashkiosk

The database will be populated automatically when running *Dashkiosk*
for the first time.

.. _Node.js: http://nodejs.org/
.. _npm: https://www.npmjs.org
.. _how to install Node.js via the package manager: https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager
.. _tarball for Dashkiosk: https://github.com/vincentbernat/dashkiosk/releases
.. _Deezer: http://www.deezer.com
.. _PostgreSQL: http://www.postgresql.org
.. _Sequelize.js: http://sequelizejs.com