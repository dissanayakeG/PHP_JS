# Flutter Local Database with Drift and Riverpod

This guide shows a simple structure for a Flutter application that uses:

```text
Flutter UI
  → Riverpod providers
    → Repository
      → Drift database
        → SQLite
```

The UI should not contain database queries. The repository owns database access, while Riverpod creates and supplies shared objects.

## Riverpod service pattern in Dart

Riverpod is useful for wiring dependencies and exposing application operations to the UI. A service is a good fit when one user action coordinates a repository with another dependency, such as a file picker, API client, or notification gateway.

### 1. Wrap the app with `ProviderScope`

```dart
void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}
```

`ProviderScope` makes Riverpod providers available throughout the widget tree.

### 2. Define the service contract

```dart
abstract interface class CsvExportService {
  Future<CsvExportResult> export();
}
```

The interface describes the operation without exposing its implementation. This keeps widgets independent of how the export is performed.

### 3. Implement the service

```dart
class RepositoryCsvExportService implements CsvExportService {
  RepositoryCsvExportService({
    required this.repository,
  });

  final AppRepository repository;

  @override
  Future<CsvExportResult> export() async {
    // Read data from the repository.
    // Encode it as CSV.
    // Save or return the result.
  }
}
```

### 4. Register the service with Riverpod

```dart
final csvExportServiceProvider = Provider<CsvExportService>((ref) {
  return RepositoryCsvExportService(
    repository: ref.watch(appRepositoryProvider),
  );
});
```

The provider exposes the service through its interface while Riverpod creates it and supplies its dependencies. `ref.watch(...)` is appropriate while declaring dependencies because the service is recreated if a dependency changes.

### 5. Use the service from a widget

```dart
final result = await ref.read(csvExportServiceProvider).export();
```

Use `read` for one-time actions such as save, update, delete, import, or export. Use `watch` when the widget should rebuild when provider state changes.

```dart
ref.read(provider);   // obtain the current value for an action
ref.watch(provider);  // listen and rebuild when it changes
```

## Using Drift for a Local Database in Dart

Drift is a type-safe SQLite library for Dart and Flutter. Tables are defined in Dart, queries are checked and generated as Dart code, and the database can be replaced with an in-memory executor in tests.

### 1. Define database tables

```dart
class Categories extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().unique()();
}
```

Drift generates a row class named `Category` from this definition. `autoIncrement()` creates a generated primary key, while `unique()` prevents duplicate category names.

Common column definitions include:

```dart
integer()                         // required integer
text()                            // required text
text().nullable()                 // nullable text
integer().autoIncrement()         // generated primary key
dateTime().withDefault(...)       // database default value
```

### 2. Create the database class

```dart
part 'app_database.g.dart';

@DriftDatabase(tables: [Categories, Items])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor])
      : super(executor ?? driftDatabase(name: 'my_app'));

  @override
  int get schemaVersion => 1;
}
```

`@DriftDatabase` registers the tables, while `_$AppDatabase` is the generated base class. The optional `QueryExecutor` makes the database easy to replace in tests.

Generate the supporting code after changing tables or the database class:

```bash
dart run build_runner build --delete-conflicting-outputs
```

Never edit `app_database.g.dart` manually.

### 3. Keep queries in a repository

```dart
class DriftAppRepository implements AppRepository {
  DriftAppRepository({required AppDatabase database})
      : _database = database;

  final AppDatabase _database;
}
```

The repository is the application-facing data API. It owns Drift queries so widgets do not depend on tables, SQL, or database lifecycle details.

### 4. Read and watch data

```dart
final query = _database.select(_database.categories)
  ..orderBy([
    (table) => OrderingTerm.asc(table.name),
  ]);

final categories = await query.get(); // one-time read
final categoryStream = query.watch(); // continuously updated stream
```

Use `where` to filter a query:

```dart
final query = _database.select(_database.items)
  ..where((table) => table.categoryId.equals(categoryId));
```

`get()` returns a `Future`; `watch()` returns a `Stream` that emits when the relevant data changes.

### 5. Write data and use transactions

```dart
final id = await _database.into(_database.categories).insert(
      CategoriesCompanion.insert(name: normalizedName),
    );

await (_database.update(_database.categories)
      ..where((table) => table.id.equals(id)))
    .write(CategoriesCompanion(name: Value(newName)));

await (_database.delete(_database.categories)
      ..where((table) => table.id.equals(id)))
    .go();
```

Use `Value(...)` when constructing a companion for an update or nullable field. Use a transaction when related writes must all succeed or all roll back:

