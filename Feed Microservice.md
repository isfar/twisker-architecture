# Feed Microservice

## Responsibilities

This service is responsible for realtime `Feed` generations and retrievals. This service listens on Kafka events/topic via long polling and in response, genereates feeds for orgs/groups/user in realtime.


## Data Storage

We will be using `NoSQL` `key-value` storage database like `MongoDB` for this as the fields may change and we can shard the database across network of storage servers.

The `Feed` Collection represents the feed for a `org`, `group` or `user `. A feed is a collection of `Story`s. In the `stories` key, we store story for a group, org or user. Stories are indexed based on `at` value in a `descending` order.


### Collections

#### Feed

```javascript
{
  owner: {
    type: 'group', // 'user', 'org' etc
    id: 98333,
  },
  stories: [
    {
      storyId: 34343,
      at: ""
    },
    {
      storyId: 34342,
      at: ""
    },
    {
      storyId: 34341,
      at: ""
    },
    {
      storyId: 34311,
      at: ""
    },
    {
      storyId: 34300,
      at: ""
    },
    {
      storyId: 34200,
      at: ""
    },
  ]
}
```

#### Story

```javascript
{
  type: 'timeline.post.created',
  post: {
    id: 0908645337,
    createdAt: "",
    createdBy: 9967443367,
  },
  at: ""
},
```

```javascript
{
  type: 'timeline.post.title.renamed',
  post: {
    previousTitle: "",
    title: "",
    renamedAt: "",
    renamedBy: "",
  },
  at: ""
},
```

```javascript
{
  type: 'timeline.post.comment.created,
  post: {
    id: 0908645337,
    createdBy: 9967443367,
    createdAt: "",
  },
  comment: {
    id: 7766545656,
    content: ""
  },
  at: ""
},
```

### Queries

The queries will be trivial read-writes. So the queries are ommitted for this doc.
