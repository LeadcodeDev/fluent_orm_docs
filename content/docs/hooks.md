# Hooks
Hooks are the actions that you can perform against a model instance during a pre-defined life cycle event. Using hooks, you can encapsulate specific actions within your models vs. writing them everywhere inside your codebase.

## Basic hook example

A great example of hooks is password hashing. You can define a hook that runs before the save call and converts the plain text password to a hash.
```dart
import 'package:crypto/crypto.dart';

final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: HookModel(
      beforeSave: [
        (Map<String, dynamic> user) {
          final bytes = utf8.encode(user.password);
          user['password'] = sha1.convert(bytes);
        }
      ]
    )
    // highlight-end
  );
}
```

- The hooks are asynchronous in nature, so you can use the `await` keyword.
- All hooks start with `before<action>` are executed before the action they represent.
:::note
In the case of `BeforeHook`, we can't provide you with a "fake" or "algebraic" instance of your model containing only your data, because we use getters in our model. 
It would be impossible for you to alter them.
:::

## Improve code readability
You can improve the quality and readability of your code by deporting your hook into a separate `static` function.

:::note
We recommend that you deport your business logic to a dedicated function in order to preserve the "single responsibility" concept of the SOLID paradigm.
:::

```dart
import 'package:crypto/crypto.dart';

final class User extends Model<User> {
  User(): super(
    hooks: HookModel(
      // highlight-start
      beforeSave: [hashPassword]
      // highlight-end
    )
  );
  
  // highlight-start
  static Future<void> hashPassword(Map<String, dynamic> user) async {
    final bytes = utf8.encode(user.password);
    user['password'] = sha1.convert(bytes);
  }
  // highlight-end
}
```

## Available hooks
Each hook is linked to an event in your model's lifecycle, and some hooks are executed in more than one case.

:::note
Hooks can only be used in the context of [models](models).
:::

### BeforeCreate
The `BeforeCreate` hook lets you alter your model's data **before** its insertion and persistence in the database.

:::codegroup
```dart
// title: Constructor
final class User extends Model<User> {
  User(): super(
    hooks: HookModel(
      // highlight-start
      beforeCreate: [
        (Map<String, dynamic> user) {
          user['created_at'] = DateTime.now().millisecondsSinceEpoch;
        }
      ]
      // highlight-end
    )
    // highlight-end
  );
}
```
```dart
// title: Static method
final class User extends Model<User> {
  User(): super(
    hooks: HookModel(
      // highlight-start
      beforeSave: [setCreatedAt]
      // highlight-end
    )
  );
  
  // highlight-start
  static Future<void> setCreatedAt(Map<String, dynamic> user) async {
    user['created_at'] = DateTime.now().millisecondsSinceEpoch;
  }
  // highlight-end
}
```
:::

### BeforeSave
The `BeforeSave` hook lets you alter your model's data **before** its updating and persistence in the database.

:::codegroup
```dart
// title: Constructor
import 'package:crypto/crypto.dart';

final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: HookModel(
      beforeSave: [
        (Map<String, dynamic> user) {
          final bytes = utf8.encode(user.password);
          user['password'] = sha1.convert(bytes);
        }
      ]
    )
    // highlight-end
  );
}
```
```dart
// title: Static method
import 'package:crypto/crypto.dart';

final class User extends Model<User> {
  User(): super(
    hooks: HookModel(
      // highlight-start
      beforeSave: [hashPassword]
      // highlight-end
    )
  );

  // highlight-start
  static Future<void> hashPassword(Map<String, dynamic> user) async {
    final bytes = utf8.encode(user.password);
    user['password'] = sha1.convert(bytes);
  }
// highlight-end
}
```
:::

### BeforeDelete
The `BeforeDelete` hook lets you alter your model's data **before** its deletion from the database.
You have access to the query builder in order to perform additional actions.

:::codegroup
```dart
// title: Constructor
import 'package:crypto/crypto.dart';

final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: HookModel(
      beforeDelete: [
        (AbstractStandaloneQueryBuilder<User> query) {
          final lastDate = DateTime.now().millisecondsSinceEpoch - 3600;
          query.andWhere('last_connexion', '<=', lastDate);
        }
      ]
    )
    // highlight-end
  );
}
```
```dart
// title: Static method
import 'package:crypto/crypto.dart';

final class User extends Model<User> {
  User(): super(
    hooks: HookModel(
      // highlight-start
      beforeDelete: [deleteOldUsers]
      // highlight-end
    )
  );
  
  // highlight-start
  static Future<void> deleteOldUsers(query) {
    final lastDate = DateTime.now().millisecondsSinceEpoch - 3600;
    query.andWhere('last_connexion', '<=', lastDate);
  }
  // highlight-end
}
```
:::
