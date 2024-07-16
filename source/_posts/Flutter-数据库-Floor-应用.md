---
title: Flutter 数据库 Floor 应用
date: 2024-05-16 15:26:46
categories: "Flutter"
tags:
  - Flutter
---

## 前言
### 数据库分类
云数据库与边缘数据库，以及嵌入式数据库与内存数据库。然而，数据库可以通过附加标准进一步区分，例如它们支持的数据类型或扩展方式以及定义可能会有所不同。

请问:
移动端的存储数据库属于以上哪种数据库？
sql是哪家公司开发推出的？

非关系型数据库 (NoSQL) 与关系型数据库 (SQL)
SQL (Structured Query Language) 数据库:

-  优点:
   - 数据一致性：SQL 数据库采用 ACID（原子性、一致性、隔离性、持久性）事务属性，确保数据的一致性。
   - 强大的查询语言：SQL 提供了灵活而强大的查询语言，可以进行复杂的数据查询和分析。
   - 数据模型稳定：SQL 数据库使用表格结构，适合于需要严格定义数据结构的应用程序。
- 缺点:
   - 扩展性有限：SQL 数据库在处理大规模数据时可能会遇到扩展性问题。
   - 数据结构变更困难：对于频繁变更数据结构的应用程序，SQL 数据库可能需要进行复杂的迁移操作。

NoSQL (Not Only SQL) 数据库:

- 优点:
   - 高扩展性：NoSQL 数据库可以更好地处理大规模数据和高并发访问。
   - 灵活的数据模型：NoSQL 数据库支持灵活的数据模型，适合于需要频繁变更数据结构的应用程序。
   - 高性能：在某些情况下，NoSQL 数据库可以提供更高的性能和吞吐量。
- 缺点:
   - 缺乏标准化：NoSQL 数据库缺乏统一的标准，不同类型的 NoSQL 数据库之间的语法和功能可能有所不同。
   - 数据一致性挑战：某些 NoSQL 数据库可能牺牲了一致性以换取更高的性能，可能会出现数据一致性方面的挑战。
### 什么是 ORM？ 
对象关系映射器 (ORM) 不是数据库。我们主要提出这个问题，因为我们经常看到它令人困惑。它是位于数据库之上的一层，使其更易于使用。当数据库是关系数据库 (SQL) 并且使用的编程语言是面向对象时，这一点通常尤其重要。如上所述，Dart 是一种面向对象的编程语言。

## Flutter 常用数据库插件

| 分类 | 名称 | 描述 | 优点 | 缺点 |
| --- | --- | --- | --- | --- |
| sql | sqflite | 是一个轻量级的 SQLite 数据库插件，适用于简单的本地数据库操作。它提供了基本的 SQLite 功能，适合小型应用或者对数据库操作要求不高的场景。 | 1. 支持事务和批处理<br>2. 数据库操作在后台线程中执行 | 1. 在某些情况下，性能可能受到影响，特别是在复杂查询或大规模数据操作时<br>2. 需要手动编写SQL查询语句，不如一些ORM（对象关系映射）库提供的便利 |
| sql | Drift | 是功能最丰富的 Flutter 关系数据库解决方案。它也是 sqflite 的包装器，但是它提供了更强大的功能，并且包含一个查询 API。尽管 Web 支持是实验性的，但它支持所有潜在的平台。此外，它还为事务、模式迁移、复杂的筛选器和表达式以及批处理提供了极好的支持。 | 1. Drift 具有对事务、模式迁移、复杂过滤器和表达式、批量更新和联接的内置支持<br>2. 提供了灵活且强大的查询功能，您可以轻松地执行各种查询操作，如筛选、排序、连接等 | 1. 需要学习的DSL（领域特定语言），可能对新手有一定的学习曲线，不如一些ORM 库提供的高级功能 |
| sql | Floor | 为您的Flutter应用程序提供了一个整洁的 SQLite 抽象，灵感来自 Room 持久化库。它可以在内存对象和数据库行之间进行自动映射，同时还可以通过SQL完全控制数据库，还提供了观察数据、事务处理和数据库迁移等功能。 | 1. 支持事务，使用了一些性能优化技巧，例如批量插入等，可以提升数据库操作的效率<br>2. 支持 ORM<br>3. 可以在 Flutter 应用中方便地处理数据库操作而不会阻塞主线程 | 1. 由于 Floor 是基于 SQLite 的 ORM 库，因此在某些情况下可能无法满足复杂的数据库操作需求 |
| nosql | realm | 是一个可以直接在手机、平板、电脑或可穿戴设备中运行的本地数据库。Realm是一个 KeyValue 的本地数据库，相比SQLite 和 Firestore 更简单并且运行速度更快，是SQLite 和 Firestore 的替代方案。 Realm 是离线的、面向对象的、直观的，跨平台的本地存储数据库。 | 1. 数据库具有出色的性能，读写速度快，适合处理大量数据<br>2. 面向对象模型，提供了简洁的API，易于学习和使用<br>3. 基于 mongodb atlas 云储存支持多设备同步（收费） | 1. Realm的功能相对于一些传统的关系型数据库可能会有一些限制，如复杂的查询操作可能不如SQL数据库灵活 |
| nosql | hive | Hive 是一个轻量级、快速的 NoSQL 数据库。它具有高性能、低延迟和简单易用的特点。Hive 使用键值对的形式来存储数据，支持自定义类型的序列化和反序列化，同时还提供了强大的查询功能和数据观察机制。 | 1. 轻量级、快速、简单的NoSQL数据库，适用于存储简单的键值对数据<br>2. 支持自定义对象序列化，性能较高<br>3. 支持数据加密 | 1. 不支持复杂的查询操作，适用于简单的数据存储场景<br>2. 不支持事务特性 |
| nosql | isar | Isar 是一个高性能、类型安全的嵌入式 NoSQL 数据库。Isar 允许您在 Flutter 应用程序中存储和检索对象，同时提供了强大的查询功能和数据模型定义。它专为 Flutter 开发者设计，可以轻松集成到您的应用程序中，并且在处理大量数据时表现优异。Isar 还支持事务、索引、观察者模式等功能，使得在 Flutter 应用程序中管理本地数据变得更加简单和高效 | 1. 高度可扩展<br>2. 功能丰富。复合和多条目索引、查询修饰符、JSON 支持等<br>3. 异步。默认情况下并行查询操作和多重隔离支持 | 1. isar 相对较新，可能在某些方面缺乏一些成熟的功能和生态支持<br>2. 对于一些开发者来说，NoSQL 数据库模型可能需要适应和学习成本 |


