# Flutter Local Database with Drift and Riverpod Project Setup Reference


## 1. Create Flutter Project

```bash
flutter create my_app
cd my_app
```

Run once to confirm the starter app works:

```bash
flutter run
```

## 2. Add Packages

For local SQLite database with Drift:

```bash
flutter pub add drift drift_flutter
flutter pub add flutter_riverpod go_router
flutter pub add --dev drift_dev build_runner sqlite3
```

Then fetch packages:

```bash
flutter pub get
```

Package purposes:

- `drift` - database ORM/query layer
- `drift_flutter` - Flutter SQLite setup helper
- `flutter_riverpod` - app state management/providers
- `go_router` - navigation/routing
- `drift_dev` - Drift code generator
- `build_runner` - runs code generation
- `sqlite3` - SQLite support for tests/tools

## 3. Create Below Folder Structure

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
    add_entry/
      add_entry_page.dart

test/
  repository/
    app_repository_test.dart
```

## 4. Define Tables

File:

```text
lib/data/local/tables.dart
```

Example:

```dart
import 'package:drift/drift.dart';

class Items extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  RealColumn get price => real()();
}
```

## 5. Create App Database

File:

```text
lib/data/local/app_database.dart
```

Example:

```dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';

import 'tables.dart';

part 'app_database.g.dart';

@DriftDatabase(tables: [Items])
class AppDatabase extends _$AppDatabase {
  
  AppDatabase([QueryExecutor? executor]) : super(executor ?? driftDatabase(name: 'app_database'));

  @override
  int get schemaVersion => 1;

  @override
  MigrationStrategy get migration => MigrationStrategy(
        onCreate: (migrator) async => migrator.createAll(),
      );
}
```

## 6. Generate Drift Code

Run after creating or changing database/table files:

```bash
dart run build_runner build
```

If generated file conflicts happen:

```bash
dart run build_runner build --delete-conflicting-outputs
```

This generates:

```text
lib/data/local/app_database.g.dart
```

Do not manually edit generated files.

## 7. Create Repository

File:

```text
lib/data/repository/app_repository.dart
```

The repository is the app-facing data API.

Purpose:

```text
UI talks to Repository
Repository talks to Drift database
Drift talks to SQLite
```

Example:

```dart
import 'package:drift/drift.dart';

import '../local/app_database.dart';

abstract interface class AppRepository {
  Stream<List<Item>> watchItems();

  Future<List<Item>> getAllItems();

  Future<Item> createItem({required String name, required double price});

  // Future<void> updateItem({required Item item});

  // Future<void> deleteItem({required Item item});

  // Future<Item> getItemById({required int id});
}

class DriftAppRepository implements AppRepository {
  DriftAppRepository({required this._database});

  final AppDatabase _database;

  @override
  Stream<List<Item>> watchItems() {
    final query = _database.select(_database.items)
      ..orderBy([(t) => OrderingTerm.asc(t.name)]);
    return query.watch();
  }

  @override
  Future<List<Item>> getAllItems() {
    final query = _database.select(_database.items)
      ..orderBy([(t) => OrderingTerm.asc(t.name)]);
    return query.get();
  }

  @override
  Future<Item> createItem({
    required String name,
    required double price,
  }) async {
    final id = await _database
        .into(_database.items)
        .insert(
          ItemsCompanion.insert(
            name: name,
            price: price,
          ),
        );

    return (_database.select(
      _database.items,
    )..where((t) => t.id.equals(id))).getSingle();
  }
}
```

## 8. Create Riverpod Providers

File:

```text
lib/data/repository/repository_providers.dart
```
//this is not meant to duplicate every method from AppRepository

//repository_providers.dart should contain providers for things the app watches or shares, not copies of every repository method.


Example:

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../local/app_database.dart';
import 'app_repository.dart';

final appDatabaseProvider = Provider<AppDatabase>((ref) {
  final database = AppDatabase();
  ref.onDispose(() => database.close());
  return database;
});

final appRepositoryProvider = Provider<AppRepository>((ref) {
  return DriftAppRepository(database: ref.watch(appDatabaseProvider));
});


final itemStreamProvider = StreamProvider<List<Item>>((ref) {
  return ref.watch(appRepositoryProvider).watchItems();
});

//For StreamProviders, It makes sense having their own provider because Riverpod can manage loading/data/error states and rebuild the UI when Drift emits new item lists.

//For one-time actions like createItem() or getAllItems(), you usually call them through the repository provider:
```

