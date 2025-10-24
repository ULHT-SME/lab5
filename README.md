# Lab5

## Description

This hands-on lab introduces students to Firebase and Cloud Firestore, Google's real-time cloud database. Students will set up Firebase in a Flutter project, create a Firestore collection, and build a Notes application that reads, writes, and deletes data in real-time. By the end of this lab, students will have a working notes app with cloud persistence and understand fundamental Firestore operations.

## Learning Objectives

By completing this lab, students will be able to:

- Set up Firebase in a Flutter project and configure Firestore database
- Understand the structure of Firestore collections and documents
- Read real-time data from Firestore using StreamBuilder
- Add new documents to Firestore collections using write operations
- Delete documents from Firestore
- Handle asynchronous operations with Firestore and update UI accordingly
- Implement a functional CRUD application with cloud backend

## Setup Instructions

### Part 1: Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **"Create a project"** or select an existing project
3. Enter project name (e.g., "flutter-notes-app")
4. Follow the setup steps, accepting default settings
5. Wait for project to be created
6. You now have a Firebase project

### Part 2: Create a Flutter Project

Option 1: Open terminal and run:

```bash
flutter create firebase_notes_app
cd firebase_notes_app
```

Option 2: Create the project via Android Studio > File > New > New Flutter Project

### Part 3: Add Flutter App to Firebase

