# Getting Started

The quickstart will help create a new project from scratch so you can start using Fluent.

## Prerequisites
To take advantage of the latest developments in the Dart language, we've chosen to exploit the full power of the new API released with version 3 of the language.
```yaml
environment:
  sdk: ^3.0.5
```

## Your first project

### Create a new Dart project
First, you need to create a new Dart project. If you are not familiar with Dart, you can refer to Official Dart Documentation.
```bash
dart create fluent_demo
```

### Add Fluent to your project
Add the following dependencies to your `pubspec.yaml` file and run `dart pub get` to install them.

:::codegroup
```bash
// title: Installation
dart pub add fluent_orm
```

```yaml
// title: pubspec.yaml
dependencies:
  fluent_orm: ^1.0.0
```
:::

### Create the fluent manager
The manager is a central point in the use of fluent, as it is responsible for the connection to your database, but also allows you to act on it thanks to the QueryBuilder.

:::warning
At present, only [Postgres](https://docs.postgresql.fr/current/app-psql.html) is supported. In the future, we'd like to support [Sqlite](https://en.wikipedia.org/wiki/MariaDB) as well as [MariaDB](https://en.wikipedia.org/wiki/MariaDB).
:::

```dart
import 'package:fluent_orm/fluent_orm.dart';

final psqlClient = PostgresClient(
  database: 'you_database_name',
  host: 'localhost',
  user: 'postgres',
  password: 'postgres',
  port: 5432
);

final FluentManager manager = FluentManager(
  models: [],
  migrations: [],
  client: psqlClient
);

await manager.client.connect();
```

### Structure our database
Now that our client has been created, we need to create the structure of our SQL tables. To do this, we're providing a class dedicated to SQL migrations.

In our example, we're going to create an `articles` table, which will contain discussion topics.

[Discover migrations](migrations)
```dart
// title: Migration
final class Article1691342071 extends Schema {
  final String tableName = 'articles';

  @override
  Future<void> up () async {
    schema.createTable(tableName, (table) {
      table.increments('id');
      table.string('title').notNullable();
      table.text('content');
    });
  }

  @override
  Future<void> down () async {
    schema.dropTable(tableName);
  }
}
```

:::warning
Don't forget to add your migration class into your manager declaration, then you can migrate to your own database.
:::

Although your migration has been created, it is not yet effective in your database.

You must therefore explicitly ask your application to send the contents of your migration via your manager.

:::codegroup
```dart
// title: Declaration
final FluentManager manager = FluentManager(
  // highlight-start
  migrations: [Article1691342071.new],
  // highlight-end
);
```

```dart
// title: Database migration
await manager.runMigration();
```
:::

### Using table into your application
[Active Record](https://en.wikipedia.org/wiki/Active_record_pattern) is the philosophy we follow. It introduces an obligation to represent each SQL table within our application in the form of a class.

This clean alternative makes it possible to encapsulate a multitude of methods, to obtain Model-dependent auto-completion and to know every property available for use.

To represent our SQL table, we will extend the Model class provided by Fluent.

:::codegroup
```dart
// title: Basic model
import 'package:fluent_orm/fluent_orm.dart';

final class Article extends Model<Article> {
  // model declaration
}
```

```dart
// title: Define properties
import 'package:fluent_orm/fluent_orm.dart';

final class Article extends Model<Article> {
  // highlight-start
  int get id => model.property('id');
  String get title => model.property('title');
  String get content => model.property('content');
  // highlight-end
}
```
:::

Now that your template is complete, you need to add it to the manager in the appropriate property
```dart
final FluentManager manager = FluentManager(
  // highlight-start
  models: [Article.new],
  // highlight-end
);
```

Taking into account the migration, model and manager created earlier, we'll now look at a CRUD in the form of the example below.

:::note
The following examples are not intended to be exhaustive; the only limit is your imagination.
:::

:::codegroup
```dart
// title: Create
Future<void> main () async {
  final Map<String, String> payload = {
    'title': 'My first article',
    'content': 'Lorem ipsum dolor sit amet, consectetur adipiscingd.'
  };
  
  final article = await Database.of(manager)
    .model<Article>()
    .create(payload);

  expect(article, isNotNull);
  expect(article, isA<Article>());
}
```

```dart
// title: Get articles
Future<void> main () async {
  final articles = await Database.of(manager)
    .model<Article>()
    .all();

  expect(article, isA<List<Article>>());
}
```

```dart
// title: Get one article
Future<void> main () async {
  final Article? article = await Database.of(manager)
    .model<Article>()
    .find(1);
}
```

```dart
// title: Update article
Future<void> main () async {
  final article = await Database.of(manager)
    .model<Article>()
    .findOrFail(1);
  
  await article.update({ 'title': 'Hello World' });
}
```

```dart
// title: Delete article
Future<void> main () async {
  final article = await Database.of(manager)
    .model<Article>()
    .findOrFail(1);
  
  await article.delete();
}
```
:::
