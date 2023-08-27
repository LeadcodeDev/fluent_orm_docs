# Relations

Models can be linked to each other in a number of ways. Relationships are defined in the model's `relationships` property.

Relations can be defined with `HasOne`, `HasMany`, `BelongTo` or `ManyToMany` method of the model, or by defining the relationship directly in the `relations` property of the model.

## HasOne

HasOne creates a `one to one` relationship between two models. For example, A `user` has __only one__ `profil`. The has one relationship needs a foreign key in the related table.

Following is an example table structure for the has one relationship. The `profils.user_id` is the foreign key and forms the relationship with the users.id column.
![has one relation](/assets/images/has_one_relation.webp)

:::codegroup
```dart
// title: users
final class UsersSchema extends Schema {
  final String tableName = 'users';

  @override
  Future<void> up () async {
    schema.createTable(tableName, (table) {
      table.increments('id');
      table.string('email').unique().notNullable();
      table.string('password').notNullable();
    });
  }

  @override
  Future<void> down () async {
    schema.dropTable(tableName);
  }
}
```
```dart
// title: profiles
final class ProfilesSchema extends Schema {
  final String tableName = 'profiles';

  @override
  Future<void> up () async {
    schema.createTable(tableName, (table) {
      table.increments('id');
      table.string('firstname').notNullable();
      table.string('lastname').notNullable();
      table.integer('user_id').references(column: 'id', table: 'users');
    });
  }

  @override
  Future<void> down () async {
    schema.dropTable(tableName);
  }
}
```
:::

### Defining a relationship
In this example, we need to declare a relationship within the `User` model in order to perform a junction to the `Profile` model.

```dart
final class User extends Model<User> {
  // highlight-start
  User(): super(
    relations: [
      Relation<Profil>.hasOne()
    ]
  )
  // highlight-end

  // highlight-start
  Profil get profile => model.hasOne<Profile>();
  // highlight-end
}
```

### Override relationship keys
In some cases, it may be necessary to modify the name given to certain keys to enable a possible junction between 2 identical tables within the same model.

However, you can also define a custom foreign key.
```dart
final class Article extends Model<Article> {
  User(): super(
    relations: [
      Relation<Profil>.hasOne(
        // highlight-start
        localKey: 'id',
        foreignKey: 'profil_user_id'
        // highlight-end
      )
    ]
  )
}
```
:::warning
The local key is always the **primary key of the parent model** but can also be defined explicitly.
:::

## HasMany
HasMany creates a `one to many` relationship between two models. For example, A `category` has __many__ `articles`. The has many relationship needs a foreign key in the related table.

Following is an example table structure for the has one relationship. The `articles.category_id` is the foreign key and forms the relationship with the categories.id column.
![has many relation](/assets/images/has_many_relation.webp)

:::codegroup
```dart
// title: categories
final class CategorySchema extends Schema {
  final String tableName = 'categories';

  @override
  Future<void> up () async {
    schema.createTable(tableName, (table) {
      table.increments('id');
      table.string('label').notNullable();
    });
  }

  @override
  Future<void> down () async {
    schema.dropTable(tableName);
  }
}
```
```dart
// title: articles
final class ArticlesSchema extends Schema {
  final String tableName = 'articles';

  @override
  Future<void> up () async {
    schema.createTable(tableName, (table) {
      table.increments('id');
      table.string('title').notNullable();
      table.text('content');
      table.integer('category_id').references(column: 'id', table: 'categories');
    });
  }

  @override
  Future<void> down () async {
    schema.dropTable(tableName);
  }
}
```
:::

### Defining a relationship
In this example, we need to declare a relationship within the `Category` model in order to perform a junction to the `Article` model.
```dart
final class Category extends Model<Category> {
  // highlight-start
  Category(): super(
    relations: [
      Relation<Article>.hasMany()
    ]
  )
  // highlight-end

// highlight-start
  List<Article> get articles => model.hasMany<Article>();
// highlight-end
}
```
### Override relationship keys
In some cases, it may be necessary to modify the name given to certain keys to enable a possible junction between 2 identical tables within the same model.

