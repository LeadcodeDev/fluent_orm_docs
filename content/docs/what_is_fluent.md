# What is Fluent ORM ?

As a Typescript developer at heart, I've always been accustomed to coming across a multitude of packages in the Node.js ecosystem, each one more incredible than the last, to satisfy my every desire.

When I was learning the Dart language I was surprised to see, or rather not to have, a wide choice of packages fulfilling the role of an ORM. The only really interesting alternative is Prisma, an adaptation of the world-famous Prisma ORM for the Javascript language.

As a big fan of Adonis as I am, I didn't find the essence of the standards surrounding the database domain, such as migration management, enabling tables to be altered and evolved over time, or the concept of model, the representation of each table within its application.

It was with the aim of solidifying my knowledge in this field while offering a robust and reliable alternative to everyone that I designed Fluent, the model-based ORM for Dart.

```dart
import 'package:fluent_orm/fluent_orm.dart';

final class Article extends Model {
  Article(): super(properties: ['id', 'title', 'content'])
  
  int get id => get('id');
  String get title => get('title');
  String get content => get('content');
}

final articles = await Article.all();
```
