===============
Druid connector
===============

.. raw:: html

  <img src="../_static/img/druid.png" class="connector-logo">

The Druid connector allows querying an `Apache Druid <https://druid.apache.org/>`_
database from Trino.

Requirements
------------

To connect to Druid, you need:

* Druid version 0.18.0 or higher.
* Network access from the Trino coordinator and workers to your Druid broker.
  Port 8082 is the default port.

Configuration
-------------

Create a catalog properties file that specifies the Druid connector by setting
the ``connector.name`` to ``druid`` and configuring the ``connection-url`` with
the JDBC string to connect to Druid.

For example, to access a database as ``druid``, create the file
``etc/catalog/druid.properties``. Replace ``BROKER:8082`` with the correct
host and port of your Druid broker.

.. code-block:: properties

    connector.name=druid
    connection-url=jdbc:avatica:remote:url=http://BROKER:8082/druid/v2/sql/avatica/

You can add authentication details to connect to a Druid deployment that is
secured by basic authentication by updating the URL and adding credentials:

.. code-block:: properties

    connection-url=jdbc:avatica:remote:url=http://BROKER:port/druid/v2/sql/avatica/;authentication=BASIC
    connection-user=root
    connection-password=secret

Now you can access your Druid database in Trino with the ``druiddb`` catalog
name from the properties file.

.. include:: jdbc-common-configurations.fragment

.. |default_domain_compaction_threshold| replace:: ``32``
.. include:: jdbc-domain-compaction-threshold.fragment

.. include:: jdbc-procedures.fragment

.. include:: jdbc-case-insensitive-matching.fragment

.. _druid-type-mapping:

Type mapping
------------

Because Trino and Druid each support types that the other does not, this
connector modifies some types when reading data.

Druid type to Trino type mapping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The connector maps Druid types to the corresponding Trino types according to the
following table:

.. list-table:: Druid type to Trino type mapping
  :widths: 30, 30, 50
  :header-rows: 1

  * - Druid type
    - Trino type
    - Notes
  * - ``STRING``
    - ``VARCHAR``
    -
  * - ``FLOAT``
    - ``REAL``
    -
  * - ``DOUBLE``
    - ``DOUBLE``
    -
  * - ``LONG``
    - ``BIGINT``
    - Except for the special ``_time`` column, which is mapped to ``TIMESTAMP``.
  * - ``TIMESTAMP``
    - ``TIMESTAMP``
    - Only applicable to the special ``_time`` column.

No other data types are supported.

Druid does not have a real ``NULL`` value for any data type. By
default, Druid treats ``NULL`` as the default value for a data type. For
example, ``LONG`` would be ``0``, ``DOUBLE`` would be ``0.0``, ``STRING`` would
be an empty string ``''``, and so forth.

.. include:: jdbc-type-mapping.fragment

.. _druid-sql-support:

SQL support
-----------

The connector provides :ref:`globally available <sql-globally-available>` and
:ref:`read operation <sql-read-operations>` statements to access data and
metadata in the Druid database.

Table functions
---------------

The connector provides specific :doc:`table functions </functions/table>` to
access Druid.

.. _druid-query-function:

``query(varchar) -> table``
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``query`` function allows you to query the underlying database directly. It
requires syntax native to Druid, because the full query is pushed down and
processed in Druid. This can be useful for accessing native features which are
not available in Trino or for improving query performance in situations where
running a query natively may be faster.

As an example, use ``STRING_TO_MV`` and ``MV_LENGTH`` from
`Druid SQL's multi-value string functions <https://druid.apache.org/docs/latest/querying/sql-multivalue-string-functions.html>`_
to split and then count the number of comma-separated values in a column::

    SELECT
      num_reports
    FROM
      TABLE(
        druid.system.query(
          query => 'SELECT
            MV_LENGTH(
              STRING_TO_MV(direct_reports, ",")
            ) AS num_reports
          FROM company.managers'
        )
      );