```dart
await _database.transaction(() async {
  final category = await createOrGetCategory(name: categoryName);
  await createList(categoryId: category.id, name: listName);
});
```

## 1. Project setup

Create a Flutter project:

```bash
flutter create my_app
cd my_app
```

Add the packages:

```bash
flutter pub add drift drift_flutter flutter_riverpod
flutter pub add --dev drift_dev build_runner sqlite3
```

A useful structure is:

```text
lib/
  data/
    local/
      tables.dart
      app_database.dart
    repository/
      app_repository.dart
      repository_providers.dart
  ui/
    dashboard/
      dashboard_page.dart
    add_item/
      add_item_page.dart
test/
  repository/
    app_repository_test.dart
```

## 2. Define Drift tables

File: `lib/data/local/tables.dart`

```dart
import 'package:drift/drift.dart';

class Items extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  RealColumn get price => real()();
  DateTimeColumn get createdAt =>
      dateTime().withDefault(currentDateAndTime)();
}
```

`Items` is the table definition. Drift generates a row class named `Item` from it.

Common column definitions:

```dart
integer()                         // integer column
text()                            // required text column
text().nullable()                 // nullable text column
integer().autoIncrement()        // generated primary key
text().unique()                   // no duplicate values
dateTime().withDefault(...)       // database default value
```

A foreign-key relationship can be defined like this:

```dart
IntColumn get categoryId =>
    integer().references(Categories, #id);
```

## 3. Create the Drift database

File: `lib/data/local/app_database.dart`

```dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

import 'tables.dart';

part 'app_database.g.dart';

@DriftDatabase(tables: [Items])
class AppDatabase extends _$AppDatabase {
  AppDatabase([QueryExecutor? executor])
      : super(executor ?? driftDatabase(name: 'my_app'));

  @override
  int get schemaVersion => 1;

  @override
  MigrationStrategy get migration => MigrationStrategy(
        onCreate: (migrator) async {
          await migrator.createAll();
        },
      );
}
```

What these parts mean:

- `@DriftDatabase` registers the tables.
- `_$AppDatabase` is a generated base class.
- `QueryExecutor` allows a custom database, which is useful for tests.
- `schemaVersion` identifies the database schema version.
- `onCreate` creates tables for a new database.
- `app_database.g.dart` is generated code and must not be edited manually.

Generate Drift code after changing tables or the database class:

```bash
dart run build_runner build --delete-conflicting-outputs
```

## 4. Create the repository interface

File: `lib/data/repository/app_repository.dart`

The repository is the app-facing data API. The UI calls repository methods instead of writing Drift queries directly.

```dart
import '../local/app_database.dart';

abstract interface class AppRepository {
  Stream<List<Item>> watchItems();
  Future<List<Item>> getItems();
  Future<Item> createItem({
    required String name,
    required double price,
  });
}
```

The interface is a contract. It says what the application can do, without saying how the data is stored.

## 5. Implement the repository with Drift

File: `lib/data/repository/app_repository.dart`

```dart
import 'package:drift/drift.dart';

import '../local/app_database.dart';
import 'app_repository.dart';

class DriftAppRepository implements AppRepository {
  DriftAppRepository({required AppDatabase database})
      : _database = database;

  final AppDatabase _database;

  @override
  Stream<List<Item>> watchItems() {
    final query = _database.select(_database.items)
      ..orderBy([
        (table) => OrderingTerm.asc(table.name),
      ]);
    return query.watch();
  }

  @override
  Future<List<Item>> getItems() {
    final query = _database.select(_database.items)
      ..orderBy([
        (table) => OrderingTerm.asc(table.name),
      ]);
    return query.get();
  }

  @override
  Future<Item> createItem({
    required String name,
    required double price,
  }) async {
    final id = await _database.into(_database.items).insert(
          ItemsCompanion.insert(
            name: name,
            price: price,
          ),
        );

    return (_database.select(_database.items)
          ..where((table) => table.id.equals(id)))
        .getSingle();
  }
}
```

### Understanding `_database`

```dart
final AppDatabase _database;
```

`_database` is a private field containing the `AppDatabase` instance. The underscore is Dart's convention for a private library member.

### Reading data

```dart
_database.select(_database.items)
```

Creates a typed select query.

```dart
query.get()
```

Executes the query once and returns a `Future`.

```dart
query.watch()
```

Returns a `Stream` that emits new results when the table changes.

### Filtering data

```dart
final query = _database.select(_database.items)
  ..where((table) => table.id.equals(id));
```

The `..` is Dart's cascade operator. It configures the query and then keeps the same query object.

