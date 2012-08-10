[DBAL](http://www.doctrine-project.org/projects/dbal.html) is the database abstraction layer in the Doctrine project. The ORM in Doctrine is built on top of this abstraction layer. DBAL is a thin layer on top of a PDO like API. This means a database implementation does not need to use PDO internally. Potentially good for Akiban as we could use the native PostgreSQL PHP driver in that case.

DBAL can be used independently of the ORM.

# Akiban Implementation

To implement support for Akiban Server, the following interfaces and abstract classes need to be implemented:

 * \Doctrine\DBAL\Driver\Driver (interface)
 * \Doctrine\DBAL\Driver\Statement (class - does not appear to be interface or abstract class?)
 * \Doctrine\DBAL\Platforms\AbstractPlatform (abstract class)
 * \Doctrine\DBAL\Schema\AbstractSchemaManager (abstract class)

The DriverManager class in `lib/Doctrine/DBAL/DriverManager.php` needs to know about Akiban Server. 

# PostgreSQL Implementation

The PostgreSQL implementation is based on PDO. Thus the `PDOStatement` and `PDOConnection` classes are used with PostgreSQL. The only interface implemented in the PostgreSQL driver is the `Driver` interface. That implementation is minimal. The only interesting function is the function that constructs the PDO DSN.

The `AbstractSchemaManager` and `AbstractPlatform` class implementations are more involved. Much of what is in the PostgreSQL implementation for these abstract classes can be used as a basis for Akiban. The main items that will need to be changed are the queries on the `pg_*` tables. Most of these can be converted to queries on tables in the `information_schema` for Akiban. One table that has been verified to be missing as of 08/10 is a table to view information on sequences defined in Akiban.

# Oracle Implementation

Oracle has 2 implementations. The first implementation is based on PDO and as is the case with the PostgreSQL implementation, on the the `Driver` interface is implemented. 

The other Oracle implementation uses the [Oracle Call Interface](http://www.oracle.com/technetwork/database/features/oci/index.html). Since this implementation does not use PDO, it needs to implement all 4 of the required interfaces and abstract classes.

Thus, the Oracle implementation is a good implementation to learn from for performing the Akiban work.