---
title: Multi-dimensional indexes
menuTitle: Multi-dimensional Indexes
weight: 25
description: >-
  Multi-dimensional indexes allow you to index two- or higher-dimensional data
  such as time ranges, for efficient intersection of multiple range queries
---
A multi-dimensional index maps multi-dimensional data in the form of multiple
numeric attributes to one dimension while mostly preserving locality so that
similar values in all of the dimensions remain close to each other in the mapping
to a single dimension. Queries that filter by multiple value ranges at once can
be better accelerated with such an index compared to a persistent index.

The multi-dimensional index type is called `zkd`.

{{< warning >}}
`zkd` indexes are an **experimental** feature.
{{< /warning >}}


The `fields` property of a `zkd` index describes which document attributes to
use as dimensions. It is required that the attributes are present and have
numeric values.

Multi-dimensional indexes can be declared as `unique` to only allow a single
document with a given combination of attribute values, using all of the `fields`
attributes.

## Querying documents within a 3D box

Assume we have documents in a collection `points` of the form

```json
{"x": 12.9, "y": -284.0, "z": 0.02}
```

and we want to query all documents that are contained within a box defined by
`[x0, x1] * [y0, y1] * [z0, z1]`.

To do so one creates a multi-dimensional index on the attributes `x`, `y` and
`z`, e.g. in _arangosh_:

```js
db.collection.ensureIndex({
    type: "zkd",
    fields: ["x", "y", "z"],
    fieldValueTypes: "double"
});
```

Unlike with other indexes, the order of the `fields` does not matter.

`fieldValueTypes` is required and the only allowed value is `"double"` to use a
double-precision (64-bit) floating-point format internally.

Now we can use the index in a query:

```aql
FOR p IN points
  FILTER x0 <= p.x && p.x <= x1
  FILTER y0 <= p.y && p.y <= y1
  FILTER z0 <= p.z && p.z <= z1
  RETURN p
```

## Possible range queries

Having an index on a set of fields does not require you to specify a full range
for every field. For each field you can decide if you want to bound
it from both sides, from one side only (i.e. only an upper or lower bound)
or not bound it at all.

Furthermore you can use any comparison operator. The index supports `<=` and `>=`
naturally, `==` will be translated to the bound `[c, c]`. Strict comparison
is translated to their non-strict counterparts and a post-filter is inserted.

```aql
FOR p IN points
  FILTER 2 <= p.x && p.x < 9
  FILTER p.y >= 80
  FILTER p.z == 4
  RETURN p
```

## Example Use Case

If you build a calendar using ArangoDB you could create a collection for each user
that contains the appointments. The documents would roughly look as follows:

```json
{
  "from": 345365,
  "to": 678934,
  "what": "Dentist",
}
```

`from`/`to` are the timestamps when an appointment starts/ends. Having an
multi-dimensional index on the fields `["from", "to"]` allows you to query
for all appointments within a given time range efficiently.

### Finding all appointments within a time range

Given a time range `[f, t]` we want to find all appointments `[from, to]` that
are completely contained in `[f, t]`. Those appointments clearly satisfy the
condition

```
f <= from and to <= t
```

Thus our query would be:

```aql
FOR app IN appointments
    FILTER f <= app.from
    FILTER app.to <= t
    RETURN app
```

### Finding all appointments that intersect a time range

Given a time range `[f, t]` we want to find all appointments `[from, to]` that
intersect `[f, t]`. Two intervals `[f, t]` and `[from, to]` intersect if
and only if

```
f <= to and from <= t
```

Thus our query would be:

```aql
FOR app IN appointments
  FILTER f <= app.to
  FILTER app.from <= t
  RETURN app
```

## Lookahead Index Hint

<small>Introduced in: v3.10.0</small>

Using the lookahead index hint can increase the performance for certain use
cases. Specifying a lookahead value greater than zero makes the index fetch
more documents that are no longer in the search box, before seeking to the
next lookup position. Because the seek operation is computationally expensive,
probing more documents before seeking may reduce the number of seeks, if
matching documents are found. Please keep in mind that it might also affect
performance negatively if documents are fetched unnecessarily.

You can specify the `lookahead` value using the `OPTIONS` keyword:

```aql
FOR app IN appointments OPTIONS { lookahead: 32 }
    FILTER @to <= app.to
    FILTER app.from <= @from
    RETURN app
```

## Limitations

Currently there are a few limitations:

- Using array expansions for attributes is not possible (e.g. `array[*].attr`)
- The `sparse` property is not supported.
- You can only index numeric values that are representable as IEEE-754 double.
- A high number of dimensions (more than 5) can impact the performance considerably.
- The performance can vary depending on the dataset. Densely packed points can
  lead to a high number of seeks. This behavior is typical for indexing using
  space filling curves.
