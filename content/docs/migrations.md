# Schema migrations
Migrations are files used to describe changes to be made to the database. 
They are used to create new tables, add columns, modify existing columns or delete tables.

Migrations must follow the following rules :
- Each migration must be a class extending the `Schema` class.
- Each migration must have a `tableName` property, which will be used to name the table to be created.
- Each migration must have an `up` method, which will be used to create the table.
- Each migration must have a `down` method, which will be used to reverse the current migration.
- Each migration must be named according to the following pattern `{SingularTableName}{timestamp}` (in milliseconds).
- Each migration must be imported and added into the FluentManager` class of the project.

## Creating a migration
A migration class must extend the `Schema` abstract class and implement the `up` and `down` methods.

:::note
The `up` method is used to create the table, while the `down` method is used to cancel the migration.
:::

:::codegroup
```dart
// title: 1. Minimale structure
final class CreateFoo1691342071 extends Schema {
  final String tableName = 'foos';

  @override
  Future<void> up () async {}
  
  @override
  Future<void> down () async {}
}
```
```dart
// title: 2. Create table
final class CreateFoo1691342071 extends Schema {
  final String tableName = 'foos';
  
  @override
  Future<void> up () async {
    // highlight-start
    schema.createTable(tableName, (table) {
      table.increments('id');
      table.string('label').notNullable();
      table.text('description');
    });
    // highlight-end
  }
  
  @override
  Future<void> down () async {}
}
```
```dart
// title: 3. Drop table
final class CreateFoo1691342071 extends Schema {
  final String tableName = 'foos';

  @override
  Future<void> up () async {
    schema.createTable(tableName, (table) {
      table.increments('id');
      table.string('label').notNullable();
      table.text('description');
    });
  }
  
  @override
  Future<void> down () async {
    // highlight-start
    schema.dropTable(tableName);
    // highlight-end
  }
}
```
:::

## Run & rollback migrations
Once you've created the migration files you need, you need to alter your database by sending your migrations within it.

To do this, the `FluentManager` class has an object called `Migrator` which allows migrations to interact with your database.

### Migrator
There are 3 actions you can take to alter your database.
- `run` : This method will run all migrations that have not yet been run.
- `rollback` : This method will cancel the last migration that has been run.
- `fresh` : This method will cancel all migrations that have been run.

:::codegroup
```dart
// title: Run
final FluentManager manager = FluentManager(
  migrations: [CreateFoo1691342071.new],
  client: psqlClient
);

await manager.migrator.run();
```
```dart
// title: Rollback
final FluentManager manager = FluentManager(
  migrations: [CreateFoo1691342071.new],
  client: psqlClient
);

await manager.migrator.rollback();
```
```dart
// title: Fresh
final FluentManager manager = FluentManager(
  migrations: [CreateFoo1691342071.new],
  client: psqlClient
);

await manager.migrator.fresh();
```
:::

### Batches
Batch is a key concept in our migration model.

When you migrate to your database, all your migrations will affect it *once* and will be recorded as batch `1`.

The group of migrations created after the first batch will have their batch value incremented.

Batch migrations allow you to create "groups" of migrations so that you can revert them one after the other, instead of deleting the entire structure of your database each time you decide to rollback your migrations.

### Tracking completed migrations
Fluent follows the file path of migrations performed in the fluent_schemas database table. This avoids re-executing the same migration files.

Here are the columns in the fluent_schemas table.

| id | name                     | batch | migration_time                   |
|----|--------------------------|-------|----------------------------------|
| 1  | create_column_1691342071 | 1     | 2023-08-26 10:41:31.176333+05:30 |
| 2  | create_column_1691342072 | 1     | 2023-08-26 10:41:31.176333+05:30 |

- `name` : The name of the migration file.
- `batch` : The batch number of the migration.
- `migration_time` : The date and time the migration was performed.

## Avoiding backtracking in production
Backtracking during development is perfectly feasible, as there is no fear of data loss. However, backtracking in production is not an option in most cases. Let's take the following example:

- You create and run a migration to configure the user table.
- Over time, this table has received data since the application has been running in production.
- Your table has therefore evolved and you now wish to add a new column to the user table.

You can't simply go back, modify the existing migration and run it again, as going back would delete the user table and all its data.

Instead, you need to create a new migration file to modify the existing user table by adding the required column. In other words, migrations must always go forward.

## Create a table
You can use the `schema.createTable` method to create a new database table. The method accepts the table name as the first argument and a callback function to define the table columns.

```dart
// title: Migration
final class Foo1691342071 extends Schema {
  final String tableName = 'foos';

  @override
  Future<void> up () async {
    // highlight-start
    schema.createTable(tableName, (table) {
      table.increments('id');
      table.string('title').notNullable();
      table.text('content');
    });
    // highlight-end
  }

  @override
  Future<void> down () async {
    schema.dropTable(tableName);
  }
}
```
:::note
See the [Schema Builder](schema-builder#schema-builder) section for more information on the methods available to you.
:::

## Alter a table
You can alter an existing database table using the `schema.alterTable` method. The method accepts the table name as the first argument and a callback function to alter/add the table columns.
```dart
// title: Migration
final class Foo1691342071 extends Schema {
  final String tableName = 'foos';

  @override
  Future<void> up () async {
    // highlight-start
    schema.alterTable(tableName, (table) {
      table.renameColumn('title', 'label');
      table.number('word_count');
    });
    // highlight-end
  }
}
```
## Rename & drop table
You can rename an existing database table using the `schema.renameTable` method. The method accepts the old table name as the first argument and the new table name as the second argument.

:::codegroup
```dart
// title: Rename table
final class Foo1691342071 extends Schema {
  final String tableName = 'foos';

  @override
  Future<void> up () async {
    // highlight-start
    schema.renameTable('foos', 'bars');
    // highlight-end
  }
}
``` 
```dart
// title: Drop table
final class Foo1691342071 extends Schema {
  final String tableName = 'foos';

  @override
  Future<void> down () async {
    // highlight-start
    schema.dropTable(tableName);
    // highlight-end
  }
}
```
:::

## Performing other database operations
When structuring your database, you may be interested in seeding data used during production.

You can seeder your table with the schema.defer method. 
This method accepts a callback function that receives the QueryBuilder instance. You can use this builder to create and modify tables.

```dart
// title: Migration
final class Foo1691342071 extends Schema {
  final String tableName = 'foos';

  @override
  Future<void> up () async {
    schema.createTable(tableName, (table) {
      table.increments('id');
      table.string('title').notNullable();
      table.text('content');
    });

    // highlight-start
    schema.defer((database) async {
      final Map<String, dynamic> payload = {
        'title': 'Hello World',
        'content': 'Lorem ipsum dolor sit amet consectetur.'
      };

      await database.forModel<Foo>()
        .insert(payload)
        .save();
    }
    // highlight-end
  }
}
```