Once you expose appRepositoryProvider, the UI or controllers can already call any method from AppRepository:

```dart
await ref.read(appRepositoryProvider).createItem(
  name: name,
  price: price,
);
```

- We added itemStreamProvider because watchItems() is a stream that the UI will commonly watch:

```dart
final itemsAsync = ref.watch(itemStreamProvider);
```

Provider behavior:

- `Provider` creates shared objects like database/repository.
- `StreamProvider` exposes live database data to the UI.
- `ref.watch(...)` listens and rebuilds UI when data changes.
- `ref.read(...)` is used for actions like save/update/delete.

## 9. Enable Riverpod in main.dart

File:

```text
lib/main.dart
```

Example:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import 'ui/dashboard/dashboard_page.dart';

void main() {
  runApp(const ProviderScope(child: MyApp()));
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Database App',
      home: const DashboardPage(),
    );
  }
}
```

Without `ProviderScope`, Riverpod providers will not work.

## 10. Read Data in UI

File:

```text
lib/ui/dashboard/dashboard_page.dart
```

Example:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../data/repository/repository_providers.dart';

class DashboardPage extends ConsumerWidget {
  const DashboardPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final itemStream = ref.watch(itemStreamProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Entries')),

      floatingActionButton: FloatingActionButton(
        onPressed: () => context.push('/add-entry'),
        child: const Icon(Icons.add),
      ),
      
      body: itemStream.when(
        data: (items) {
          if (items.isEmpty) {
            return const Center(child: Text('No entries yet.'));
          }

          return ListView.builder(
            itemCount: items.length,
            itemBuilder: (context, index) {
              final entry = items[index];
              return ListTile(title: Text(entry.name));
            },
          );
        },
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, stackTrace) => Center(child: Text('Error: $error')),
      ),
    );
  }
}
```

## 11. Save Data from UI

File:

```text
lib/ui/add_entry/add_entry_page.dart
```

Example:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../data/repository/repository_providers.dart';

class AddEntryPage extends ConsumerStatefulWidget {
  const AddEntryPage({super.key});

  @override
  ConsumerState<AddEntryPage> createState() => _AddEntryPageState();
}

class _AddEntryPageState extends ConsumerState<AddEntryPage> {
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _priceController = TextEditingController();

  @override
  void dispose() {
    _nameController.dispose();
    _priceController.dispose();
    super.dispose();
  }

  Future<void> _saveEntry() async {
    final name = _nameController.text;
    final price = double.tryParse(_priceController.text) ?? 0.0;
    if (name.isNotEmpty) {
      await ref
          .read(appRepositoryProvider)
          .createItem(name: name, price: price);
      if (!mounted) return;
      Navigator.pop(context);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Add Entry')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              controller: _nameController,
              decoration: const InputDecoration(labelText: 'Name'),
            ),

            const SizedBox(height: 20),

            TextField(
              controller: _priceController,
              decoration: const InputDecoration(labelText: 'Price'),
              keyboardType: TextInputType.number,
            ),

            const SizedBox(height: 20),

            ElevatedButton(
              onPressed: () async {
                await _saveEntry();
              },
              child: const Text('Save'),
            ),
          ],
        ),
      ),
    );
  }
}
```

Save behavior:

```text
User taps Save
→ UI reads repository
→ repository inserts data into Drift/SQLite
→ Drift stream updates
→ Riverpod provider receives new data
→ UI rebuilds automatically
```

## 12. Add Routing

File:

```text
lib/main.dart
```

Add `go_router` if your app has multiple pages and Use it in `MaterialApp`:

Example:

```dart
import 'package:go_router/go_router.dart';

