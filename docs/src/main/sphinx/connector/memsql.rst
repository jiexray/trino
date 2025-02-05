==============================
SingleStore (MemSQL) connector
==============================

.. raw:: html

  <img src="../_static/img/singlestore.png" class="connector-logo">

The SingleStore (formerly known as MemSQL) connector allows querying and
creating tables in an external SingleStore database.

Requirements
------------

To connect to SingleStore, you need:

* SingleStore version 7.1.4 or higher.
* Network access from the Trino coordinator and workers to SingleStore. Port
  3306 is the default port.

.. _singlestore-configuration:

Configuration
-------------

To configure the SingleStore connector, create a catalog properties file
in ``etc/catalog`` named, for example, ``singlestore.properties``, to
mount the SingleStore connector as the ``singlestore`` catalog.
Create the file with the following contents, replacing the
connection properties as appropriate for your setup:

.. code-block:: text

    connector.name=singlestore
    connection-url=jdbc:singlestore://example.net:3306
    connection-user=root
    connection-password=secret

The ``connection-url`` defines the connection information and parameters to pass
to the SingleStore JDBC driver. The supported parameters for the URL are
available in the `SingleStore JDBC driver documentation
<https://docs.singlestore.com/db/v7.6/en/developer-resources/connect-with-application-development-tools/connect-with-java-jdbc/the-singlestore-jdbc-driver.html#connection-string-parameters>`_.

The ``connection-user`` and ``connection-password`` are typically required and
determine the user credentials for the connection, often a service user. You can
use :doc:`secrets </security/secrets>` to avoid actual values in the catalog
properties files.


.. _singlestore-tls:

Connection security
^^^^^^^^^^^^^^^^^^^

If you have TLS configured with a globally-trusted certificate installed on your
data source, you can enable TLS between your cluster and the data
source by appending a parameter to the JDBC connection string set in the
``connection-url`` catalog configuration property.

Enable TLS between your cluster and SingleStore by appending the ``useSsl=true``
parameter to the ``connection-url`` configuration property:

.. code-block:: properties

  connection-url=jdbc:singlestore://example.net:3306/?useSsl=true

For more information on TLS configuration options, see the `JDBC driver
documentation <https://docs.singlestore.com/db/v7.6/en/developer-resources/connect-with-application-development-tools/connect-with-java-jdbc/the-singlestore-jdbc-driver.html#tls-parameters>`_.

Multiple SingleStore servers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can have as many catalogs as you need, so if you have additional
SingleStore servers, simply add another properties file to ``etc/catalog``
with a different name (making sure it ends in ``.properties``). For
example, if you name the property file ``sales.properties``, Trino
will create a catalog named ``sales`` using the configured connector.

.. include:: jdbc-common-configurations.fragment

.. |default_domain_compaction_threshold| replace:: ``32``
.. include:: jdbc-domain-compaction-threshold.fragment

.. include:: jdbc-procedures.fragment

.. include:: jdbc-case-insensitive-matching.fragment

.. include:: non-transactional-insert.fragment

Querying SingleStore
--------------------

The SingleStore connector provides a schema for every SingleStore *database*.
You can see the available SingleStore databases by running ``SHOW SCHEMAS``::

    SHOW SCHEMAS FROM singlestore;

If you have a SingleStore database named ``web``, you can view the tables
in this database by running ``SHOW TABLES``::

    SHOW TABLES FROM singlestore.web;

You can see a list of the columns in the ``clicks`` table in the ``web``
database using either of the following::

    DESCRIBE singlestore.web.clicks;
    SHOW COLUMNS FROM singlestore.web.clicks;

Finally, you can access the ``clicks`` table in the ``web`` database::

    SELECT * FROM singlestore.web.clicks;

If you used a different name for your catalog properties file, use
that catalog name instead of ``singlestore`` in the above examples.

.. _singlestore-type-mapping:

Type mapping
------------

Because Trino and Singlestore each support types that the other does not, this
connector modifies some types when reading or writing data. Data types may not
map the same way in both directions between Trino and the data source. Refer to
the following sections for type mapping in each direction.

Singlestore to Trino read type mapping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The connector maps Singlestore types to the corresponding Trino types following
this table:

