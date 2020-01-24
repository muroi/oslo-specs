..
  This template should be in ReSTructured text.  For help with syntax,
  see http://sphinx-doc.org/rest.html

  To test out your formatting, build the docs using tox, or see:
  http://rst.ninjs.org

  The filename in the git repository should match the launchpad URL,
  for example a URL of
  https://blueprints.launchpad.net/oslo?searchtext=awesome-thing should be
  named awesome-thing.rst.

  For specs targeted at a single project, please prefix the first line
  of your commit message with the name of the project.  For example,
  if you're submitting a new feature for oslo.config, your git commit
  message should start something like: "config: My new feature".

  Wrap text at 79 columns.

  Do not delete any of the sections in this template.  If you have
  nothing to say for a whole section, just write: None

  If you would like to provide a diagram with your spec, ascii diagrams are
  required.  http://asciiflow.com/ is a very nice tool to assist with making
  ascii diagrams.  The reason for this is that the tool used to review specs is
  based purely on plain text.  Plain text will allow review to proceed without
  having to look at additional files which can not be viewed in gerrit.  It
  will also allow inline feedback on the diagram itself.

=================================
Proposed new library oslo.metrics
=================================

This is a proposal to create a new library to collect metrics of oslo libraries.

Proposed library mission
=========================

The mission of oslo.metrics is exposing internal metrics infomation of oslo
libraries. OpenStack processes create a connection to other middleware using
oslo library, e.g. oslo.messaging to connect another OpenStack process and
oslo.db to connect DB. The oslo.messaging creates its own RPC protocol over the
connection to RabbitMQ, which is a default messaging middleware in OpenStack
project. The usage of RabbitMQ can be monitored by RabbitMQ management tool,
but the usage of oslo.messaging's RPC can't be monitored. Enabling OpenStack
admin and operator to monitor usage of oslo libraries is the goal of
oslo.metrics.

The oslo.metrics supports metrics of the oslo.messaging's RPC as first goal.
The metrics infomation of the RPC doesn't appear anywhere. For example, when
an user calls the Create New Instance API to Nova, there is no infomation about
how many rpc are called, which rpc target are used, and etc. And for another
case, if operator adds 10 compute nodes to their OpenStack cluster, how may rpc
will be increased, and etc.

Consuming projects
==================

The OpenStack services which use oslo.messaging are the consuming project
since oslo.messaging is a default RPC mechanism to OpenStack services.

Alternatives library
====================

If oslo.messaging exposes its metrics, the notification feature of
oslo.messaging can be a one of alternatives. However, the notification is
also implemented over the RabbitMQ. It makes cross reference in the metrics
so it's not good idea to do.

Proposed adoption model/plan
============================

The basic architecture is the oslo.metrics works as metrics data serializer
for outside system.
The existing oslo libraries sends original metrics information through
an unix socket, then the oslo.metrics gathers the metrics information
and exposes the data.

The oslo.metrics listens unix sockets to recieve metrics data from
each OpenStack processes. The reason oslo.metrics uses the socket is
to collect messaging metrics information from multi processes which
runs on the same node. One metrics process represents one node or one
OpenStack project.

The oslo.messaging sends the rpc information one by one to oslo.metrics
with oslo.metrics's format. All information sent by oslo.messaging are
putted together into one metrics data in oslo.metrics. Then oslo.metrics
exposes the one data to any monitoring system.

The monitoring system is really depending on operator. So oslo.metrics
exposes the data by common format. For example, Prometheus takes PULL
approach to get a metrics. To support Prometheus, oslo.metrics exposes
the metrics data over HTTP.

.. code-block:: none

 +--------------+        +--------------+     +-------------+
 |              |        |              |     | any         |
 |oslo.messaging+--------> oslo.metrics <-----> monitoring  |
 |              |        |              |     | system      |
 +--------------+        +--------------+     +-------------+
                unix socket


The data oslo.messaging sends to the oslo.metrics includes:

* topic
* namespace
* version
* server
* fanout
* timeout
* type of call: call or cast

Hostname are added by oslo.metrics side.

Reviewer activity
=================

For the changes in oslo.messaging, the support from oslo.messaging core is needed.
For the oslo.metrics, the member of Large Scale SIG could review the patches.

Implementation
==============

Author(s)
---------

Primary authors:
  Masahito Muroi (masahito-muroi)
  <launchpad-id or None>
Other contributors:
  <launchpad-id or None>

Work Items
----------

* Create a new library named oslo.metrics
* Change oslo.messaging to support metrics sending

References
==========

* Discussion in Large-Scale SIG:  https://etherpad.openstack.org/p/large-scale-sig-cluster-scaling

Revision History
================

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Ussuri
     - Introduced

.. note::

  This work is licensed under a Creative Commons Attribution 3.0
  Unported License.
  http://creativecommons.org/licenses/by/3.0/legalcode