final GoRouter _router = GoRouter(
  routes: [
    GoRoute(path: '/', builder: (context, state) => const DashboardPage()),
    GoRoute(
      path: '/add-entry',
      builder: (context, state) => const AddEntryPage(),
    ),
  ],
);

void main() {
  runApp(const ProviderScope(child: MyApp()));
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router,
      title: 'Flutter Demo',
      theme: ThemeData(),
    );
  }
}

```

To Navigate:

```dart
context.push('/add-entry');
```

## 13. Add a Migration

File:

```text
lib/data/local/app_database.dart
```

When changing the database schema, increase the version:

```dart
@override
int get schemaVersion => 2;
```

Then add upgrade logic:

```dart
@override
MigrationStrategy get migration => MigrationStrategy(
      onCreate: (migrator) async => migrator.createAll(),
      onUpgrade: (migrator, from, to) async {
        if (from < 2) {
          await migrator.addColumn(entries, entries.date);
        }
      },
    );
```

Example table change:

```dart
class Items extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  RealColumn get price => real()();

  DateTimeColumn get date => dateTime().nullable()();
}
```

After changing schema files, regenerate code:

```bash
dart run build_runner build
```

## 14. Add Repository Tests

File:

```text
test/repository/app_repository_test.dart
```

Example:

```dart
import 'package:drift/native.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:drift_riverpod/ data/local/app_database.dart';
import 'package:drift_riverpod/ data/repository/app_repository.dart';

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

  test('createItem inserts and returns the created item', () async {
    final item = await repository.createItem(name: 'Milk', price: 4.25);

    expect(item.id, isPositive);
    expect(item.name, 'Milk');
    expect(item.price, 4.25);

    final items = await repository.getAllItems();
    expect(items, hasLength(1));
    expect(items.single, item);
  });

  test('getAllItems returns items ordered by name', () async {
    await repository.createItem(name: 'Yogurt', price: 3.50);
    await repository.createItem(name: 'Apples', price: 2.25);
    await repository.createItem(name: 'Bread', price: 1.75);

    final items = await repository.getAllItems();

    expect(items.map((item) => item.name), ['Apples', 'Bread', 'Yogurt']);
  });

  test('watchItems emits items ordered by name after changes', () async {
    expect(await repository.watchItems().first, isEmpty);

    final emissions = expectLater(
      repository.watchItems().map((items) => items.map((item) => item.name)),
      emitsInOrder([
        isEmpty,
        ['Oranges'],
        ['Bananas', 'Oranges'],
      ]),
    );

    await repository.createItem(name: 'Oranges', price: 5);
    await repository.createItem(name: 'Bananas', price: 2);

    await emissions;
  });
}
```

Run tests:

```bash
flutter test
```

## 15. Useful Commands

Get packages:

```bash
flutter pub get
```

Add package:

```bash
flutter pub add package_name
```

Add dev package:

```bash
flutter pub add --dev package_name
```

Generate Drift files:

```bash
dart run build_runner build
```

Clean generated conflicts:

```bash
dart run build_runner build --delete-conflicting-outputs
```

Analyze code:

```bash
flutter analyze
```

Run tests:

```bash
flutter test
```

Run app:

```bash
flutter run
```

List devices:

```bash
flutter devices
```

Clean build files:

```bash
flutter clean
```

After `flutter clean`, run:

```bash
flutter pub get
```

Then run the app again:

```bash
flutter run
```

## 16. Typical Development Flow

```text
Create/update table
→ update schemaVersion if needed
→ add migration if existing users need upgrade
→ run build_runner
→ update repository methods
→ expose data through providers
→ use providers in UI
→ run analyze/tests
→ run app
```

## 17. Important Notes

- Do not edit generated files like `app_database.g.dart` manually.
- Run `build_runner` whenever Drift tables or database definitions change.
- Use `ref.watch(...)` for live UI data.
- Use `ref.read(...)` for save/update/delete actions.
- Keep database queries out of UI files when possible.
- Put database operations inside a repository.
- Use migrations when changing tables after the app already has stored user data.
```

