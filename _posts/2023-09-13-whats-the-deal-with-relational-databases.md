---
layout: post
title: "What's the deal with Relational Databases?"
excerpt_separator: <!--more-->
---

I recently wrote an internal blog post at [Truepill](https://truepill.com) to make relational databases a little more approachable for folks that are coming from a more document-oriented world. The main things to get used to were:
- Schemas
- Normalization of Data
- Relationships

<!--more-->

## Schemas

In relational database models, it is necessary to understand the structure of the data in order to be able to define relationships between it. Further, there are other perks of a schema oriented system - I won't get into those here outside of saying that schemas have the same general perks as types in a programming language, and schemas enable performant execution of queries by an intelligent database engine.

Each table in a relational database is defined before any data gets written to it, and this definition is the table's schema. Any attempt to write data to the table that doesn't conform to the schema will cause an exception to occur. This allows us to enforce data integrity, but doesn't mean we aren't able to evolve the schema over time.

Generally, the state of the art for handling schema changes over time is [Liquibase](https://liquibase.org). In this setup, migrations are layered on top of each other one-by-one. Applying every migration in sequence will bring you to the current database state, and if you add a new migration that is bad, it is trivial to roll it back as it simply is the latest "layer".

Here's an example:

`00010_initial_schema.sql`
```sql
CREATE TABLE fortunes (
       fortune varchar(150) not null
);

INSERT INTO fortunes(fortune) VALUES
       ('Land is always on the mind of a flying bird.'),
       ('LIFE CONSISTS NOT IN HOLDING GOOD CARDS, BUT IN PLAYING THOSE YOU HOLD WELL.'),
       ('If you look back, you''ll soon be going that way.'),
       ('The fortune you seek is in another cookie.'),
       ('The love of your life is stepping into your planet this summer.'),
       ('Help! I am being held prisoner in a fortune cookie factory.');

-- Many more fortunes, but you get the idea.
```

The only data that can be written to `fortunes` is a string, max length 150, and null values aren't allowed.

Should we wish to extend this table, we would write a migration that works on top of the latest database state. Let's say we want to add a column for the `author` of the fortune, which will be a string.

`00020_add-author-to-fortunes.sql`
```sql
ALTER TABLE fortunes ADD COLUMN author varchar(150) not null;
```

The database, intelligently, will complain that `author` currently has null values, despite the `not null` constraint put upon the `author` column. This is because `fortunes` already has existing data, which needs to be brought along for the ride with this schema change.

To keep things simple, let's just pretend nulls are okay for now. The revised migration now looks like:

`00020_add-author-to-fortunes.sql`
```sql
ALTER TABLE fortunes ADD COLUMN author varchar(150);
```

Once applying the above two migrations, here's how our table looks:

```plaintext
+---------+------------------------+-----------+
| Column  | Type                   | Modifiers |
|---------+------------------------+-----------|
| fortune | character varying(150) |  not null |
| author  | character varying(150) |           |
+---------+------------------------+-----------+
```

This layered approach lets us evolve our data model over time, but it also forces us to evolve our understanding of our data model right alongside it, and helps catch edge cases related to existing data that might not fit so nicely into our new schema.

While it isn't often the best choice for rapid prototyping, it often starts to pay off quickly as projects move past MVP, or teams grow large enough that knowledge becomes diffused. The database ends up containing everything you need to understand the data in the database, which helps folks familiar with relational databases orient themselves quickly.

## Data Normalization

In databases with well understood schemas, starting to normalize your data pays off. This is probably the biggest adjustment to make when moving from a document oriented world.

It's probably easiest to go through an example; let's reuse our `fortunes` table from earlier. We introduced the concept of an `author` but we have it as a column under `fortunes`. If we had multiple fortunes with the same author, this wouldn't be optimal from a storage perspective or a query perspective, as we'd be storing the same string over and over again for a given author, and doing a full string search any time we wanted to find fortunes written by a particular author.

Further, maybe there is other information about authors it would be helpful to retain... having a string to cram all that into becomes limiting pretty quickly.

While we could modify the column to be `JSON` and call it a day, it still wouldn't remedy the fact that the same author could show up multiple times, and even worse, if we ever needed to update the stored author information, we'd have to update every single row that stored it, which has substantial consistency risks.

```plaintext
+--------------------------+                               +---------------------------+
| Authors                  |                               | Fortunes                  |
+--------------------------+                               +---------------------------+
| *   id integer not null  |                               | *   id integer not null   |
+--------------------------+                               +---------------------------+
| **  name varchar(150)    |                               | **  fortune varchar(150)  |
+--------------------------+                               +---------------------------+
```

The above is an **ER Diagram**, somewhat stylized to fit my blog's motif. ER stands for Entity Relationship. Like everything, there's a million ways to represent things that all get called ER diagrams, so you'll likely see other formats in the wild. For the purposes of this section, we'll draw relationships with arrows, but we won't really get into the meat of how to leverage them until the next section.

This data is considered normalized, because there is no redundancy in the columns. However, it is a good time to introduce the concept of a **primary key**. In the diagram above, these are denoted with a single `*`. Without a primary key, nothing would stop us from storing two rows with the exact same values, which wouldn't be terribly useful. Having a primary key defined on `id` means there is at most one row with a given `id` - that is, we'll never two different Authors with the same `id`, or two different Fortunes with the same `id`. Another term for these is **unique index**.

We decided to do a little better, though. We know people are likely to search for an author using their name, rather than their id. So, we are able to create an **index** on name, which helps make these searches as performant as possible.

Sometimes, indexes and primary keys can involve more than one column. These are called **composite**.

There's one crucial piece of data not captured yet in the above diagram - the relationship between authors and their fortunes. This is just another table, so let's pop it in!

```plaintext
        +--------------------------+                               +---------------------------+
        | Authors                  |                               | Fortunes                  |
        +--------------------------+                               +---------------------------+
+-----> | *   id integer not null  |                        +----> | *   id integer not null   |
|       +--------------------------+                        |      +---------------------------+
|       | **  name varchar(150)    |                        |      | **  fortune varchar(150)  |
|       +--------------------------+                        |      +---------------------------+
|                                                           |
|                                                           |
|                                                           |
+-----------------------------------------------+           |
                                                |           |
                                                |           |
                                                |           |
                                                |           |
        +----------------------------------+    |           |
        | Author_Fortunes                  |    |           |
        +----------------------------------+    |           |
        | *   author_id integer not null   | +--+           |
        +----------------------------------+                |
        | *   fortune_id integer not null  | +--------------+
        +----------------------------------+

```

Whoa! Arrows! Relationship tables! What this diagram says is, now we have an Author_Fortunes table that references the id from the Authors table, and the id from the Fortunes table. Those with a keen eye may also notice that the primary key of Author_Fortunes is a composite key of (author_id, fortune_id), meaning that the same relationship couldn't be stored more than once.

This is an example of a Many-to-Many relationship. Note that we couldn't put a `fortune_id` in the Authors table without saying that the Author could have only ever written one fortune. Similarly, we couldn't put the `author_id` in the Fortunes table without saying that a given fortune could only have been written by one author... er, wait a second - we can totally say that! Let's update our diagram to reflect this:

```plaintext
      +--------------------------+                               +---------------------------------+
      | Authors                  |                               | Fortunes                        |
      +--------------------------+                               +---------------------------------+
+---> | *   id integer not null  |                               | *   id integer not null         |
|     +--------------------------+                               +---------------------------------+
|     | **  name varchar(150)    |                               | **  fortune varchar(150)        |
|     +--------------------------+                               +---------------------------------+
|                                                          +---+ | *** author_id integer not null  |
|                                                          |     +---------------------------------+
|                                                          |
+----------------------------------------------------------+
```

Now we have correctly represented that a Fortune can have one author, but an author can have many fortunes. I believe Confucious is quite a prolific author of fortunes, for example. However, if we ever wanted to represent collaborative fortune-writing, we could always reintroduce the Many-to-Many relationship later.

There's a lot of extra detail one can dive into here, but it is kind of like formatting preferences on PR's - there's a lot of diminishing value after getting a high level understanding of it, and it quickly descends into normalization for the sake of normalization. Generally, data is cheap, and normalized versus denormalized data is a tradeoff. However, for those interested:
- [Boyce-Codd Normal Form](https://en.wikipedia.org/wiki/Boyce%E2%80%93Codd_normal_form)
- [ER Diagrams](https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model)

## Relations and Relational Algebra

We brushed on relations and types of relationships in the previous section, with an example of both a Many-to-Many relationship and a One-to-Many relationship. Technically, there's also a Many-to-One relationship, but that is mostly for human benefit, as it just has directionality - a Many-to-One relationship is just going backwards on a One-to-Many relationship. E.g., An Author has many Fortunes; a Fortune has one Author.

The real fun comes in when you start figuring out how to use schemas and relations to answer complex queries performantly. Since there are a lot of implementations of SQL, an implementation agnostic discussion is to use relational algebra, which often also is used in academia. It's useful to know for whiteboarding sessions and any roving academics you come across, but I'll also pair each example with the corresponding Postgresql.

For this, let's populate our tables with some data.

```plaintext
      +--------------------------+                    +---------------------------------+
      | Authors                  |                    | Fortunes                        |
      +--------------------------+                    +---------------------------------+
+---> | *   id integer not null  |                    | *   id integer not null         |
|     +--------------------------+                    +---------------------------------+
|     | **  name varchar(150)    |                    | **  fortune varchar(150)        |
|     +------------+-------------+                    +---------------------------------+
|                  |                            +---- | *** author_id integer not null  |
|                  |                            |     +----------------+----------------+
|                  |                            |                      |
+-----------------------------------------------+                      |
                   |                                                   |
                   |                                                   |
                   |                                                   |
                   |                                                   |
+------------------------------------+    +--------+-------------------+--------------+-------------+
| id               | name            |    | id     | fortune                          | author_id   |
+------------------------------------+    +---------------------------------------------------------+
| 1                | Joe             |    | 1      | You will die a terrible death.   | 1           |
+------------------------------------+    +---------------------------------------------------------+
| 2                | Mortimer        |    | 2      | You will live a good life.       | 2           |
+------------------------------------+    +---------------------------------------------------------+
| 3                | Frank           |    | 3      | You're going to be hungry.       | 2           |
+------------------------------------+    +---------------------------------------------------------+
| 4                | Mindy           |    | 4      | You're going to be alright.      | 5           |
+------------------------------------+    +--------+----------------------------------+-------------+
| 5                | Sue             |
+------------------+-----------------+
```

Now, let's introduce our cast of characters.

- π Projection
- σ Selection
- X Cartesian Product
- U Union
- \- Set Difference
- ∩ Set Intersection
- ρ Rename

If you're used to set theory, some of these probably sound familiar. We'll only go in depth for a few of them.

### π Projection

```plaintext
     π ( Fortunes )
      fortune
```

The subscript `fortune` refers to the column named fortune, and the regular text `Fortunes` refers to the table. It is equivalent to the following SQL:

```sql
SELECT fortune FROM fortunes;
```

Generally, we don't capitalize table names or column names in SQL, but we do like to YELL! The result of the projection above would be:
```plaintext
+----------------------------------+
| fortune                          |
+----------------------------------+
| You will die a terrible death.   |
+----------------------------------+
| You will live a good life.       |
+----------------------------------+
| You're going to be hungry.       |
+----------------------------------+
| You're going to be alright.      |
+----------------------------------+
```

### σ Selection

```plaintext
     σ ( Fortunes )
      id=1
```

Selection is about selection criteria, or conditional statements. You can think of it as a filter. It is equivalent to the following SQL:

```
SELECT * FROM fortunes WHERE id = 1;
```

And yes, annoyingly, the "Selection" operator deals with the Where clause, and the "Projection" operator deals with the select clause.

Here's the data that would match from the fortunes table:

```plaintext
+--------+-------------------+--------------+-------------+
| id     | fortune                          | author_id   |
+---------------------------------------------------------+
| 1      | You will die a terrible death.   | 1           |
+--------+------------------------------------------------+
```

### X Cartesian Product
```plaintext
     Fortunes X Authors
```
A cartesian product combines two sets of data; the above statement combines the Fortunes and Authors table. The equivalent SQL is:
```sql
SELECT * FROM fortunes, authors;
```

Here's the data produced:

```plaintext
+------------+---------------------------------+-------------------+-----------+-------------+
| fortune.id | fortune.fortune                 | fortune.author_id | author.id | author.name |
+--------------------------------------------------------------------------------------------+
| 1          | You will die a terrible death.  | 1                 | 1         | Joe         |
+--------------------------------------------------------------------------------------------+
| 1          | You will die a terrible death.  | 1                 | 2         | Mortimer    |
+--------------------------------------------------------------------------------------------+
| 1          | You will die a terrible death.  | 1                 | 3         | Frank       |
+--------------------------------------------------------------------------------------------+
| 1          | You will die a terrible death.  | 1                 | 4         | Mindy       |
+--------------------------------------------------------------------------------------------+
| 1          | You will die a terrible death.  | 1                 | 5         | Sue         |
+--------------------------------------------------------------------------------------------+
| 2          | You will live a good life.      | 2                 | 1         | Joe         |
+--------------------------------------------------------------------------------------------+
| 2          | You will live a good life.      | 2                 | 2         | Mortimer    |
+--------------------------------------------------------------------------------------------+
| 2          | You will live a good life.      | 2                 | 3         | Frank       |
+--------------------------------------------------------------------------------------------+
| 2          | You will live a good life.      | 2                 | 4         | Mindy       |
+--------------------------------------------------------------------------------------------+
| 2          | You will live a good life.      | 2                 | 5         | Sue         |
+--------------------------------------------------------------------------------------------+
| 3          | You're going to be hungry.      | 2                 | 1         | Joe         |
+--------------------------------------------------------------------------------------------+
| 3          | You're going to be hungry.      | 2                 | 2         | Mortimer    |
+--------------------------------------------------------------------------------------------+
| 3          | You're going to be hungry.      | 2                 | 3         | Frank       |
+--------------------------------------------------------------------------------------------+
| 3          | You're going to be hungry.      | 2                 | 4         | Mindy       |
+--------------------------------------------------------------------------------------------+
| 3          | You're going to be hungry.      | 2                 | 5         | Sue         |
+--------------------------------------------------------------------------------------------+
| 4          | You're going to be alright.     | 5                 | 1         | Joe         |
+--------------------------------------------------------------------------------------------+
| 4          | You're going to be alright.     | 5                 | 2         | Mortimer    |
+--------------------------------------------------------------------------------------------+
| 4          | You're going to be alright.     | 5                 | 3         | Frank       |
+--------------------------------------------------------------------------------------------+
| 4          | You're going to be alright.     | 5                 | 4         | Mindy       |
+--------------------------------------------------------------------------------------------+
| 4          | You're going to be alright.     | 5                 | 5         | Sue         |
+--------------------------------------------------------------------------------------------+
```

This is about as good a time as any to discuss **cardinality**, which refers to the number of elements in a collection. In relational algebra, it is denoted by the pipe character (`|`) surrounding the table name.
```
| Authors | = 5
| Fortunes | = 4
| Authors X Fortunes | = | Authors | * | Fortunes | = 20
```

As you can imagine, this cardinality explodes as one joins more and more tables together. However, why would you ever join tables together like this? It's probably fair to say that if we were looking at the Authors table and the Fortunes table together, we would likely be interested in getting the author information for a given fortune. Fortunately, we can combine the tools we already have to make a more useful data set:

```plaintext
σ (Authors X Fortunes)
 fortunes.author_id = authors.id
```

This yields:

```plaintext
+------------+---------------------------------+-------------------+-----------+-------------+
| fortune.id | fortune.fortune                 | fortune.author_id | author.id | author.name |
+--------------------------------------------------------------------------------------------+
| 1          | You will die a terrible death.  | 1                 | 1         | Joe         |
+--------------------------------------------------------------------------------------------+
| 2          | You will live a good life.      | 2                 | 2         | Mortimer    |
+--------------------------------------------------------------------------------------------+
| 3          | You're going to be hungry.      | 2                 | 2         | Mortimer    |
+--------------------------------------------------------------------------------------------+
| 4          | You're going to be alright.     | 5                 | 5         | Sue         |
+--------------------------------------------------------------------------------------------+
```

For more on Relational Algebra, there are a ton of resources on the World Wide Web! [Wikipedia](https://en.wikipedia.org/wiki/Relational_algebra) is a good jumping-off point.

## Putting it Together

Given entities and relational algebra, you can build queries to furnish your app with all the data it needs. Rather than storing a flat document with everything pre-built, you instead store the recipe to produce that document given your entities. As it turns out, computers are good at this kind of stuff, and so it is still blazingly fast most of the time. When it isn't, 95% of the time there is a simple answer on stack overflow, and for the other 5%, having an expert DBA on staff could be helpful.

It's worth noting that fetching of a flat document will still be faster, as fetching from a cache of frequently used data can happen in nearly constant time. However, these two systems are not incompatible - you can have your frontend hit the cache for the result of a recipe, but have the cache be populated by a periodic refresh job on the backend. Some databases even do this for you automatically via mechanisms like materialized views. Further, a lot of relational databases have first-class support for `JSON`, allowing you to enjoy the best of both worlds - having a schemaless JSON column for arbitrary data, and other columns for the more well structured data. This also showcases that you can start out with simple (id, JSON) tables, then over time copy the data from the JSON column into separate columns, perhaps with indexes, as needed.

Some other helpful axioms provided by these concepts:
- If you update a single entity, you only need to update it in one place.
- When you read a single entity, it is guaranteed to be consistent, even if it is referenced in multiple places.
- If you change an entity's structure, any issues with the new structure tend to show up when creating the schema, rather than showing up later when reading or writing the data. This kind of static analysis saves a lot of grief over time, just like types in programming languages.
- If you try to write weird data to a table, the database notices and complains, rather than writing the data anyway and compromising the integrity of the table. The earlier the failure, the less painful it is, as the alternative would be noticing after thousands of entries have been written, and finding out a way to untangle that mess.