.. list-table:: Singlestore to Trino type mapping
  :widths: 30, 20, 50
  :header-rows: 1

  * - Singlestore type
    - Trino type
    - Notes
  * - ``BIT``
    - ``BOOLEAN``
    -
  * - ``BOOLEAN``
    - ``BOOLEAN``
    -
  * - ``TINYINT``
    - ``TINYINT``
    -
  * - ``TINYINT UNSIGNED``
    - ``SMALLINT``
    -
  * - ``SMALLINT``
    - ``SMALLINT``
    -
  * - ``SMALLINT UNSIGNED``
    - ``INTEGER``
    -
  * - ``INTEGER``
    - ``INTEGER``
    -
  * - ``INTEGER UNSIGNED``
    - ``BIGINT``
    -
  * - ``BIGINT``
    - ``BIGINT``
    -
  * - ``BIGINT UNSIGNED``
    - ``DECIMAL(20, 0)``
    -
  * - ``DOUBLE``
    - ``DOUBLE``
    -
  * - ``REAL``
    - ``DOUBLE``
    -
  * - ``DECIMAL(p, s)``
    - ``DECIMAL(p, s)``
    - See :ref:`Singlestore DECIMAL type handling <singlestore-decimal-handling>`
  * - ``CHAR(n)``
    - ``CHAR(n)``
    -
  * - ``TINYTEXT``
    - ``VARCHAR(255)``
    -
  * - ``TEXT``
    - ``VARCHAR(65535)``
    -
  * - ``MEDIUMTEXT``
    - ``VARCHAR(16777215)``
    -
  * - ``LONGTEXT``
    - ``VARCHAR``
    -
  * - ``VARCHAR(n)``
    - ``VARCHAR(n)``
    -
  * - ``LONGBLOB``
    - ``VARBINARY``
    -
  * - ``DATE``
    - ``DATE``
    -
  * - ``TIME``
    - ``TIME(0)``
    -
  * - ``TIME(6)``
    - ``TIME(6)``
    -
  * - ``DATETIME``
    - ``TIMESTAMP(0)``
    -
  * - ``DATETIME(6)``
    - ``TIMESTAMP(6)``
    -
  * - ``JSON``
    - ``JSON``
    -

No other types are supported.

Trino to Singlestore write type mapping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The connector maps Trino types to the corresponding Singlestore types following
this table:

.. list-table:: Trino to Singlestore type mapping
  :widths: 30, 20, 50
  :header-rows: 1

  * - Trino type
    - Singlestore type
    - Notes
  * - ``BOOLEAN``
    - ``BOOLEAN``
    -
  * - ``TINYINT``
    - ``TINYINT``
    -
  * - ``SMALLINT``
    - ``SMALLINT``
    -
  * - ``INTEGER``
    - ``INTEGER``
    -
  * - ``BIGINT``
    - ``BIGINT``
    -
  * - ``DOUBLE``
    - ``DOUBLE``
    -
  * - ``REAL``
    - ``FLOAT``
    -
  * - ``DECIMAL(p, s)``
    - ``DECIMAL(p, s)``
    - See :ref:`Singlestore DECIMAL type handling <singlestore-decimal-handling>`
  * - ``CHAR(n)``
    - ``CHAR(n)``
    -
  * - ``VARCHAR(65535)``
    - ``TEXT``
    -
  * - ``VARCHAR(16777215)``
    - ``MEDIUMTEXT``
    -
  * - ``VARCHAR``
    - ``LONGTEXT``
    -
  * - ``VARCHAR(n)``
    - ``VARCHAR(n)``
    -
  * - ``VARBINARY``
    - ``LONGBLOB``
    -
  * - ``DATE``
    - ``DATE``
    -
  * - ``TIME(0)``
    - ``TIME``
    -
  * - ``TIME(6)``
    - ``TIME(6)``
    -
  * - ``TIMESTAMP(0)``
    - ``DATETIME``
    -
  * - ``TIMESTAMP(6)``
    - ``DATETIME(6)``
    -
  * - ``JSON``
    - ``JSON``
    -

No other types are supported.

.. _singlestore-decimal-handling:

Decimal type handling
^^^^^^^^^^^^^^^^^^^^^

``DECIMAL`` types with precision larger than 38 can be mapped to a Trino ``DECIMAL``
by setting the ``decimal-mapping`` configuration property or the ``decimal_mapping`` session property to
``allow_overflow``. The scale of the resulting type is controlled via the ``decimal-default-scale``
configuration property or the ``decimal-rounding-mode`` session property. The precision is always 38.

By default, values that require rounding or truncation to fit will cause a failure at runtime. This behavior
is controlled via the ``decimal-rounding-mode`` configuration property or the ``decimal_rounding_mode`` session
property, which can be set to ``UNNECESSARY`` (the default),
``UP``, ``DOWN``, ``CEILING``, ``FLOOR``, ``HALF_UP``, ``HALF_DOWN``, or ``HALF_EVEN``
(see `RoundingMode <https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/math/RoundingMode.html#enum.constant.summary>`_).

.. include:: jdbc-type-mapping.fragment

.. _singlestore-sql-support:

SQL support
-----------

The connector provides read access and write access to data and metadata in
a SingleStore database.  In addition to the :ref:`globally available
<sql-globally-available>` and :ref:`read operation <sql-read-operations>`
statements, the connector supports the following features:

* :doc:`/sql/insert`
* :doc:`/sql/delete`
* :doc:`/sql/truncate`
* :doc:`/sql/create-table`
* :doc:`/sql/create-table-as`
* :doc:`/sql/drop-table`
* :doc:`/sql/alter-table`
* :doc:`/sql/create-schema`
* :doc:`/sql/drop-schema`

.. include:: sql-delete-limitation.fragment

.. include:: alter-table-limitation.fragment

Performance
-----------

The connector includes a number of performance improvements, detailed in the
following sections.

.. _singlestore-pushdown:

Pushdown
^^^^^^^^

The connector supports pushdown for a number of operations:

* :ref:`join-pushdown`
* :ref:`limit-pushdown`
* :ref:`topn-pushdown`

.. include:: join-pushdown-enabled-false.fragment

.. include:: no-pushdown-text-type.fragment