## Build / Run for Specific Devices

List connected devices:

```bash
flutter devices
```

Example targets:

```text
emulator-5554   Android emulator
linux           Linux desktop
chrome          Web / Chrome
```

List installed Android emulators:

```bash
flutter emulators
```

Start an Android emulator:

```bash
flutter emulators --launch emulator_name
```

Run on exact Android emulator/device:

```bash
flutter run -d emulator-5554
```

Run on Linux desktop:

```bash
flutter run -d linux
```

Run on Chrome/web:

```bash
flutter run -d chrome
```

### Builds

Build Android APK:

```bash
flutter build apk
```

Build Android App Bundle for Play Store:

```bash
flutter build appbundle
```

Build web app:

```bash
flutter build web
```

Build Linux desktop app:

```bash
flutter build linux
```

Build Windows desktop app:

```bash
flutter build windows
```

Build macOS desktop app:

```bash
flutter build macos
```

Build iOS app:

```bash
flutter build ios
```

## Platform Notes

Android can be built from Linux, macOS, or Windows.

Web can be built from Linux, macOS, or Windows.

Linux desktop must be built on Linux.

Windows desktop must be built on Windows.

macOS desktop must be built on macOS.

iOS must be built on macOS with Xcode installed.

## Enable Desktop/Web Targets

If a platform folder is missing, enable the platform and recreate it:

```bash
flutter config --enable-web
flutter config --enable-linux-desktop
flutter config --enable-windows-desktop
flutter config --enable-macos-desktop
```

Then from the project root:

```bash
flutter create .
```

This adds any missing platform folders without replacing your `lib/` code.

## Common Clean Build Flow

```bash
flutter clean
flutter pub get
dart run build_runner build --delete-conflicting-outputs
flutter analyze
flutter test
flutter run -d emulator-5554
```

# Build linux image and add desktop entry + Create terminal command

```bash
cd /my_app
flutter clean
flutter pub get
flutter build linux

# Install Fresh
# Copy the new bundle:

mkdir -p ~/.local/share/my_app
cp -r build/linux/x64/release/bundle/* ~/.local/share/my_app/

# Create terminal command:

mkdir -p ~/.local/bin
ln -sf ~/.local/share/my_app/my_app ~/.local/bin/my_app

# Test it:
my_app

# Create Desktop Entry
# Create this file:

nano ~/.local/share/applications/my_app.desktop

# Paste:
[Desktop Entry]
Type=Application
Name=My App
Exec=/home/<user-name>/.local/share/my_app/my_app
Icon=/home/<user-name>/<path-to-project>/assets/icons/my_app.svg
Terminal=false
Categories=Utility;

# Refresh app menu:
update-desktop-database ~/.local/share/applications

# Then search My App in the Linux Mint menu.
```

# Build Android APK and Install on Physical Device

Use this when you want to create an APK file, send it to your Android phone, and install it manually.

```bash
cd my_app
flutter clean
flutter pub get
flutter build apk --debug
```

Debug APK output:

```text
build/app/outputs/flutter-apk/app-debug.apk
```

Full file path example:

```text
/home/<user-name>/<path-to-project>/build/app/outputs/flutter-apk/app-debug.apk
```

Share this APK to your phone using USB, Bluetooth, Telegram, WhatsApp, Google Drive, etc.

On the phone:

```text
Open APK file
Allow "Install unknown apps" if Android asks
Tap Install
Open List Tracker
```

For a smaller release APK:

```bash
flutter build apk --release
```

Release APK output:

```text
build/app/outputs/flutter-apk/app-release.apk
```

For direct USB install, enable Developer Options and USB Debugging on the phone, then run:

```bash
flutter devices
flutter install -d <device-id>
```

Or install the APK with ADB:

```bash
adb install build/app/outputs/flutter-apk/app-debug.apk
```

Notes:

- Use `--debug` for personal testing.
- Use `--release` for a smaller/faster APK.
- A release APK should be properly signed before serious sharing or publishing.