## 选择 floor 原因

-  早期研究 isar 插件
   - 发现：混合原生下发现 iPhone 13 正常，但 iPhone XR 设备 crash 问题
   - 反馈作者 [传送门](https://github.com/isar/isar/issues/915#issuecomment-1338948317) & demo 复现 [传送门](https://github.com/zeqinjie/dart_native_demo)
- Floor 
   - 优势（简单）
      - 是基于 sqflite 插件二次封装 sql 数据库，参考谷歌 Room 引入 Dao 设计风格
      - 提供了简单易用的 API，使得定义实体、执行查询等操作更加直观和便捷。
      - 支持数据观察、事务处理和数据库迁移，有助于简化数据库操作和维护
   - 不足
      - 扩展性有限：SQL 数据库在处理大规模数据时可能会遇到扩展性问题。
      - 数据结构变更困难：对于频繁变更数据结构的应用程序，SQL 数据库可能需要进行复杂的迁移操作
## 使用 [文档](https://pinchbv.github.io/floor/getting-started/)

### 层级关系

![](/images/2024/Flutter-数据库-Floor-应用/1.png)

### 引入插件

```json
dependencies:
  flutter:
    sdk: flutter
  floor: ^1.4.0

dev_dependencies:
  floor_generator: ^1.4.0
  build_runner: ^2.1.2
```

### 定义表结构对象

```dart
@entity
class TWFloorBook implements TWDbBook {
  /* 主键 */
  @PrimaryKey(autoGenerate: true)
  int? primaryId;

  @override
  String? name;

  @override
  String? author;

  @override
  String? description;

  TWFloorBook({
    this.name,
    this.author,
    this.description,
  });

}
```

### 定义 Dao 接口

```dart
@dao
abstract class TWFloorBookDao {
  /// 查询所有
  @Query('SELECT * FROM TWFloorBook')
  Future<List<TWFloorBook>> findModels();

  /// 根据名称查询
  @Query('SELECT * FROM TWFloorBook WHERE name = :name')
  Future<List<TWFloorBook>> findModelByName(
    String name,
  );

  /// 根据作者查询
  @Query('SELECT * FROM TWFloorBook WHERE author = :author')
  Future<List<TWFloorBook>> findModelByAuthor(
    String author,
  );

  /// 根据名称和作者查询
  @Query('SELECT * FROM TWFloorBook WHERE name = :name AND author = :author')
  Future<List<TWFloorBook>> findModelByNameAndAuthor(
    String name,
    String author,
  );

  /// 根据年份查询
  @Query('SELECT * FROM TWFloorBook WHERE year = :year')
  Future<List<TWFloorBook>> findModelByYear(
    String year,
  );

  /// 删除
  @Query('DELETE FROM TWFloorBook')
  Future<int?> deleteModels();

  /// 根据名称删除
  @Query('DELETE FROM TWFloorBook WHERE name = :name')
  Future<int?> deleteModelByName(String name);

  /// 根据作者删除
  @Query('DELETE FROM TWFloorBook WHERE author = :author')
  Future<int?> deleteModelByAuthor(String author);

  /// 根据名称和作者删除
  @Query('DELETE FROM TWFloorBook WHERE name = :name AND author = :author')
  Future<int?> deleteModelByNameAndAuthor(
    String name,
    String author,
  );

  /// 删除
  @delete
  Future<int> deleteModel(
    TWFloorBook model,
  );

  /// 插入
  @insert
  Future<int> insertModel(
    TWFloorBook model,
  );

  /// 批量插入
  @insert
  Future<List<int>> insertModels(
    List<TWFloorBook> models,
  );

  /* 更新 */
  @update
  Future<int?> updateModel(
    TWFloorBook model,
  );

  /// 批量更新
  @update
  Future<int?> updateModels(
    List<TWFloorBook> models,
  );
}
```

### 定义数据库版本及对应表结构映射

```dart
class TWDateTimeConverter extends TypeConverter<DateTime?, int> {
  @override
  DateTime? decode(int databaseValue) {
    return DateTime.fromMillisecondsSinceEpoch(databaseValue);
  }

  @override
  int encode(DateTime? value) {
    if (value == null) {
      return DateTime.now().millisecondsSinceEpoch;
    }
    return value.millisecondsSinceEpoch;
  }
}

/// 数据库，版本迁移需要修改版本号
@Database(version: 1, entities: [
  TWFloorBook,
])
@TypeConverters([TWDateTimeConverter])
abstract class TWFloorDatabase extends FloorDatabase {
  TWFloorBookDao get bookDao;
}

static Future<TWFloorDatabase> database() async {
  final callback = Callback(
    onCreate: (database, version) {
      TWLog('TWFloorDbHelper onCreate: $version');
    },
    onOpen: (database) {
      TWLog('TWFloorDbHelper onOpen');
    },
    onUpgrade: (database, startVersion, endVersion) {
      TWLog('TWFloorDbHelper onUpgrade: $startVersion -> $endVersion');
    },
  );
  if (TWFloorDbHelper()._database != null) {
    return TWFloorDbHelper()._database!;
  }
  try {
    final database = await $FloorTWFloorDatabase
        .databaseBuilder('tw_floor_database.db')
        .addCallback(callback)
    .build();
    TWFloorDbHelper()._database = database;
    return database;
  } catch (e) {
    TWLog('TWFloorDbHelper database error: $e');
    rethrow;
  }
}
```

### Db 版本迁移

比如新增一个属性 `year` 如下
```dart
final migration1to2 = Migration(1, 2, (database) async {
  await database.execute('ALTER TABLE TWFloorBook ADD COLUMN year TEXT');
});
final database = await $FloorTWFloorDatabase
    .databaseBuilder('tw_floor_database.db')
        .addMigrations(
      [
        migration1to2,
      ],
    )
    .build();
  TWFloorDbHelper()._database = database;
  return database;
} catch (e) {
  TWLog('TWFloorDbHelper database error: $e');
  rethrow;
}
```

### 其他能力

- 支持虚拟表，注意虚拟表仅支持只读不支持增、删、改。使用 @DatabaseView 注解定义 [传送门](https://pinchbv.github.io/floor/database-views/)
- 支持 Dao 支持 @Query ，@insert，@delete，@update 能力 [传送门](https://pinchbv.github.io/floor/daos/)
- 支持 Stream , 使用 @Query 是不支持 Stream
- 支持 @transaction 事务操作，以保证数据安全
   - 相关策略
   - 枚举 OnConflictStrategy 包含以下几种策略：
      - replace：替换旧数据并继续事务
      - rollback：回滚事务
      - abort：中止事务
      - fail：事务失败
      - ignore：忽略冲突
- 类型转换 @TypeConverters，目前任然处于测试阶段 [传送门](https://pinchbv.github.io/floor/type-converters/)
- 支持内存数据库 final database = await $FloorAppDatabase.inMemoryDatabaseBuilder().build();

## 注意事项

- 继承或者实现接口是不支持重写属性，所以我们可以通过 @ignore 忽略父类的同名变量
- 注意表对应的 model 新增字段，如果不新增到表里，记得 @ignore 否则被更新到表结构
- 使用 @Query 是不支持 Stream
- 版本迁移记得调整目标数据库版本
- 其他：使用 `flutter packages pub run build_runner build --delete-conflicting-outputs` 命令生成代码时记得将 example 移除文件夹，原因是 reaml ~
- 低版本可以升级高版本，但高版本不能再安装低版本了

## 参考

- [Flutter 5 大本地数据库解决方案 - 掘金](https://juejin.cn/post/7170706225410080776?searchId=20240216114201D950F220DACCDE2FEDBD)
- [SQLite vs Realm: Which Database to Choose in 2024 | Orangesoft](https://orangesoft.co/blog/realm-vs-sqlite)
- [Flutter floor数据库类orm插件使用详解 - 掘金](https://juejin.cn/post/7101947846118604807)
- [Top 10 Flutter Database, that you should know about it](https://shirsh94.medium.com/top-10-flutter-database-that-you-should-know-about-it-27a350bc7f40)
- [Flutter databases – hive, ObjectBox, sqflite](https://objectbox.io/flutter-databases-sqflite-hive-objectbox-and-moor/)
