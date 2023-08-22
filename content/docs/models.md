# Fluent models
You can query your database directly using the `Database` class provided by Fluent.

However, there is a much better way of interacting with your database, managing relationships between the various tables in your application or designing CRUDs.

## What is the active record pattern
The active record pattern is an architectural pattern found in software that stores in-memory object data in relational databases. It was named by Martin Fowler in his 2003 book Patterns of Enterprise Application Architecture.

Active Record is also the name of the ORM used by Ruby on Rails, Eloquent or Lucid on Adonis. However, the active record pattern is a broader concept that any programming language or framework can implement.

The active record pattern paradigm emphasizes **the need to know the SQL structure** of your tables **in advance**.

:::note
In fact, each SQL table has its own representation within your application, in the form of a `model` (or called `entity`).
:::

:::codegroup
```dart
// title: Standalone usage
final Map<String, dynamic>? result = await Database.of(manager)
  .query()
  .table('articles')
  .where(column: 'id', value: 1)
  .first();

await Database.of(manager).query()
  .table('articles')
  .where(column: 'id', value: result['id'])
  .update({ 'title': 'My second article' });

exepect(result, isNotNull);
exepect(result, isA<String, String>());
expect(result['title'], equals(payload['title']));
```
```dart
// title: Based on models
final Article article = await Database.of(manager)
  .model<Article>()
  .findOrFail(1);

await article.update({'title': 'My second article'});

exepect(result, isNotNull);
exepect(result, isA<Article>());
expect(article.title, equals(payload['title']));
```
:::

## Defining columns
You will need to define your database columns as class getters.

Since Dart is a strictly typed language, it's difficult to deliver clean, consistent autocompletion without first forcing the definition of fields in the database.

:::warning
It's important to note that a model __does not create__ a table at SQL level.
:::

If your migrations represent your SQL structure, then the templates represent them within your application context.

```dart
// title: Model definition
final class Article extends Model<Article> {
  String get title => properties.get('title');
  String get content => properties.get('content');
}
```

The `DeclareProperty` class, accessible from the `properties` property, lets you access your data bucket.

When you declare class properties, you need to point your `getter` at one of the keys in your bucket like this.

The key used in the bucket is that of your column name in SQL.

```dart
final class Foo extends Foo<Article> {
  String get propertyAccess => properties.get('property_in_database');
}
```

:::note
Your table's primary field (named `id`) is pre-defined in your model's parent class, so you don't need to redefine it.
:::

## Computed properties
Computed properties are properties that are not stored in your database, but are calculated on the fly.
You can define them as class getters.

```dart
final class User extends Model<User> {
  // Properties
  String get firstName => properties.get('firstname');
  String get lastName => properties.get('lastname');
  
  // Computed property
  String get fullName => '$firstName $lastName';
}
```

## Primary key
The primary key is the unique identifier of your table. It's used to identify a specific row in your table.

The primary key is a standard in SQL structure that forces you to make each entry in your table unique.

Because of its unique nature, it is used to index your data and, above all, to establish secure relationships between your tables.

By default, the primary key of each table is set to `id`, but you can override the configuration by re-defining the `primaryKey` key.
```dart
// title: Override primaryKey 
final class Article extends Model<Article> {
  @override
  String get primaryKey => 'other_primary_id';
}
```