1. In Firebase Console, click **"Add app"** and select **Flutter**
2. Install [Firebase CLI](https://firebase.google.com/docs/cli?hl=pt-BR&authuser=0&_gl=1*17czby4*_ga*NTYwMzcxMDg5LjE3NjA5NzI2MTY.*_ga_CW55HF8NVT*czE3NjA5NzI2MTYkbzEkZzEkdDE3NjA5NzM0ODMkajYwJGwwJGgw#install_the_firebase_cli)
  - You may need the instalation of [Node.js](https://nodejs.org/en/download)
  - In a terminal in your computer run the following ```npm install -g firebase-tools```
3. After success instalation of firebase-tools Run the following command: ```firebase login```. This will log you in to firebase, but in console. Some web browser prompts will pop
4. Follow the steps in described in Firebase console by enabling the flutterfire_cli in dart: ```dart pub global activate flutterfire_cli````
5. Finally, in a terminal with root directory your project, execute the following command: ```flutterfire configure --project=<This ID is visible in firebase console instructions>```
6. If everything ran correctly, you should have a new file under ```lib/firebase_options.dart```. This file is **extremely important** and cannot be deleted or else the connection of the project to Firebase is lost.


### Part 4: Add Firebase Dependencies

Open `pubspec.yaml` and add these packages under dependencies:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.24.0
  cloud_firestore: ^4.13.0
```

Run to install:

```bash
flutter pub get
```

### ATTENTION: For Mac  Users

If you are building for iOS, you will probably have issues building with Firebase dependencies.
You need to add the following in your Podfile located under ``ìos```folder.

```
post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
    if target.name == 'BoringSSL-GRPC'
      target.source_build_phase.files.each do |file|
        if file.settings && file.settings['COMPILER_FLAGS']
          flags = file.settings['COMPILER_FLAGS'].split
          flags.reject! { |flag| flag == '-GCC_WARN_INHIBIT_ALL_WARNINGS' }
          file.settings['COMPILER_FLAGS'] = flags.join(' ')
        end
      end
    end
  end
end

```

### Part 5: Initialize Firebase in Flutter

Replace `lib/main.dart` with:

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Firebase Notes App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: NotesScreen(),
    );
  }
}

class NotesScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('My Notes'),
      ),
      body: Center(
        child: Text('Firebase initialized! Add notes UI here.'),
      ),
    );
  }
}
```

### Part 6: Verify Setup

Option 1: Run your app:

```bash
flutter run
```

Option 2: In Android Studio press the green icon ▶️ with the chosen device (Emulator of Phisical mobile)


If you see no errors and the app displays "Firebase initialized!", you're ready for the lab tasks.

---

## Lab Tasks

### Task 1: Create Firestore Collection & Add Sample Data 

**Objective:** Set up a Firestore collection with sample data in Firebase Console.

**In Firebase Console:**

1. Go to your Firebase project
2. Click **"Firestore Database"** from the left menu
3. Click **"Create database"**
4. Select **"Start in test mode"** (for development; never use for production)
5. Choose your region (closest to you) and click **"Create"**
6. Click **"Create collection"** and name it `notes`
7. Add sample documents with these fields:
   - `title`: "Welcome to Notes" (string)
   - `content`: "Create your first note!" (string)
   - `timestamp`: (auto-generated by Firestore)

**Repeat to add 2-3 more sample notes.**

**Your Collection Structure Should Look Like:**
```
notes/ (collection)
  ├── doc1
  │   ├── title: "Welcome to Notes"
  │   ├── content: "Create your first note!"
  │   └── timestamp: 1234567890
  ├── doc2
  │   ├── title: "Note 2"
  │   ├── content: "This is another note"
  │   └── timestamp: 1234567891
  └── doc3
      ├── title: "Note 3"
      ├── content: "And another one"
      └── timestamp: 1234567892
```

**Firestore Concepts:**
- **Collection:** A group of documents (like a table in SQL)
- **Document:** Individual records with an ID and fields
- **Fields:** Key-value pairs within documents

### Task 2: Create a Note Model Class 

**Objective:** Create a Dart model to represent notes from Firestore.

**Your Challenge:**

Create a new file `lib/models/note.dart`:

```dart
class Note {
  final String id;
  final String title;
  final String content;
  final DateTime timestamp;

  Note({
    required this.id,
    required this.title,
    required this.content,
    required this.timestamp,
  });

  // Factory constructor to create Note from Firestore document
  factory Note.fromFirestore(Map<String, dynamic> data, String documentId) {
    return Note(
      id: documentId,
      title: data['title'] ?? '',
      content: data['content'] ?? '',
      // TODO: Convert Firestore timestamp to DateTime
      // Hint: Firestore stores timestamps as special Timestamp type
      // Use: (data['timestamp'] as dynamic).toDate() if available
      timestamp: data['timestamp'] != null 
          ? (data['timestamp'] as dynamic).toDate() 
          : DateTime.now(),
    );
  }

  // Convert Note to Map for Firestore
  Map<String, dynamic> toFirestore() {
    return {
      // TODO: Map Note properties to Firestore fields
      // Hint: 'title': title, 'content': content
    };
  }
}
```

**Complete the Model:**
- Fill in the `toFirestore()` method to convert Note back to Firestore format
- For timestamp, use `FieldValue.serverTimestamp()` when adding to Firestore

**Hints:**
- Firestore timestamps need special handling with `.toDate()`
- Server timestamp ensures all timestamps are consistent

### Task 3: Read and Display Notes with StreamBuilder 

**Objective:** Fetch notes from Firestore in real-time and display them in a ListView.

**Your Challenge:**

Create a new file `lib/screens/notes_screen.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/note.dart';

class NotesScreen extends StatelessWidget {
  final FirebaseFirestore _firestore = FirebaseFirestore.instance;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('My Notes'),
      ),
      body: StreamBuilder<QuerySnapshot>(
        // TODO: Add stream from Firestore
        // Hint: _firestore.collection('notes').orderBy('timestamp', descending: true).snapshots()
        stream: _firestore
            .collection('notes')
            .orderBy('timestamp', descending: true)
            .snapshots(),
        builder: (context, snapshot) {
          // TODO: Handle loading state
          if (snapshot.connectionState == ConnectionState.waiting) {
            return Center(
              child: CircularProgressIndicator(),
            );
          }

          // TODO: Handle error state
          if (snapshot.hasError) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.error_outline, size: 60, color: Colors.red),
                  SizedBox(height: 16),
                  Text('Error: ${snapshot.error}'),
                ],
              ),
            );
          }

          // TODO: Handle empty state
          if (!snapshot.hasData || snapshot.data!.docs.isEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.note_outlined, size: 60, color: Colors.grey),
                  SizedBox(height: 16),
                  Text('No notes yet. Create one to get started!'),
                ],
              ),
            );
          }

          // TODO: Display notes in ListView
          List<Note> notes = snapshot.data!.docs
              .map((doc) => Note.fromFirestore(doc.data() as Map<String, dynamic>, doc.id))
              .toList();

          return ListView.builder(
            itemCount: notes.length,
            itemBuilder: (context, index) {
              Note note = notes[index];
              return Card(
                margin: EdgeInsets.symmetric(horizontal: 12, vertical: 6),
                child: ListTile(
                  title: Text(
                    note.title,
                    style: TextStyle(fontWeight: FontWeight.bold),
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  subtitle: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      SizedBox(height: 4),
                      Text(
                        note.content,
                        maxLines: 2,
                        overflow: TextOverflow.ellipsis,
                      ),
                      SizedBox(height: 4),
                      Text(
                        _formatDate(note.timestamp),
                        style: TextStyle(fontSize: 12, color: Colors.grey),
                      ),
                    ],
                  ),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      // TODO: Add edit button (in optional challenge)
                      IconButton(
                        icon: Icon(Icons.delete, color: Colors.red),
                        onPressed: () {
                          // TODO: Delete note in Task 5
                          print('Delete ${note.id}');
                        },
                      ),
                    ],
                  ),
                  onTap: () {
                    // TODO: View full note details (optional)
                  },
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // TODO: Show add note dialog in Task 4
          print('Add note');
        },
        child: Icon(Icons.add),
      ),
    );
  }

  String _formatDate(DateTime date) {
    return '${date.month}/${date.day}/${date.year} ${date.hour}:${date.minute}';
  }
}
```

**Update main.dart:**
```dart
import 'screens/notes_screen.dart';

// In MyApp:
home: NotesScreen(),
```

**Key Concepts:**
- `StreamBuilder` listens to real-time updates from Firestore
- `snapshot.connectionState.waiting` - data still loading
- `snapshot.hasError` - connection error occurred
- `snapshot.hasData` - data is available
- `.orderBy()` sorts documents
- `.snapshots()` creates a stream of updates

**Test Your App:**
- Run the app and see your sample notes displayed
- Try adding a note manually in Firebase Console and watch it appear in your app
- Notice how the list updates in real-time

### Task 4: Add Notes with Dialog 

**Objective:** Create a dialog to add new notes to Firestore.

**Your Challenge:**

Add this method to `NotesScreen`:

```dart
void _showAddNoteDialog(BuildContext context) {
  final TextEditingController titleController = TextEditingController();
  final TextEditingController contentController = TextEditingController();

  showDialog(
    context: context,
    builder: (BuildContext context) {
      return AlertDialog(
        title: Text('Add Note'),
        content: SingleChildScrollView(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              TextField(
                controller: titleController,
                decoration: InputDecoration(
                  labelText: 'Title',
                  border: OutlineInputBorder(),
                ),
              ),
              SizedBox(height: 16),
              TextField(
                controller: contentController,
                decoration: InputDecoration(
                  labelText: 'Content',
                  border: OutlineInputBorder(),
                ),
                maxLines: 4,
              ),
            ],
          ),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('Cancel'),
          ),
          ElevatedButton(
            onPressed: () {
              // TODO: Add note to Firestore
              _addNote(
                titleController.text,
                contentController.text,
              );
              Navigator.pop(context);
            },
            child: Text('Add'),
          ),
        ],
      );
    },
  );
}
```

Add this method to add notes to Firestore:

```dart
Future<void> _addNote(String title, String content) async {
  if (title.isEmpty || content.isEmpty) {
    // TODO: Show validation error
    print('Title and content cannot be empty');
    return;
  }

  try {
    // TODO: Add document to 'notes' collection
    // Hint: Use _firestore.collection('notes').add()
    await _firestore.collection('notes').add({
      'title': title,
      'content': content,
      'timestamp': FieldValue.serverTimestamp(),
    });
    print('Note added successfully');
  } catch (e) {
    print('Error adding note: $e');
  }
}
```

**Update FAB in NotesScreen:**

Replace the FAB's onPressed:

```dart
floatingActionButton: FloatingActionButton(
  onPressed: () => _showAddNoteDialog(context),
  child: Icon(Icons.add),
),
```

**Don't forget to import:**
```dart
import 'package:cloud_firestore/cloud_firestore.dart';
```

**Key Firestore Concepts:**
- `.add()` creates a new document with auto-generated ID
- `FieldValue.serverTimestamp()` uses server time (recommended over client time)
- Always validate input before adding to database

**Test Your App:**
- Tap the FAB to open the dialog
- Enter title and content
- Click "Add"
- Watch the new note appear in the list in real-time

### Task 5: Delete Notes 

**Objective:** Implement the ability to delete notes from Firestore.

**Your Challenge:**

Add this method to `NotesScreen`:

```dart
Future<void> _deleteNote(String noteId) async {
  try {
    // TODO: Delete document from Firestore
    // Hint: Use _firestore.collection('notes').doc(noteId).delete()
    
    print('Note deleted successfully');
  } catch (e) {
    print('Error deleting note: $e');
  }
}
```

**Update the delete button in ListView:**

Replace the delete button's onPressed:

```dart
IconButton(
  icon: Icon(Icons.delete, color: Colors.red),
  onPressed: () {
    // TODO: Show confirmation dialog before deleting
    _showDeleteConfirmation(context, note.id);
  },
),
```

**Add confirmation dialog:**

```dart
void _showDeleteConfirmation(BuildContext context, String noteId) {
  showDialog(
    context: context,
    builder: (BuildContext context) {
      return AlertDialog(
        title: Text('Delete Note?'),
        content: Text('This action cannot be undone.'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('Cancel'),
          ),
          ElevatedButton(
            onPressed: () {
              _deleteNote(noteId);
              Navigator.pop(context);
            },
            style: ElevatedButton.styleFrom(
              backgroundColor: Colors.red,
            ),
            child: Text('Delete', style: TextStyle(color: Colors.white)),
          ),
        ],
      );
    },
  );
}
```

**Key Concepts:**
- `.doc(documentId).delete()` removes a document
- Use confirmation dialogs to prevent accidental deletion
- Firestore updates the stream immediately after deletion

**Test Your App:**
- Tap delete on any note
- Confirm deletion
- Watch the note disappear from the list

### Task 6: Challenge - Edit Notes 

**Objective:** Allow users to edit existing notes.

**Your Challenge:**

1. Add an edit button next to the delete button in ListView
2. Create `_showEditNoteDialog()` method similar to add dialog
3. Implement `_updateNote()` method using `.doc(id).update()`

**Edit Button in ListView:**

```dart
trailing: Row(
  mainAxisSize: MainAxisSize.min,
  children: [
    IconButton(
      icon: Icon(Icons.edit, color: Colors.blue),
      onPressed: () {
        // TODO: Show edit dialog with current note data
        _showEditNoteDialog(context, note);
      },
    ),
    IconButton(
      icon: Icon(Icons.delete, color: Colors.red),
      onPressed: () {
        _showDeleteConfirmation(context, note.id);
      },
    ),
  ],
),
```

**Edit Dialog Method:**

```dart
void _showEditNoteDialog(BuildContext context, Note note) {
  final TextEditingController titleController = TextEditingController(text: note.title);
  final TextEditingController contentController = TextEditingController(text: note.content);

  showDialog(
    context: context,
    builder: (BuildContext context) {
      return AlertDialog(
        title: Text('Edit Note'),
        content: SingleChildScrollView(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              TextField(
                controller: titleController,
                decoration: InputDecoration(
                  labelText: 'Title',
                  border: OutlineInputBorder(),
                ),
              ),
              SizedBox(height: 16),
              TextField(
                controller: contentController,
                decoration: InputDecoration(
                  labelText: 'Content',
                  border: OutlineInputBorder(),
                ),
                maxLines: 4,
              ),
            ],
          ),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('Cancel'),
          ),
          ElevatedButton(
            onPressed: () {
              // TODO: Update note in Firestore
              // Do the function call here
              Navigator.pop(context);
            },
            child: Text('Save'),
          ),
        ],
      );
    },
  );
}
```

**Update Method:**

```dart
Future<void> _updateNote(String noteId, String title, String content) async {
  try {
    // TODO: Update document using .doc(id).update()
    // Hint: _firestore.collection('notes').doc(noteId).update(...)

    print('Note updated successfully');
  } catch (e) {
    print('Error updating note: $e');
  }
}
```

**Key Concepts:**
- `.update()` modifies specific fields without overwriting entire document
- Pre-populate TextFields with existing data for better UX
- Always validate before updating

---

## Deliverables

You are required to .zip file this project and deliver it in Moodle before the next class. The .zip should contain all the content of inside project folder with the exception of the .dart_tool and build folders (these will make the .zip file extremely large and they are not necessary for evaluation).

---

## Tips and Resources

### Common Issues and Solutions

**Firebase initialization fails:**
- Ensure `flutterfire configure` was run successfully
- Check that `google-services.json` is in `android/app/`
- Verify `GoogleService-Info.plist` is added to Xcode project
- Clean and rebuild: `flutter clean && flutter pub get && flutter run`

**Collection not appearing in Firestore Console:**
- Make sure you manually created the collection and added documents
- Wait a few seconds for console to refresh
- Collections appear only after first document is added

**Timestamp issues:**
- Use `FieldValue.serverTimestamp()` for consistency
- Always call `.toDate()` when reading timestamps from Firestore
- Check that timestamp field exists in all documents

**Permission denied errors:**
- You're in test mode which allows all read/write
- For production, you'll need security rules (covered in future labs)
- Never deploy test mode to production

### Best Practices

- Always validate user input before adding to database
- Use confirmation dialogs for destructive operations (delete)
- Show loading indicators during async operations
- Handle errors gracefully with user-friendly messages
- Use server timestamps to avoid client time inconsistencies
- Organize data efficiently to minimize read operations

### Additional Learning

- Explore Firestore in the Firebase Console to understand database structure
- Try querying with `.where()` clauses to filter documents
- Implement pagination for apps with many documents
- Learn about composite indexes for complex queries
- Add field validation using Firebase Cloud Functions
- Explore Firestore offline persistence