The condition is similar to:

```sql
SELECT * FROM items WHERE id = ?;
```

### Inserting data

```dart
final id = await _database.into(_database.items).insert(
      ItemsCompanion.insert(
        name: name,
        price: price,
      ),
    );
```

`into` chooses the table. `ItemsCompanion.insert` contains the values to insert. The returned value is the generated row ID.

For nullable fields, use `Value` when constructing a companion:

```dart
ItemsCompanion(
  name: Value(name),
)
```

### Updating data

```dart
final changedRows = await (_database.update(_database.items)
      ..where((table) => table.id.equals(id)))
    .write(
      ItemsCompanion(
        name: Value(newName),
        price: Value(newPrice),
      ),
    );
```

`changedRows` tells how many rows were updated.

### Deleting data

```dart
final deletedRows = await (_database.delete(_database.items)
      ..where((table) => table.id.equals(id)))
    .go();
```

`go()` executes the delete query and returns the number of deleted rows.

## 6. Use transactions for related operations

```dart
return _database.transaction(() async {
  final category = await createOrGetCategory(name: categoryName);

  return createList(
    categoryId: category.id,
    name: listName,
  );
});
```

A transaction groups operations into one unit:

```text
begin transaction
  ├── find or create category
  └── create list
commit if successful
rollback if an error occurs
```

If an operation throws an exception, Drift rolls back the transaction. This prevents partially saved data.

Use a transaction when multiple writes must all succeed or all fail.

## 7. Connect Drift to Riverpod

File: `lib/data/repository/repository_providers.dart`

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../local/app_database.dart';
import 'app_repository.dart';

final appDatabaseProvider = Provider<AppDatabase>((ref) {
  final database = AppDatabase();
  ref.onDispose(database.close);
  return database;
});

final appRepositoryProvider = Provider<AppRepository>((ref) {
  return DriftAppRepository(
    database: ref.watch(appDatabaseProvider),
  );
});

final itemStreamProvider = StreamProvider<List<Item>>((ref) {
  return ref.watch(appRepositoryProvider).watchItems();
});
```

`Provider<T>` supplies a shared object of type `T`.

`StreamProvider<T>` exposes a stream and gives the UI loading, data, and error states.

The dependency chain is:

```text
itemStreamProvider
  → appRepositoryProvider
    → appDatabaseProvider
      → AppDatabase
```

## 8. Enable Riverpod in the app

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}
```

`ProviderScope` makes Riverpod providers available to the widget tree.

Without it, widgets cannot read or watch providers.

## 9. Read live data in a widget

```dart
class DashboardPage extends ConsumerWidget {
  const DashboardPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final itemsAsync = ref.watch(itemStreamProvider);

    return itemsAsync.when(
      data: (items) {
        if (items.isEmpty) {
          return const Center(child: Text('No items yet.'));
        }

        return ListView.builder(
          itemCount: items.length,
          itemBuilder: (context, index) {
            return ListTile(
              title: Text(items[index].name),
            );
          },
        );
      },
      loading: () => const Center(
        child: CircularProgressIndicator(),
      ),
      error: (error, stackTrace) => Center(
        child: Text('Error: $error'),
      ),
    );
  }
}
```

`ref.watch(itemStreamProvider)` listens for changes. When Drift emits a new list, Riverpod rebuilds the widget.

`.when` chooses the correct UI for the asynchronous state:

```text
loading → progress indicator
data    → list of items
error   → error message
```

## 10. Save data from a widget

Use `ConsumerWidget` or `ConsumerState` when a widget needs `ref`:

```dart
Future<void> saveItem(WidgetRef ref) async {
  await ref.read(appRepositoryProvider).createItem(
        name: 'Milk',
        price: 4.25,
      );
}
```

Use `read` for actions such as save, update, delete, import, or export. The action does not need to rebuild the widget when the provider changes.

Use `watch` when the widget should rebuild as the provider changes.

```dart
ref.read(provider);   // get the current object
ref.watch(provider);  // listen and rebuild when it changes
```

## 11. Database migrations

Suppose a version-1 application has `id`, `name`, and `price`, and version 2 adds an optional note. Add the new column to the table:

```dart
TextColumn get note => text().nullable()();
```

Then increase the schema version:

```dart
@override
int get schemaVersion => 2;
```

Add upgrade logic:

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
      onCreate: (migrator) async {
        await migrator.createAll();
      },
      onUpgrade: (migrator, from, to) async {
        if (from < 2) {
          await migrator.addColumn(items, items.note);
        }
      },
    );
