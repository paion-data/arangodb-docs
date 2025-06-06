---
title: Match documents with `FILTER`
menuTitle: Match documents
weight: 10
---
So far, you either looked up a single document, or returned the entire character
collection. For the lookup, you used the `DOCUMENT()` function, which means you
can only find documents by their key or ID.

To find documents that fulfill certain criteria more complex than key equality,
there is the `FILTER` operation in AQL, which enables you to formulate arbitrary
conditions for documents to match.

## Equality condition

```aql
FOR c IN Characters
  FILTER c.name == "Ned"
  RETURN c
```

The filter condition reads like: "the `name` attribute of a character document
must be equal to the string `Ned`". If the condition applies, character
document gets returned. This works with any attribute likewise:

```aql
FOR c IN Characters
  FILTER c.surname == "Stark"
  RETURN c
```

## Range conditions

Strict equality is one possible condition you can state. There are plenty of
other conditions you can formulate, however. For example, you could ask for all
adult characters:

```aql
FOR c IN Characters
  FILTER c.age >= 13
  RETURN c.name
```

```json
[
  "Joffrey",
  "Tyrion",
  "Samwell",
  "Ned",
  "Catelyn",
  "Cersei",
  "Jon",
  "Sansa",
  "Brienne",
  "Theon",
  "Davos",
  "Jaime",
  "Daenerys"
]
```

The operator `>=` stands for *greater-or-equal*, so every character of age 13
or older is returned (only their name in the example). You can return names
and age of all characters younger than 13 by changing the operator to
*less-than* and using the object syntax to define a subset of attributes to
return:

```aql
FOR c IN Characters
  FILTER c.age < 13
  RETURN { name: c.name, age: c.age }
```

```json
[
  { "name": "Tommen", "age": null },
  { "name": "Arya", "age": 11 },
  { "name": "Roose", "age": null },
  ...
]
```

You may notice that it returns name and age of 30 characters, most with an
age of `null`. The reason for this is, that `null` is the fallback value if
an attribute is requested by the query, but no such attribute exists in the
document, and the `null` is compares to numbers as lower (see
[Type and value order](../../aql/fundamentals/type-and-value-order.md)). Hence, it
accidentally fulfills the age criterion `c.age < 13` (`null < 13`).

## Multiple conditions

To not let documents pass the filter without an age attribute, you can add a
second criterion:

```aql
FOR c IN Characters
  FILTER c.age < 13
  FILTER c.age != null
  RETURN { name: c.name, age: c.age }
```

```json
[
  { "name": "Arya", "age": 11 },
  { "name": "Bran", "age": 10 }
]
```

This could equally be written with a boolean `AND` operator as:

```aql
FOR c IN Characters
  FILTER c.age < 13 AND c.age != null
  RETURN { name: c.name, age: c.age }
```

And the second condition could as well be `c.age > null`.

## Alternative conditions

If you want documents to fulfill one or another condition, possibly for
different attributes as well, use `OR`:

```aql
FOR c IN Characters
  FILTER c.name == "Jon" OR c.name == "Joffrey"
  RETURN { name: c.name, surname: c.surname }
```

```json
[
  { "name": "Joffrey", "surname": "Baratheon" },
  { "name": "Jon", "surname": "Snow" }
]
```

See more details about [Filter operations](../../aql/high-level-operations/filter.md).
