# Firebase-FireStore-Learning

Cloud Firestore is a NoSQL, document-oriented database. Unlike a SQL database, there are no tables or rows. Instead, you store data in documents, which are organized into collections.

Each document contains a set of key-value pairs. Cloud Firestore is optimized for storing large collections of small documents.

All documents must be stored in collections. Documents can contain subcollections (but not another document directly, similarly A collection contains documents and nothing else) and nested objects, both of which can include primitive fields like strings or complex objects like lists. **Documents support extra data types and are limited in size to 1 MB (after that you have to break down the document.)**
``` 
** always follow this pattern.**
val messageRef = db
        .collection("rooms").document("roomA")
        .collection("messages").document("message1") 
```

To know more about Documents and Collections: Visit -> https://firebase.google.com/docs/firestore/data-model

### Adding data :
```
val docData = hashMapOf(
        "stringExample" to "Hello world!",
        "booleanExample" to true,
        "numberExample" to 3.14159265,
        "dateExample" to Timestamp(Date()),
        "listExample" to arrayListOf(1, 2, 3),
        "nullExample" to null
)

val nestedData = hashMapOf(
        "a" to 5,
        "b" to true
)

docData["objectExample"] = nestedData

db.collection("data").document("new-city-id")
        .set(docData)           // This will overwrite or create the document if not present.
        .addOnSuccessListener { Log.d(TAG, "DocumentSnapshot successfully written!") }
        .addOnFailureListener { e -> Log.w(TAG, "Error writing document", e) } 
or
db.collection("data").
        .add(docData) //By this firebase will create its own id for document. 
        .addOn...
      
```        

### Updating:
If you update a nested field without dot notation, you will overwrite the entire map field
```
// Assume the document contains:
// {
//   name: "Frank",
//   favorites: { food: "Pizza", color: "Blue", subject: "recess" }
//   age: 12
// }
//
// To update age and favorite color:
db.collection("users").document("frank")
        .update(mapOf(
                "age" to 13,
                "favorites.color" to "Red"
        ))
```
For array element, arrayUnion() adds elements to an array but only elements not already present. arrayRemove() removes all instances of each given element.
```
// Atomically add a new region to the "regions" array field.
washingtonRef.update("regions", FieldValue.arrayUnion("greater_virginia"))

// Atomically remove a region from the "regions" array field.
washingtonRef.update("regions", FieldValue.arrayRemove("east_coast"))

// Atomically increment the population of the city by 50.
washingtonRef.update("population", FieldValue.increment(50))
```
### Deleting:
To delete a document, use the delete() method:
```
db.collection("cities").document("DC")
        .delete()
        .addOnSuccessListener { Log.d(TAG, "DocumentSnapshot successfully deleted!") }
        .addOnFailureListener { e -> Log.w(TAG, "Error deleting document", e) }
```
Deleting Fields:
```
val docRef = db.collection("cities").document("BJ")

// Remove the 'capital' field from the document
val updates = hashMapOf<String, Any>(
        "capital" to FieldValue.delete()
)

docRef.update(updates).addOnCompleteListener { }
```

### Reading Data:
There are two ways to retrieve data stored in Cloud Firestore. Either of these methods can be used with documents, collections of documents, or the results of queries:

1. Call a method to get the data.
2. Set a listener to receive data-change events.
When you set a listener, Cloud Firestore sends your listener an initial snapshot of the data, and then another snapshot each time the document changes.

#### 1. Call a method to get the data:
```
val docRef = db.collection("cities").document("SF")
docRef.get()
        .addOnSuccessListener { document ->
            if (document != null) {
                Log.d(TAG, "DocumentSnapshot data: ${document.data}")
            } else {
                Log.d(TAG, "No such document")
            }
        }
        .addOnFailureListener { exception ->
            Log.d(TAG, "get failed with ", exception)
        }
or val source = Source.CACHE
docRef.get(source).addOn... to make use of offline storage while running offline.

//To get the Custom objects like these
docRef.get().addOnSuccessListener { documentSnapshot ->
    val city = documentSnapshot.toObject<City>()
}
```        
#### 2. Listening to the Updates:
```
val docRef = db.collection("cities").document("SF")
val listnervariable = docRef.addSnapshotListener { snapshot, e ->
    if (e != null) {
        Log.w(TAG, "Listen failed.", e)
        return@addSnapshotListener
    }

    if (snapshot != null && snapshot.exists()) {
        Log.d(TAG, "Current data: ${snapshot.data}")
    } else {
        Log.d(TAG, "Current data: null")
    }
}

//After work has been done so that your event callbacks stop getting called. This allows the client to stop using bandwidth to receive updates., detach the listner
listnervariable.remove()
```
