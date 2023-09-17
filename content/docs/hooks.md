# Hooks
Hooks are the actions that you can perform against a model instance during a pre-defined life cycle event. Using hooks, you can encapsulate specific actions within your models vs. writing them everywhere inside your codebase.

## Basic hook example

A great example of hooks is password hashing. You can define a hook that runs before the save call and converts the plain text password to a hash.
```dart
import 'package:crypto/crypto.dart';

final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.beforeSave(
      (Map<String, dynamic> user) {
        final bytes = utf8.encode(user.password);
        user['password'] = sha1.convert(bytes);
      }
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
  static void hashPassword(Map<String, dynamic> user) async {
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

### BeforeFetch
The `BeforeFetch` hook lets you alter your model's data **before** data recovery from the database.

:::codegroup
```dart
// title: Constructor
final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.beforeFetch(
      (AbstractStandaloneQueryBuilder<User> query) {
        query.andWhere('is_deleted', true);
      })
    // highlight-end
  );
}
```
```dart
// title: Static method
final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.beforeFetch(softRecovery)
    // highlight-end
  );
  
  // highlight-start
  static void softRecovery(AbstractStandaloneQueryBuilder<User> query) async {
    query.andWhere('is_deleted', true);
  }
  // highlight-end
}
```
:::

### AfterFetch
The `AfterFetch` hook lets you alter your model's data **after** data recovery from the database.

:::codegroup
```dart
// title: Constructor
final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.afterFetch(
      (List<User> users) {
        print('Users count: ${users.length}');
      })
    // highlight-end
  );
}
```
```dart
// title: Static method
final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.afterFetch(printUserCount)
    // highlight-end
  );
  
  // highlight-start
  static void printUserCount(List<User> users) async {
    print('Users count: ${query.length}');
  }
  // highlight-end
}
```
:::

### BeforeFind
The `BeforeFind` hook lets you alter your model's data **before** one element recovery from the database.

:::codegroup
```dart
// title: Constructor
final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.beforeFind(
      (AbstractStandaloneQueryBuilder<User> query) {
        query.andWhere('is_deleted', true);
      })
    // highlight-end
  );
}
```
```dart
// title: Static method
final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.beforeFind(softRecovery)
    // highlight-end
  );
  
  // highlight-start
  static void softRecovery(AbstractStandaloneQueryBuilder<User> query) async {
    query.andWhere('is_deleted', true);
  }
  // highlight-end
}
```
:::

### AfterFetch
The `AfterFetch` hook lets you alter your model's data **after** one element recovery from the database.

:::codegroup
```dart
// title: Constructor
final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.afterFind(
      (User users) {
        print('Current users: $user');
      })
    // highlight-end
  );
}
```
```dart
// title: Static method
final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.afterFind(printUser)
    // highlight-end
  );
  
  // highlight-start
  static void printUser(User user) async {
    print('Current users: $user');
  }
  // highlight-end
}
```
:::

### BeforeCreate
The `BeforeCreate` hook lets you alter your model's data **before** its insertion and persistence in the database.

:::codegroup
```dart
// title: Constructor
final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.beforeSave(
      (Map<String, dynamic> user) {
        user['created_at'] = DateTime.now().millisecondsSinceEpoch;
      })
    // highlight-end
  );
}
```
```dart
// title: Static method
final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.beforeSave(setCreatedAt)
    // highlight-end
  );
  
  // highlight-start
  static void setCreatedAt(Map<String, dynamic> user) async {
    user['created_at'] = DateTime.now().millisecondsSinceEpoch;
  }
  // highlight-end
}
```
:::

### AfterCreate
The `AfterCreate` hook lets you alter your model's data **after** its insertion and persistence in the database.

:::codegroup
```dart
// title: Constructor
final class User extends Model<User> {
  String? foo;
  
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.afterCreate(
      (User user) {
        foo = 'bar';
      })
    // highlight-end
  );
}
```
```dart
// title: Static method
final class User extends Model<User> {
  User(): super(
    // highlight-start
      hooks: (hooks) => hooks.afterCreate(setFoo)
    // highlight-end
  );
  
  // highlight-start
  static void setFoo(User user) async {
    foo = 'bar';
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
    hooks: (hooks) => hooks.beforeSave(
      (Map<String, dynamic> user) {
        final bytes = utf8.encode(user.password);
        user['password'] = sha1.convert(bytes);
      }
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
    // highlight-start
    hooks: (hooks) => hooks.beforeSave(hashPassword)
    // highlight-end
  );

  // highlight-start
  static void hashPassword(Map<String, dynamic> user) async {
    final bytes = utf8.encode(user.password);
    user['password'] = sha1.convert(bytes);
  }
// highlight-end
}
```
:::

### AfterSave
The `AfterSave` hook lets you alter your model's data **after** its updating and persistence in the database.

:::codegroup
```dart
// title: Constructor
import 'package:crypto/crypto.dart';

final class User extends Model<User> {
  User(): super(
    // highlight-start
    hooks: (hooks) => hooks.afterSave(
      (User user) {
        print('User ${user.id} has been updated');
      }
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
    // highlight-start
    hooks: (hooks) => hooks.afterSave(printWhenSave)
    // highlight-end
  );

  // highlight-start
  static void printWhenSave(User user) async {
    print('User ${user.id} has been updated');
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
      hooks: (hooks) => hooks.beforeDelete(
        (AbstractStandaloneQueryBuilder<User> query) {
          final lastDate = DateTime.now().millisecondsSinceEpoch - 3600;
          query.andWhere('last_connexion', '<=', lastDate);
        }
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
    // highlight-start
    hooks: (hooks) => hooks.beforeDelete(deleteOldUsers)
    // highlight-end
  );
  
  // highlight-start
  static void deleteOldUsers(query) {
    final lastDate = DateTime.now().millisecondsSinceEpoch - 3600;
    query.andWhere('last_connexion', '<=', lastDate);
  }
  // highlight-end
}
```
:::