However, you can also define a custom foreign key.
```dart
final class Category extends Model<Category> {
  Category(): super(
    relations: [
      Relation<Article>.hasMany(
        // highlight-start
        localKey: 'id',
        foreignKey: 'category_id'
        // highlight-end
      )
    ]
  )
}
```
:::warning
The local key is always the **primary key of the parent model** but can also be defined explicitly.
:::

## ManyToMany
HasMany creates a `many to many` relationship between two models. For example, A `article` has __many__ `tag` **and** `tag` has __many__ `article`. The many to many relationship needs a foreign key in the related table.

In the case of a ManyToMany relationship, we need to create a 3rd table called `pivot`, which will act as a junction between the two initial models.

:::note
This pivot table is named by the singular combination of the two tables linked by the relationship and ordered alphabetically by their table name.
:::

Following is an example table structure for the has one relationship.

![many to many relation](/assets/images/many_to_many_relation.webp)

:::codegroup
```dart
// title: tags
final class TagSchema extends Schema {
  final String tableName = 'tags';

  @override
  Future<void> up () async {
    schema.createTable(tableName, (table) {
      table.increments('id');
      table.string('label').notNullable();
    });
  }

  @override
  Future<void> down () async {
    schema.dropTable(tableName);
  }
}
```
```dart
// title: articles
final class ArticlesSchema extends Schema {
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
```dart
// title: article_tag
final class ArticleTagSchema extends Schema {
  final String tableName = 'article_tag';

  @override
  Future<void> up () async {
    schema.createTable(tableName, (table) {
      table.increments('id');
      table.integer('article_id').references(column: 'id', table: 'articles');
      table.integer('tag_id').references(column: 'id', table: 'tags');
    });
  }

  @override
  Future<void> down () async {
    schema.dropTable(tableName);
  }
}
```
:::

### Defining a relationship
In this example, we need to declare a relationship within the `Category` model in order to perform a junction to the `Article` model.
```dart
final class Article extends Model<Article> {
  // highlight-start
  Article(): super(
    relations: [
      Relation<Article>.manyToMany()
    ]
  )
  // highlight-end

// highlight-start
  List<Tag> get tags => model.manyToMany<Tag>();
// highlight-end
}
```
### Override relationship keys
In some cases, it may be necessary to modify the name given to certain keys to enable a possible junction between 2 identical tables within the same model.

However, you can also define a custom foreign key.
```dart
final class Article extends Model<Article> {
  Article(): super(
    relations: [
      Relation<Tag>.manyToMany(
        // highlight-start
        localKey: 'id',
        foreignKey: 'article_id',
        pivotForeignKey: 'id',
        pivotRelatedForeignKey: 'tag_id'
        // highlight-end
      )
    ]
  )
}
```
:::warning
The local key is always the **primary key of the parent model** but can also be defined explicitly.
:::

## BelongTo
The `BelongTo` relationship can be used as the inverse of any relationship to find its "parent" except `ManyToMany`.

:::note
In the case of a ManyToMany relationship, you'll need to use the same [`ManyToMany`](relationships#manytomany) relationship.
:::

Consider the hasOne relationship example below. From the `Profil` model, you can define a reverse relationship to the `User` model.

![has one relation](/assets/images/has_one_relation.webp)
From the user's point of view, the `Profile` model is the target model of a `HasOne` relationship, but from the `Profile` model's point of view, it represents a `BelongTo` relationship to the `User` model.

### Defining a relationship
In this example, we need to declare a relationship within the `User` model in order to perform a junction to the `Profile` model.

```dart
final class Profile extends Model<Profile> {
  // highlight-start
  Profile(): super(
    relations: [
      Relation<User>.belongTo()
    ]
  )
  // highlight-end
}
```

### Override relationship keys
In some cases, it may be necessary to modify the name given to certain keys to enable a possible junction between 2 identical tables within the same model.

However, you can also define a custom foreign key.
```dart
final class Article extends Model<Article> {
  User(): super(
    relations: [
      Relation<Profil>.belongTo(
        // highlight-start
        localKey: 'user_id',
        foreignKey: 'id'
        // highlight-end
      )
    ]
  )
}
```
:::warning
The local key is always the **primary key of the parent model** but can also be defined explicitly.
:::
