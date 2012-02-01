Software Architecture Overview
==============================


Scope
:::::

The scope of this section is to help a developer to understand the deployment configuration and to provide
an understanding of how to modify the existing features and to create new ones. The system is composed of
many different modules and each requires knowledge of the used technology, this section provides a global vision
and does not attempt to try to explain everything.

Description
:::::::::::

To describe the architecture of Bungeni, UML will be used. Some snippets of code will be provided where necessary to
describe bash file configurations, python code, paster and deliverance configuration files.

Logical Views
:::::::::::::

Logical Elements
----------------

The main logical elements are described below:

    * Theming component: this provides a coherent look and feel to the BungeniPortal and BungeniCMS.
    * Dispatcher redirects the incoming requests to the correct URL inside the main applications.
    * BugneniPortal provides the parliament functionalities.
    * BugeniDB is a relational DB, it stores the data of BungeniPortal
    * BungeniCMS contains the general materials, it manages the content that is stored in the system. BungeniCMS uses an integrated object DB (not shown in the diagram)
    * Static resources are the images and CSS files which are served straight from the server's file system.

Diagram 1: Logical Elements

.. image:: images/logical.png

Theme Delivering
----------------

This component provides a coherent look-and-feel across all the applications in the site. The HTML coming from a source (e.g. Bungeni Portal or
BungeniCMS) is re-written based on a "theme" which is a static HTML page: the component extracts the parts from the page and fills the empty spaces
in the static template. This operation is done using a set of rules based on an XPath syntax.

Dispatcher
----------

This component simply calls the application using a mapping between URLs and apps.

BungeniPortal
-------------

This is the application that provides the specific parliament features, it can be broken up into the following sections:

    * *Business:* in this area there are the daily operations of the various parliament activities.
    * *What's on:* an overview of the daily operations of the parliament
    * *Committees:* list of committees, for each one there is metadata about the matter of discussion, membership and sittings.
    * *Bills:* list of bills and metadata for each bill. Actions are provided to version the bill and access the workflow associated with the bill.
    * *Questions:* list of questions and associated metadata about the matter of discussion, membership and sittings.
    * *Motions:* list of motions and associated metadata. Workflow and versioning actions are provided.
    * *Tabled documents:* list of tabled documents and metadata. Workflow and versioning actions are provided.
    * *Agenda items:* this is a list of agenda items and metadata.
    * *Sittings:* calendar showing the sittings of the plenary and the committees.
    * *Parliamentary publications:* this is a list of publications and information; these publications are the reports that come out of sittings.
    * *Members:* in this section, one can search for information about members of parliament (MPs)
    * *Member of parliament:* general information such as name and election date.
    * *Personal Info:* a complete biography of the member.
    * *Offices held:* information about offices in which the member has a title
    * *Parliamentary activities:* a list of content workflows the member has participated in. e.g. questions created by the member or motions moved by the member.
    * *Archive:* access to current and historical activities of the parliament, the categories are:

        * Parliaments
        * Political groups
        * Committees
        * Governments
    * *Workspace:* This is available for members of parliament and for clerks. This provides access to to the most relevant and current information for the user in a single page. e.g. for the Member of Parliament - the following

    * *Administration:* This is an administration section provided to the Admin. This is used for adding parliaments, new users, closing parliaments, entering preliminary metatdata etc.

The following diagram shows the logical components of the BungeniPortal.

Diagram 2: Logical Components of BungeniPortal

.. image:: images/logical_components.png

The versions and workflow functionalities provide traversals into the content; for example in a motion there are the links to past workflow states and older versions of the motion -
allowing the user to browse not just the current state of the motion but also the full audited history of the motion.

From some sections a user can reach contents in other sections. An example of this is the "Parliamentary activities" tab of a member,
it is possible to take a look at a bill moved by that MP (member of parliament).


BungeniCMS
----------

This is the content management system part of the portal, it provides a set of functionalities which are designed to:

    * Allow a large number of people of contribute to and share stored contents
    * Control access to contents, based on permissions
    * User roles or group membership define what each user can do (not only edit and view).
    * Improve communication between users using comments on contents.
    * Publication workflow and versioning support

The CMS contains various contents: documents, events, news, pictures, files are the main types.
The information architecture is organized in a tree structure, at this moment it looks as:

    * How we work
    * Rules and regulations
    * How parliament works
    * Seating plan
    * Administrative
    * Reference material
    * History of parliament
    * Online resources
    * Useful links
    * Picture gallery
    * Have your say
    * Vote in the election
    * Become an member of parliament
    * Present a petition
    * Visit parliament

This is the base structure but subject to changes due to the specificity of each parliaments.

Logical relationships
---------------------

The following diagram shows how the different parts of the system communicate with each other:

