[DBAL](http://www.doctrine-project.org/projects/dbal.html)is the database abstraction layer in the Doctrine project. The ORM in Doctrine is built on top of this abstraction layer. DBAL is a thin layer on top of a PDO like API. This means a database implementation does not need to use PDO internally. Potentially good for Akiban as we could use the native PostgreSQL PHP driver in that case.

DBAL can be used independently of the ORM.

# Installing on OSX