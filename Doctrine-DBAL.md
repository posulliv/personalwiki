[DBAL](http://www.doctrine-project.org/projects/dbal.html) is the database abstraction layer in the Doctrine project. The ORM in Doctrine is built on top of this abstraction layer. DBAL is a thin layer on top of a PDO like API. This means a database implementation does not need to use PDO internally. Potentially good for Akiban as we could use the native PostgreSQL PHP driver in that case.

DBAL can be used independently of the ORM.

# Akiban Implementation

To implement support for Akiban Server, the following interfaces and abstract classes need to be implemented:

 * \Doctrine\DBAL\Driver\Driver
 * \Doctrine\DBAL\Driver\Statement
 * \Doctrine\DBAL\Platforms\AbstractPlatform
 * \Doctrine\DBAL\Schema\AbstractSchemaManager

The DriverManager class in `lib/Doctrine/DBAL/DriverManager.php` needs to know about Akiban Server. 

# PostgreSQL Implementation

# Oracle Implementation