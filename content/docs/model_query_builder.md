# Queries based on model

As described in our section on templates, their implementation is essential if you want to be able to use the full functionality of the ORM.

Templates provide access to a query builder that simplifies the multitude of chainings normally required for a simple query, or allow the pre-loading of associated relations and declarations within them.

## Querying
The QueryBuilder is accessed from the `Database.of(manager).model<Model>()` object, which initiates a query builder dedicated to the current model.

:::note
If a query returns data, it will automatically be instantiated as a Model by the generic.
:::

In the following example, we will consider an Article model with the following structure

:::codegroup
```dart
// title: Model
final class Article extends Model<Article> {
  String get title => model.property('title');
  String get content => model.property('content');
}
```
```dart
// title: Schema
final class ArticleSchema1691342072 extends Schema {
  final String tableName = 'articles';

  @override
  Future<void> up() async {
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
:::

### Create
Create a new model instance, save it to the database then return on the Article instance.
Article cannot be `null`.
```dart
final article = await Database.of(manager)
  .model<Article>()
  .create({});

expect(article, notNull);
expect(article, isA<Article>());
```

### CreateMany
Create multiple model instances, save them to the database then return on the Article instance.
```dart
final articles = await Database.of(manager)
  .model<Article>()
  .createMany([{}, {}]);

expect(articles, isA<List<Article>>());
expect(articles, hasLength(2));
```

### Find
The `Find` method retrieves the first element of your table whose primary column is the value passed in parameter, the result can be `null`.

:::note
The primary column is defined by the `primaryKey` property of your model, you can override it if you want to use another column as primary key.

See [model](models#primary-key) section for more details.
:::

```dart
final Article? article = await Database.of(manager)
  .model<Article>()
  .find(1);
```

### FindBy
The `FindBy` method retrieves the first element of your table whose given column has the value passed in parameter, the result can be `null`.

```dart
final Article? article = await Database.of(manager)
  .model<Article>()
  .findBy(column: 'id', value: 1);
```

### FindOrFail
The `FindOrFail` method retrieves the first element of your table whose primary column is the value passed in parameter, the result can't be `null`.

:::note
The primary column is defined by the `primaryKey` property of your model, you can override it if you want to use another column as primary key.

See [model](models#primary-key) section for more details.
:::

```dart
final article = await Database.of(manager)
  .model<Article>()
  .findOrFail(1);

expect(article, allOf([notNull, isA<Article>()]);
expect(article.id, equals(1));
```

### FindByOrFail
The `FindByOrFail` method retrieves the first element of your table whose given column has the value passed in parameter, the result can't be `null`.

```dart
final Article article = await Database.of(manager)
  .model<Article>()
  .findByOrFail(column: 'id', value: 1);

expect(article, allOf([notNull, isA<Article>()]);
expect(article.id, equals(1));
```

### First
The `First` method retrieves the first element of your table, result can be `null`.
```dart
final Article? article = await Database.of(manager)
  .model<Article>()
  .first();
```
### FirstOrFail
The `FirstOrFail` method retrieves the first element of your table, if the result is `null`, it throws a `QueryException`.
```dart
final Article article = await Database.of(manager)
  .model<Article>()
  .firstOrFail();
```

### All
The `all` method retrieves your entire table, and the result is always an `List`.
```dart
final Article article = await Database.of(manager)
  .model<Article>()
  .all();
```
