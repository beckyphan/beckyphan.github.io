---
layout: post
title:      "SQL & ORMs"
date:       2019-07-25 21:04:32 +0000
permalink:  sql_and_orms
---


> They say big data is taking over, it's true.
> It seems we've bitten off much more than we can chew.
> Ubiquitious and evergrowing, there's more of it than our capacity of knowing.
> So we'll table it, make it manageable, analyzable, give it value.
> 

This blog will be me talking aloud~

Structured Query Language is a language used to "talk to" databases.
The structure in which data is stored with SQL is in tables.
These tables live in a database (e.g. music_library.db)

Quick reference sheet for SQL keywords & commands:
```
CREATE TABLE table_name (
     id INTEGER PRIMARY KEY,
		 name TEXT,
		 cost REAL,
		 notes BLOB
		 );
		 
DROP TABLE table_name => deletes table from database
ALTER TABLE table_name ADD COLUMN column_name => add column to a table
		 
.tables => returns tables in the database 

.schema => returns tables and their structures

.header(s) ON|OFF      Turn display of headers on or off

.mode *MODE*
                        Set output mode where MODE is one of:
                         column   Left-aligned columns.  (See .width)
                         line     One value per line
                         tabs     Tab-separated values
                         tcl      TCL list elements

.width auto      # adjusts and normalizes column width
# or
.width NUM1, NUM2 # customize column width

INSERT INTO table_name (name, age, breed) VALUES 
   ('Maru', 3, 'Scottish Fold');
	 
INSERT INTO table_name (id, etc, etc2) VALUES 
   (1, 1, 1),
	 (1, 2, 3),
	 (4, 5, 1);
	 
	 
SELECT * FROM table_name;
SELECT id, name, age, breed FROM table_name;
SELECT DISTINCT name FROM table_name; => unique names

UPDATE table_name SET column_name= "value" WHERE column_name = "valu";

DELETE FROM table_name WHERE column_name = value;

SELECT *AGGREGATOR*(column_name) FROM table_name;
                                                where AGGREGATOR is
																								AVG
																								SUM
																								COUNT
																								MIN
																								MAX
	 
	 
Tables can be joined and connected, so long as there is a foreign key to connect them.
Rows can be filtered using WHERE, ordered, grouped and further filtered using HAVING, and results limited by a number.
Note: If using both WHERE and HAVING, WHERE is used before GROUP BY and HAVING used after.
example:
  SELECT SUM(cats.net_worth)
  FROM owners
  INNER JOIN cats_owners
  ON owners.id = cats_owners.owner_id
  JOIN cats ON cats_owners.cat_id = cats.id
  WHERE cats_owners.owner_id = 2
	ORDER BY() ASC|DESC
  GROUP BY() HAVING
	LIMIT
```

Object Relational Mapping connect Ruby programs to databases. As classes are mapped to tables inside a database, Ruby programs should note in their environment the database that they are reaching into for data.
```
DB = {:conn => SQLite3::Database.new("db/music.db")}
```
Putting the above code aka saving the DB path as a hash into an environment file allows us to reach into the database using
```
DB[:conn]
```
Looks fancifully simple.

Within a class, heredocs are used to write multi-line SQL code and then thrown in execute:
```
class ClassName
  def method
    sql =  <<-SQL 
      CREATE TABLE IF NOT EXISTS table_name (
        id INTEGER PRIMARY KEY, 
        name TEXT, 
        )
        SQL
				
    DB[:conn].execute(sql) 
  end
end
```

When needing to fill in fields, a ? can be used similar to how #{} is used as expression substitution within a string literal:
```
class Song
  def self.find_by_name(name)
    sql = <<-SQL
      SELECT *
      FROM songs
      WHERE name = ?
      LIMIT 1
    SQL
 
    DB[:conn].execute(sql, name).map do |row|
      self.new_from_db(row)
    end.first
  end
end
```

In the above code, the heredoc is executed along with name as an argument filling in for the ?
DB[:conn].execute(sql) returns an array of rows that are created.
To return a modified array after doing some action on each item in the array, map or collect is used. Map/collect will return the new/modified array without modifying the original array.


When adding rows/inserting data into a table via initializing an instance of a class, the id aka PRIMARY KEY will not be assigned at initiation to avoid duplication, etc.  Thus, when saving a new object as a row, that's the time to pull the given id to assign to the instance's id variable.
```
def save
  sql = <<-SQL
    INSERT INTO songs (name, album)
    VALUES (?, ?)
  SQL
  DB[:conn].execute(sql, self.name, self.album)
  @id = DB[:conn].execute("SELECT last_insert_rowid() FROM songs")[0][0]
end
```
@id = DB[:conn].execute("SELECT last_insert_rowid() FROM songs") => returns the last row in an array. [[id, name, album]]
@id = DB[:conn].execute("SELECT last_insert_rowid() FROM songs")[0] => reaches into the array to pull out the first object
@id = DB[:conn].execute("SELECT last_insert_rowid() FROM songs")[0][0] => reaches into the first element (id) belonging to that object







