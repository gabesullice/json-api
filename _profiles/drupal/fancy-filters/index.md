---
name: Resource Versioning
short_description: |
  Defines a syntax for filters.

extended_description: |
  # Overview
  This profile establishes a rich syntax for constructing powerful
  client-driven filters.

minimum_jsonapi_version: 1.0
minimum_jsonapi_version_explanation:

discussion_url: https://www.drupal.org/project/issues/jsonapi

editors:
  - name: Gabriel Sullice
    email: gabriel@sullice.com
  - name: Mateu Aguiló Bosch
    email: mateu@mateuaguilo.com
    website: https://mateuaguilo.com
  - name: Wim Leers
    email: work@wimleers.com
    website: https://wimleers.com

categories:
  - Filtering
---
  
# Concepts
By default, collection endpoints often return every resource object of a
particular type available a server. It is usually necessary to limit these
responses to a subset of available resource objects based on client-defined
parameters. The JSON:API specification stipulates that this filtering be done
via [the `filter` query parameter](https://jsonapi.org/format/#fetching-filtering). However, it is agnostic about the strategy used.
This profile establishes one such strategy.

Fancy filters are constructed of two primary objects, _conditions_ and _groups_.

A _condition_ is a logical predicate composed of a _path_, an _operator_ and a _value_.
A condition can be thought of as a test that each resource object must pass in
order to be included a filtered collection response.
    
A condition can be read as "the `{path}` attribute of the resource object
must be `{operator}` `{value}`. For example, "the `title` attribute of the
resource object must be equal to 'Hello, world!'".

A _group_ can compose conditions into larger predicates using a logical conjuntion
or logical disjunction, these are typically written as `AND` and `OR`
respectively. Groups can also compose sub-groups into even more complex
predicates. 

# Query Parameter

This profile establishes a syntax for the entire `filter` [query parameter family](https://jsonapi.org/format/1.1/#query-parameters-families).

A server **MUST** process every query parameter with the `filter` base name
according to this specification when this profile is applied.

## Format

A `filter` query parameter can have up to four components.

```
  1st component
        |   3rd component
       _|_       _|_
      /   \     /   \
filter[foo][bar][baz][]=qux <-- parameter value
           \___/     \/
             |        \
       2nd component   4th component
```

Query parameters that share their 1st component are considered to be part of a
single _filter object_. For example, the following query string has one filter
object named `foo`.

```
?filter[foo][bar]=baz&filter[foo][qux]=quux
```

A filter object **MUST NOT** have only two components.

A filter object **MUST NOT** have more than four components.

The 2nd component of a `filter` query parameter **MUST** be either `[condition]` or
`[group]`. This component identifies the filter object type. When a `filter`
query parameter has only one component, its type **MUST** be interpreted as a
condition filter object.

## Processing Condition Filter Objects

The 3rd component of a condition filter object **MUST** be one of: `[path]`,
`[operator]`, `[value]` or `[memberOf]`.

A condition filter object with a query parameter that has only one component
**MUST NOT** contain more than one query parameter.

A condition filter object **MUST** contain a query parameter with a `[path]`
component unless it has only one query parameter and that query parameter has
only one component.

A condition filter object **MUST** contain a query parameter with an `[operator]`
component unless it contains only one query parameter and that query parameter
contains a `[value]` component or that query parameter has only one component.

A condition filter object with a query parameter object containing an `[operator]`
component **MUST** have a parameter value of:
  
  - `=`
  - `<>`
  - `>`
  - `>=`
  - `<`
  - `<=`
  - `STARTS_WITH`
  - `CONTAINS`
  - `ENDS_WITH`
  - `IN`
  - `NOT IN`
  - `BETWEEN`
  - `NOT BETWEEN`
  - `IS NULL`
  - `IS NOT NULL`

> The above values are shown unencoded simply for readability. In practice,
> these characters should be percent-encoded.

A condition filter object query parameter with more than one component **MUST**
have three components unless that query parameter contains a `[value]` component.

A condition filter object query parameter that contains a `[value]` component
**MUST NOT** have four components unless that condition filter object has a query
parameter that contains an `[operator]` component whose parameter value is `IN`,
`NOT IN`, `BETWEEN` or `NOT BETWEEN`. In that case, the `[value]` query parameter **MUST**
have `[]` as its 4th component.

A condition filter object with a query parameter that contains an `[operator]`
component whose parameter value is `IS NULL` or `IS NOT NULL` **MUST NOT** have a
query parameter that contains a `[value]` component.

A condition filter object that does not contain a query parameter that contains
an `[operator]` component **MUST** be evaluated for equality.

A condition filter object that contains a query parameter with only one
component **MUST** be evaluated with a path equal to the string within the square
brackets of the that query parameter's 1st component.

### Paths

A filter path is a dot-separated (U+002E FULL-STOP, “.”) list of field names,
keys of field objects, `meta` object member names or object keys of objects
within a `meta` object.

If a path segment is a relationship field name, it **MUST** be followed by the
string `meta` and then a `meta` object member name that's valid for the
resource identifier objects of that relationship field or a field name that is
valid for the resource objects that can be identified by resource identifier
objects of that relationship field.

If a path segment is an attribute field name, it **MUST** either be the last segment
in the path or it **MUST** be followed by a valid object key for that field.

The string `meta` **MUST NOT** be the first or last segment in a filter path and it
**MUST** follow a valid relationship field name.

If a path segment is equal to the string `meta`, then it **MUST** be followed by an
object key that is valid for a resource identifier object's `meta` object for
the relationship field identified by the preceding relationship field name in
that path.

A server **MAY** limit the number of allowed field names to any positive value.

<a href="unsupported-filter-path"></a>If a server encounters a path that is does not support, it **MUST** respond with a
`400 Bad Request` and an error object with a `type` of
`https://jsonapi.org/profiles/drupal/fancy-filters/unsupported-filter-path`.

<a href="invalid-filter-path"></a>If a server encounters an invalid path, it **MUST** respond with a
`400 Bad Request` and an error object with a `type` of
`https://jsonapi.org/profiles/drupal/fancy-filters/invalid-filter-path`.

## Processing Group Filter Objects

When the 2nd component of a filter is `[group]`, the 3rd component **MUST** be either
`[conjunction]` or `[memberOf]`.

## Server Responsibilities

TBD

## Client Responsibilities

TBD
