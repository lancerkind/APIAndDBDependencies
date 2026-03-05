# Liquibase for Spanner
Create the initial database schema based on 00_init.sql as a Liquibase migration for Spanner.  
The Oracle based SQL and tests should be left alone.
This migration will define the initial tables and columns required for the Spanner DB.
Follow Liquibase best practices and conventions for migration using plain SQL for migration and rollback scripts.

# Technical Requirements
- Use Liquibase to define the initial database schema.
- Use an XML changelog that references plain SQL migration scripts via ```<sqlFile>``` 
(do not use liquibase formatted SQL changesets in the .sql files)
- Place plain SQL migration scripts and rollback scripts in ```src/main/migrations/dbSpanner/sql/``` and follow the example of:
```src/main/migrations/dbSpanner/sql/00_initialize.sql and src/main/migrations/dbSpanner/sql/00_rollback_initialize.sql```
- Liquibase is executed by Gradle tasks using the liquibase plugin:
  - liquibaseRuntime "org.liquibase:liquibase-core:5.0.1"
  - liquibaseRuntime "org.liquibase.ext:liquibase-spanner:4.33.0.2"
- Rather than use YAML, use Liquibase's plain SQL support.
- connection details are passed in via Gradle properties to support migrations to a testcontainer or a real GCP project/instance/database.
  - ```-PliquibaseUrl=jdbc:cloudspanner:projects/test-project/instances/test-instance/databases/test-database```
- the master changelog which lists the SQL files is stored in src/main/migrations/dbSpanner/changelog/changelog.xml.
- tag the initial state as 'initial'.
- Create a Java class called LiquibaseMigrations for automated tests for Spanner to call in @BeforeAll. 
- The class will have a method ```run(url, changelogPath)``` to initialize the database.
  - url is the connection details for the database.
  - changelogPath is the path to the changelog.xml file.
- the class will have a method ```rollbackToInitial(url, changelogPath)``` to rollback the database to initial state. 
The tests can hardcode url to the initial state.

- the class will have a method ```rollback(url, changelogPath, count)``` to rollback the database to previous changeset using an argument.

# Acceptance Criteria
- The migration is run by ```./gradlew liquibaseUpdate```
- The database schema can be rolled back via Gradle by ```./gradlew liquibaseRollbackCount -PliquibaseCommandValue=1``` to previous changeset using an argument.
- The database schema can be rolled back via Gradle by ```./gradlew liquibaseRollbackToTag -PliquibaseCommandValue=initial``` to initial state.
- The migration and rollback SQL scripts are referenced in the changelog.
- The testcontainer is updated by the migration.
- The Spanner related automated test runners use LiquibaseMigrations in @BeforeAll to initialize the database.
- A real GCP project/instance/database can be updated by the Gradle based migration.
- The legacy Spanner migration script resources/dbspanner/00_init.sql is removed.