Diagram 3: Logical Relationships

.. image:: images/logical_relationships.png

The request is passed from the 'theming component' to the dispatcher that call the designated application; the returned
html is processed from 'theming component' and release to the user. In this diagram is missing the 'paster server' component
that provides main access to the web server and manage the `wsgi` messages among the parts. As shown the components are for
the most not dependent upon each other: the 'theming component' an the Dispatcher merge backend applications, the BungeniCMS can
work without the others as the BungeniPortal (in this case however there is an explicit need for the RDBMS to store and retrive data).


Deployment of logical elements to hardware components
-----------------------------------------------------

The starter point is the `supervisord` configuration: supervisord.conf (a file with structure similar to Microsoft Windows INI files).
From this file you can see which services compose the system and how they are started:

.. image:: images/conf_architecture.png

Diagram 4: Deployment

The sections are:

    * program:portal
    * program:plone
    * program:bungeni
    * program:postgres
    * program:openoffice

**program:portal**

Specify how all web servers and applications are reachable, they are served through Paster. Paster is a two-level command and the second level is pluggable, for
Bungeni `serve` that the ``serve`` command is used, which is a script to `serve` applications based on the WSGI interface (*similar to CGI*) using the
http protocol (see `http://pythonpaste.org/script/developer.html <http://pythonpaste.org/script/developer.html>`_). The configuration of `portal` is in
portal/deploy.ini: the `main` section defines a pipeline which filters requests through deliverance and serving a lot of urls: see [pipeline:main] then
[filter:deliverance] and [composite:dispatch] sections. Deliverance provides a uniform theme to all applications (`http://deliverance.openplans.org/index.html`_), it
intercepts the pages and via a set of rules applies a common look-and-feel. IN the `dispatch` section you can see the url mapped:

    * / = plone
    * /plone = plone

these are provided from server specified in [program:plone] of supervisord.conf

    * /static = static

this is provided directly as a `wsgi` service from the module bungeni.portal#static

**program:bungeni**

Bungeni Portal is served through paster with ``deploy.ini``, the mapped urls are ``'/'`` and ``'/cache'``. On ``'/'``
there is the real portal, a pipeline of ``repoze.who`` (WSGI authentication middleware) and bungeni.server (code in src/bungeni.server).
This one uses ``ore.wsgiapp`` and ``site.zcml`` is the Zope 3 instance configuration.

The names provided from bungeni sections are managed from ``bungeni.ui``: the names/urls are implemented as browser pages or
menu actions: see the configuration in ``src/bungeni.ui/bungeni/ui/menu.zcml`` (this requires understanding of zcml zope technology).

**program:plone**

BungeniCMS is based on Plone and it is served through paster (that is unusual for Plone) with the configuration file ``plone/etc/deploy.ini``.
The paster configuration is a pipeline of various middleware at end of which there is Zope2. The BungeniCMS is the `site` instance of Plone in the root of Zope.

**program:postgres**

The configuration to start up the PostgresSQL database server.

Source Code Views
:::::::::::::::::

Main Components
---------------

The following section describes the more programmatic views used by the Bungeni Portal.

Capistrano
----------

`Capistrano <https://github.com/capistrano/capistrano>`_ is a utillity and framework for executing commands in parallel on
multiple remote machines via SSH. It uses a simple DSL (*Domain Specific Language*) that allows you to define *tasks*, which may
be applied to machines in certain roles. It also supports tunnelling connections vai some gateway machine to allow operations
to be performed behind VPN's and firewalls.

Capistrano was originally designed to simplify and automate deployment of web applications to distributed environments, and
originally came bundled with a set of tasks designed for deploying Rails applications. Read the `docs <https://github.com/capistrano/capistrano/wiki/Documentation-v2.x>`_.


Supervisor
----------

`Supervisor <http://supervisord.org>`_ is a client/server system that allows its users to monitor and control a number
of processes on UNIX-like operating systems. It is responsible for starting programs at its own invocation, responding to
commands from clients, restarting crashed or exited subprocesses, logging its subprocess ``stdout`` and ``stderr`` output,
and generating and handling `events` corresponding to points in subprocess lifetimes. It provides a web user interface to view
and control process status.

`The above two components are part of the deployment system.`

Paster
------

Python Paste is a set of libraries to deploy WSGI applications, it covers all aspect of a CGI appliation: testing, dispatcher,
authentication, debugging and deployment. Specifically the Paste Deployment is a system for finding and configuring WSGI applications
and servers, it provides a single, simple function (loadapp) for loading a WSGI application from a configuration file or a Python Egg. The usual
way to deploy a WSGI application is to use ``paster serve``, this command line counterpart to serve an application using  a `Paste Deploy`
configuration file.

Deliverance
-----------

