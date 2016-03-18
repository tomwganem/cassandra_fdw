cstar_fdw
=============

Foreign Data Wrapper (FDW) that facilitates access to Cassandra 3.0+
from within PG 9.5+

Cassandra: http://cassandra.apache.org/

## Prepare

In addition to normal PostgreSQL FDW pre-reqs, the primary specific
requirement for this FDW is the Cassandra2 C/C++ driver version 2.3
(https://github.com/datastax/cpp-driver) which we will set up as part of
the following section.

## Build

### Download the source

First, download the source code under the contrib subdirectory of the
PostgreSQL source tree and change into the FDW subdirectory:

```
cd cstar_fdw
```

### Build and Install cpp-driver

Check out version 2.3 of the cpp-driver:

```
git clone git@github.com:datastax/cpp-driver.git
cd cpp-driver
git checkout 2.3.0
```

Next, build and install it:

```
cmake .
make && sudo make install
```

### Build and Install cstar_fdw

```
cd ..
USE_PGXS=1 make
USE_PGXS=1 make install
```

## Usage

The following parameters can be set on a Cassandra foreign server
object:

  * **`host`**: the address(es) or hostname(s) of the Cassandra server, Examples: "127.0.0.1" "127.0.0.1,127.0.0.2", "server1.domain.com".
  * **`port`**: the port number of the Cassandra server(s). Defaults to 9042.

The following parameters can be set on a Cassandra foreign table object:

  * **`schema_name`**: the name of the Cassandra keyspace to query.  Defaults to "public".
  * **`table_name`**: the name of the Cassandra table to query.  Defaults to the foreign table name used in the relevant CREATE command.

Here is an example:

```
	-- load EXTENSION first time after install.
	CREATE EXTENSION cstar_fdw;

	-- create server object
	CREATE SERVER cass_serv FOREIGN DATA WRAPPER cstar_fdw
		OPTIONS(host '127.0.0.1,127.0.0.2', port '9042');

	-- Create a user mapping for the server.
	CREATE USER MAPPING FOR public SERVER cass_serv OPTIONS(username 'test', password 'test');

	-- CREATE a FOREIGN TABLE on the server.
	--
	-- Note that a valid "primary_key" OPTION is required in order to use
	-- UPDATE or DELETE support.
	CREATE FOREIGN TABLE test (id int) SERVER cass_serv OPTIONS
        (schema_name 'example', table_name 'oorder', primary_key 'id');

	-- Query the foreign table.
	SELECT * FROM test limit 5;

        -- import Cassandra schema as a foreign schema. Requires TEST_SCHEMA to exist in
        -- Cassandra as well as in PostgreSql.
        IMPORT FOREIGN SCHEMA TEST_SCHEMA FROM SERVER  cassandra2_test_server INTO TEST_SCHEMA;
```
IMPORT FOREIGN SCHEMA

        The Objects imported would follow the same case sensitivity as that of Cassandra.
        This applies to the other identifiers as well such as column names etc. The case
        sensitivity also applies for the Object names referenced in the LIMIT TO and EXCEPT
        clauses.

Here are some examples:

        -- The Test_Tab1 was created as case sensitive in Cassandra and test_tab2 was created
        -- as case insensitive. Only the tables "Test_Tab1" and test_tab2 are imported from
        -- the Cassandra TEST_SCHEMA keyspace. If there are existing objects in the PostgreSql
        -- foreign schema TEST_SCHEMA they would remain.
        IMPORT FOREIGN SCHEMA TEST_SCHEMA LIMIT TO ("Test_Tab1", test_tab2) FROM SERVER  cassandra2_test_server INTO TEST_SCHEMA;

        -- Import all other objects from the Cassandra TEST_SCHEMA schema except "Test_Tab1" and test_tab2.
        IMPORT FOREIGN SCHEMA TEST_SCHEMA EXCEPT ("Test_Tab1", test_tab2) FROM SERVER  cassandra2_test_server INTO TEST_SCHEMA;
