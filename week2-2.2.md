# Homework 2.2

Write a program in the language of your choice that will remove the grade of type "homework" with the lowest score for each student from the dataset in the handout. Since each document is one grade, it should remove one document per student. This will use the same data set as the last problem, but if you don't have it, you can download and re-import.

The dataset contains 4 scores each for 200 students.

First, let’s confirm your data is intact; the number of documents should be 800.

```
use students
db.grades.count()
```
Hint/spoiler: If you select homework grade-documents, sort by student and then by score, you can iterate through and find the lowest score for each student by noticing a change in student id. As you notice that change of student_id, remove the document.

To confirm you are on the right track, here are some queries to run after you process the data and put it into the grades collection:

Let us count the number of grades we have:

```
db.grades.count()
``` 
The result should be 600. Now let us find the student who holds the 101st best grade across all grades:

```
db.grades.find().sort( { 'score' : -1 } ).skip( 100 ).limit( 1 )
```
The correct result will be:

```
{ "_id" : ObjectId("50906d7fa3c412bb040eb709"), "student_id" : 100, "type" : "homework", "score" : 88.50425479139126 }
```
Now let us sort the students by student_id, type, and score, and then see what the top five docs are:

```
db.grades.find( { }, { 'student_id' : 1, 'type' : 1, 'score' : 1, '_id' : 0 } ).sort( { 'student_id' : 1, 'score' : 1, } ).limit( 5 )
```
The result set should be:

```
{ "student_id" : 0, "type" : "quiz", "score" : 31.95004496742112 }
{ "student_id" : 0, "type" : "exam", "score" : 54.6535436362647 }
{ "student_id" : 0, "type" : "homework", "score" : 63.98402553675503 }
{ "student_id" : 1, "type" : "homework", "score" : 44.31667452616328 }
{ "student_id" : 1, "type" : "exam", "score" : 74.20010837299897 }
```
To verify that you have completed this task correctly, provide the identity of the student with the highest average in the class with following query that uses the aggregation framework. The answer will appear in the _id field of the resulting document.

```
db.grades.aggregate( { '$group' : { '_id' : '$student_id', 'average' : { $avg : '$score' } } }, { '$sort' : { 'average' : -1 } }, { '$limit' : 1 } )
```
Enter the student ID below. Please enter just the number, with no spaces, commas or other characters.

## Answer

Aggregate all homeworks with minimal score, using cursor.forEach delete them:
```
db.grades
    .aggregate(
        { $match: { type: "homework"} },
        { $group: { _id: "$student_id", score: { $min: "$score"} }}
    )
    .forEach(function(doc) {
        db.grades.remove({student_id: doc._id, score: doc.score});
    })
```

Or using C# language, a bit ugly but anyway works
```
var badHomeworks = gradesCollection.Aggregate()
    .Match(Builders<BsonDocument>.Filter.Eq("type", "homework"))
    .Group(new BsonDocument {{"_id", "$student_id"}, {"score", new BsonDocument("$min", "$score")}})
    .ToList();

foreach (var homework in badHomeworks)
{
    var builder = Builders<BsonDocument>.Filter;
    var filter = builder.Eq("student_id", homework["_id"]) & builder.Eq("score", homework["score"]);

    gradesCollection.DeleteOne(filter);
}
```