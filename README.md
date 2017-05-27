# sqflite

An experimental SQLite plugin for [Flutter](https://flutter.io).
Supports both iOS and Android.

## Getting Started

In your flutter project add the dependency:

    dependencies:
      ...
      sqflite:
       git: git://github.com/tekartik/sqflite
    

For help getting started with Flutter, view the online
[documentation](https://flutter.io/).

## Usage example

Import `sqflite.dart`

    import 'package:sqflite/sqflite.dart';
    
Demo code


    // Get a location using path_provider
    Directory documentsDirectory = await getApplicationDocumentsDirectory();
    String path = join(documentsDirectory.path, "demo.db");

    // Delete the database
    deleteDatabase(path);

    // open the database
    Database database = await openDatabase(path, version: 1,
      onCreate: (Database db, int version) async {
        // When creating the db, create the table
        await db.execute(
            "CREATE TABLE Test (id INTEGER PRIMARY KEY, name TEXT, value INTEGER)");
    });

    // Insert some record
    await database.inTransaction(() async {
      int id1 = await database
          .insert('INSERT INTO Test(name, value) VALUES("some name",1234)');
      print("inserted1: $id1");
      int id2 = await database.insert('INSERT INTO Test(name, value) VALUES(?, ?)',
          ["another name", 12345678]);
      print("inserted2: $id2");
    });

    // Update some record
    int count = await database.update(
        'UPDATE Test SET name = ?, VALUE = ? WHERE name = ?',
        ["updated name", "9876", "some name"]);
    print("updated: $count");

    // Get the records
    List<Map> list = await database.query('SELECT * FROM Test');
    List<Map> expectedList = [
      {"name": "updated name", "id": 1, "value": 9876},
      {"name": "another name", "id": 2, "value": 12345678}
    ];
    assert(const DeepCollectionEquality().equals(list, expectedList));

    // Count the records
    count = Sqflite.firstIntValue(await database.query("SELECT COUNT(*) FROM Test"));
    assert(count == 2);

    // Delete a record
    count = await database.delete('DELETE FROM Test WHERE name = ?', ['another name']);
    assert(count == 1);
      
    // Close the database
    await database.close();

## Current issues

* Due to the way transaction works in SQLite (threads), concurrent read and write transaction are not supported yet in 
this sample demo. All calls are currently synchronized and transactions block are exclusive. This will be fixed by creating 
a native thread for each transaction and zoning inTransaction calls
* Recursive transactions are not supported yet
* Only TEXT and INTEGER types are tested for now
* Need to check threading on iOS, there are bugs where sometimes the result is not available right away
