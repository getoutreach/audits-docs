# Elasticsearch Index Options

Audit events are heterogeneous. This poses the challenge of how to index them. There are 3 options that I can think of:

## Flatten the data

We could flatten and denormalize event-specific `payload` from all events into a single table

### Pros

* simple to implement
* easy to connect to from the API
* easy to filter/query
* easy to run cross-type queries

### Cons

* always have to have a `type` filter
* hard to debug
* index will be potentially really wide
* if the index gets wide it will also become sparse
* harder to surface type-doesn't-have-requested-field errors
* have to update the entire index on fiels addition/removal
 
## Nest the payload

This is almost the same thing as flattening the data. We still have a single table, but nest the event-specific `payload` into an [object](https://www.elastic.co/guide/en/elasticsearch/reference/current/object.html) type field

### Pros

The pros are moslty same as above:

* simple to implement
* easy to connect to from the API
* easy to filter/query
* maybe able run cross-type queries
* may be easier to surface type-doesn't-have-requested-field errors

### Cons

I'm uncertain if it's possible to actually have differently shaped objects or if we'll just have a flat dataset, but nested. 

If objects *cannot* vary in shape per record then cons are the same as flat:

* always have to have a `type` filter
* hard to debug
* index will be potentially really wide
* if the index gets wide it will also become sparse
* harder to surface type-doesn't-have-requested-field errors
* *but also*: it's now weirdly nested
* have to update the entire index on fiels addition/removal

If objects *can* vary in shape:

* still hard to debug
* unknown performance implications
* have to update the entire index on fiels addition/removal

## Index Per Type

Finally, we can create a dedicated index per event type. 

### Pros

* necessarily small indices
* necessarily dense(opposite of sparse?) indices
* type-filtering is no longer an issue
* faster due to size and low individual index complexity
* easy to debug/maintain
* a type appeating/disappearing/changing only impacts its index and none of the others

### Cons

* single call cross-type queries are impossible
* API has to know what index to query

## Opinion

Please weigh in on what you think we should do:

*Greg*: I propose we go with the last option as it seems to be the most structurally sound and maintainable. Yes, cross-type querying is not possible, but i'm not sure that's a thing we will ever need. 