```

After changing the schema:

```bash
dart run build_runner build --delete-conflicting-outputs
```

Do not delete or recreate the production database to handle a normal schema change. Use a migration so existing user data is preserved.

## 12. Test the repository

Use an in-memory SQLite database in tests:

```dart
import 'package:drift/native.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:my_app/data/local/app_database.dart';
import 'package:my_app/data/repository/app_repository.dart';

void main() {
  late AppDatabase database;
  late AppRepository repository;

  setUp(() {
    database = AppDatabase(NativeDatabase.memory());
    repository = DriftAppRepository(database: database);
  });

  tearDown(() async {
    await database.close();
  });

  test('creates an item', () async {
    final item = await repository.createItem(
      name: 'Milk',
      price: 4.25,
    );

    expect(item.name, 'Milk');
  });
}
```

Repository tests verify database behavior without starting the whole application.

## 13. Development and deployment commands

### Useful Commands

Run these from the Flutter project root:

```bash
flutter pub get
dart run build_runner build --delete-conflicting-outputs
flutter analyze
flutter test
flutter run
```

Use `build_runner` after changing Drift tables or database code. Use `flutter analyze` and `flutter test` before creating a release build.

### Build / Run for Specific Devices

List available targets, then pass a device ID with `-d`:

```bash
flutter devices
flutter run -d <device-id>
```

Examples:

```bash
flutter run -d linux
flutter run -d chrome
flutter run -d emulator-5554
```

### Builds

```bash
flutter build apk --release
flutter build appbundle --release
flutter build linux --release
flutter build web --release
```

The Android APK is written under `build/app/outputs/flutter-apk/`. Linux output is written under `build/linux/x64/release/bundle/`, and the web output is written under `build/web/`.

### Enable Desktop/Web Targets

Enable only the platforms required by the project:

```bash
flutter config --enable-linux-desktop
flutter config --enable-web
flutter create --platforms=linux,web .
```

Check the resulting targets with:

```bash
flutter devices
```

`flutter create --platforms=... .` adds platform files to an existing project. Review the generated files before committing them.

### Common Clean Build Flow

Use this sequence when stale generated files or build artifacts cause confusing errors:

```bash
flutter clean
flutter pub get
dart run build_runner build --delete-conflicting-outputs
flutter analyze
flutter test
```

If the problem is limited to generated Drift files, try the build-runner command first; a full clean is slower and is not normally needed after every code change.

### Build linux image and add desktop entry + Create terminal command

Build the release bundle:

```bash
flutter build linux --release
```

Create a desktop entry at `~/.local/share/applications/my_app.desktop`, adjusting the paths and executable name to match the project:

```ini
[Desktop Entry]
Name=My App
Comment=My Flutter application
Exec=/absolute/path/to/project/build/linux/x64/release/bundle/my_app
Icon=/absolute/path/to/project/assets/icons/my_app.png
Terminal=false
Type=Application
Categories=Utility;
```

Make the launcher executable and refresh the desktop-entry cache when available:

```bash
chmod +x ~/.local/share/applications/my_app.desktop
update-desktop-database ~/.local/share/applications
```

To create a terminal command without installing system-wide, add a symlink in `~/.local/bin`:

```bash
mkdir -p ~/.local/bin
ln -sfn /absolute/path/to/project/build/linux/x64/release/bundle/my_app ~/.local/bin/my-app
```

Ensure `~/.local/bin` is on `PATH`, then launch the app with:

```bash
my-app
```

### Build Android APK and Install on Physical Device

Enable USB debugging on the Android device, connect it by USB, unlock it, and accept the debugging prompt. Verify that Flutter can see it:

```bash
flutter devices
adb devices
```

Build and install a release APK directly:

```bash
flutter build apk --release
flutter install -d <device-id>
```

Alternatively, install the generated APK with ADB:

```bash
adb -s <device-id> install -r build/app/outputs/flutter-apk/app-release.apk
```

For development, use `flutter run -d <device-id>` instead; it builds, installs, and starts the app in one command.

### Typical Development Flow

```text
define or change a table
→ update schemaVersion and migration if necessary
→ generate Drift code
→ add repository methods
→ expose shared objects or streams with Riverpod
→ use providers in widgets
→ run analyzer and tests
```

## Important rules

- Do not edit `app_database.g.dart` manually.
- Keep Drift queries inside the repository layer.
- Use `watch()` and `StreamProvider` for live UI data.
- Use `read()` for one-time actions.
- Use transactions for related writes.
- Add migrations when changing a database used by existing users.
- Use interfaces when a repository or service may need a different implementation in tests or in the future.
