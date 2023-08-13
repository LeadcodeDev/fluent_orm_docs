# What is Fluent ORM ?

As a Typescript developer at heart, I've always been accustomed to coming across a multitude of packages in the Node.js ecosystem, each one more incredible than the last, to satisfy my every desire.

## No alternative in Dart
When I was learning the [Dart language](https://dart.dev) I was surprised to see, or rather not to have, a wide choice of packages fulfilling the role of an ORM. The only fascinating alternative is Prisma, an adaptation of the world-famous [Prisma ORM](https://www.prisma.io) for the Javascript language.

As a big fan of [Adonis](https://adonisjs.com) as I am, I didn't find the essence of the standards surrounding the database domain, such as migration management, enabling tables to be altered and evolved over time, or the concept of model, the representation of each table within its application.

It was with the aim of solidifying my knowledge in this field while offering a robust and reliable alternative to everyone that I designed **Fluent** : *The model-based ORM for Dart*.

## Example of use

### Make your first model
In this example, we will create a model to represent our table in a database.

```dart
import 'package:fluent_orm/fluent_orm.dart';

final class Article extends Model {
  Article(): super(properties: ['id', 'title', 'content'])
  
  int get id => properties.get('id');
  String get title => properties.get('title');
  String get content => properties.get('content');
}
```

### Structure your database with migrations
In a second step, we will create our database table using the migration system.
```dart
class CategorySchema1691342071 extends Schema {
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

### Retrieve data from the database
Finally, we can consume our table to extract results
```dart
final articles = await Database.of(manager)
  .forModel<Article>()
  .where(column: 'id', value: 1)
  .get();
```
