---
layout: post
title:  "Intro to SPARQL a graph database query language"
date:   2018-03-18 12:00:00 +0100
categories: sparql database
description: An Introduction to the SPARQL query language, a SQL-like way to query RDF triple store graph databases.
---

SPARQL is a W3C standard to process graph data in the RDF format (W3C standardized data model on directed graphs) with an SQL-like syntax.
The data is represented as triples that follow this standard format: `<subject> <predicate> <object>`. A collection of these triples creates a graph (subject and object are nodes and predicate is an edge).

Suppose we have this RDF data at the url: `http://example.org/graph`
```sparql
@prefix  foaf:  <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name     "Enrico" .
```
We can then use the following SPARQL query to get the name of the subject:

```sparql
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT  ?name
FROM    <http://example.org/graph>
WHERE   { ?x foaf:name ?name . }
```

Let's break down what everything means:
In both data and query we use a prefix of `foaf` that is located at the given url. This is just a collection of named properties and classes using RDF and it defines our `foaf:name` predicate in the WHERE clause. In our data we then have a line containing a RDF triple. The subject at the beginning of the line is a so called blank node and acts as a variable. So our data means something like "there is an object and it's name is Enrico".

The query now uses a triple in the WHERE clause to pattern match the data. Anything with a question mark in front of it is a variable which means our WHERE clause roughly translates to "any_object foaf:name any_name" and that matches with the triple in our dataset. With our SELECT clause we then tell it to forget about our `?x` variable and we only get the name back.

SPARQL also supports other SQL features like ORDER BY, GROUP BY, LIMIT, OFFSET, HAVING, SELECT DISTINCT, COUNT, ... those all work exactly the same as in SQL the only difference is that you have to remember to use a `?` in front of the variable (which in SQL would be a column). So for example: `ORDER BY ?age`. So SPARQL should be relatively easy to pick up for people who already know SQL.

But SPARQL supports more than just a SELECT for queries. There is an ASK query which basically means "is there such a data triple?" and that returns just true or false.

```sparql
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
ASK { ?x foaf:name  "Enrico" . }
FROM    <http://example.org/graph>
```

The DESCRIBE query will return a RDF graph describing the found resource. However the data provider has to decide what could be useful to send here.
```sparql
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
DESCRIBE ?x
FROM    <http://example.org/graph>
WHERE { ?x foaf:name "Enrico" . }
```

The last one is the CONSTRUCT query and it basically creates a new RDF graph by substituting variables:

```sparql
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX vcard: <http://www.w3.org/2001/vcard-rdf/3.0#>
CONSTRUCT
 {  ?x  vcard:N _:v .
    _:v vcard:givenName ?gname .
    _:v vcard:familyName ?fname .
 }
FROM    <http://example.org/graph>
WHERE
 {
    ?x foaf:firstname ?gname .
    ?x foaf:surname   ?fname .
 }
```
That looks a bit more complicated, but what it does is it takes the firstname and surname of our foaf data and for every person found creates a new vcard object and puts the names in.
So speaking in SQL we do a SELECT and an INSERT with the selected data into another table.

In case you asked yourself why there is always a `.` at the end of each line in the WHERE clause, this is something called a Graph Pattern in SPARQL. The dot basically means "and".
By using `{A} UNION {B}` where A and B are a triple as we used them before or a group of them you can use an alternative, so either A or B has to be true. Then there is also `A Minus {B}` to only get those in A but not in B and `A OPTIONAL {B}` which is simply A and maybe B. By using curly braces you can group the different clauses. An example for Graph Patterns could be:

```sparql
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?name ?mbox
FROM    <http://example.org/graph>
WHERE {
  ?x foaf:name  ?name .
  OPTIONAL { ?x  foaf:mbox  ?mbox }
}
```
Which would give you the name of an object just like before, and if that object also has an email address that would be in the result set too, if it doesn't that's ok since it's optional.


Which brings us to the last point of this post: Property Paths. Until now we only ever used one edge in our graph (remember the middle thing in our triple is an edge in our graph).
With Property Paths we can walk multiple edges in our graph at once. So if we have a lot of data in this format and try the following SPARQL query:

```sparql
_:a  foaf:name     "Enrico" .
_:a  foaf:knows    _:b .
_:b  foaf:name     "Bob" .
...
```

```sparql
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?name
FROM    <http://example.org/graph>
WHERE {
  ?x foaf:name "Enrico" .
  ?x foaf:knows/foaf:knows/foaf:name ?name .
}
```

A `/` means sequence so our query first takes the `foaf:knows` edge and from the object it ends up at it takes another `foaf:knows` edge and from there a `foaf:name` edge. Resulting in a list of names of people that are reachable by using 2 times a `foaf:knows` edge. There are also a lot of different operators not only a simple sequence. For example a `|` would give you an alternative, a `+` a "one or more" of that edge and a lot more. This can get pretty complicated quickly.

That's only a short overview on how to use SPARQL to SELECT data. There is also a "SPARQL Update language" containing INSERT DATA, DELETE DATA and more in case you want to dive deeper.
