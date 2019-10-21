# Authentication & Authorization Microservice

<!-- toc -->

- [Responsibilities](#Responsibilities)
- [Data Storage](#Data-Storage)
- [Message Broker (Apache Kafka) Events](#Message-Broker-Apache-Kafka-Events)
- [Permission Map](#Permission-Map)
- [Data Model](#Data-Model)
  * [Timeline](#Timeline)
  * [TimelineOwners](#TimelineOwners)
  * [Collections](#Collections)
  * [CollectionUsers](#CollectionUsers)
  * [Posts](#Posts)
  * [Comments](#Comments)
  * [PrivacyCollectionRoles](#PrivacyCollectionRoles)
  * [Privacies](#Privacies)
  * [Roles](#Roles)
  * [Users](#Users)
  * [Groups](#Groups)
  * [Orgs](#Orgs)
  * [UserGroups](#UserGroups)
- [API](#API)
  * [Sample Request](#Sample-Request)
  * [Sample Response](#Sample-Response)
- [Queries](#Queries)
  * [Write](#Write)
  * [Read](#Read)
    + [Use-case 1](#Use-case-1)
    + [Use-case 2](#Use-case-2)
    + [Use-case 3](#Use-case-3)

<!-- tocstop -->


## Responsibilities

- **Authentication**: JWT Authentication (with Private Key signed HMAC).
- **Authorization**
  - Check if an user is authorized to take an action on a particular resource, i.e., posts, comments etc.
  - Get all the comment id's, post id's etc for a given timeline id or post id that a user has permission to view.  


## Data Storage

We will by using `SQL` database for this service as we will be requiring many `JOIN` queries and `transaction`s. We will be storing only *authorization* related data, nothing more than that. All the columns will be `indexed` and all the string value fields (e.g.: `customeList`, `author` etc) will be of `ENUM` type, so that queries run faster.
  

## Message Broker (Apache Kafka) Events

- `user.created`
- `user.edited`
- `group.created`
- `group.edited`
- `org.created`
- `org.edited`
- `timeline.created`
- `timeline.edited`
- `post.created`
- `post.edited`
- `comment.created`
- `comment.edited`
...


## Permission Map

- `users`: Any `user` from any `group` from any `org`.
- `timeline.author`: The `user` who crates a `timeline`.
- `post.author`: The `user` who creates a `post` on a `timeline`.
- `group`: A `group`.
- `org`: An `organization`.
- `group.users`: Users that belong to that `group`.
- `org.users`: Users that belong to that `org`.
- `timeline.contributors`: `contributor`s for a `timeline`.
- `timeline.commenters`: `commenters`s for a `timeline`.
- `timeline.viewers`: `viewers`s for a `timeline`.
- `post.visiblity`: `visibility` of a `post`.

> A user can add a group, org or a list of user(s) as `contributor`, `commenter` , ` viewer` etc!

<table>
<thead>
  <tr>
    <th rowspan="2"><code>timeline.privacy</code>&darr;</th>
    <th rowspan="2"><code>timeline.owner</code></th>
    <th><code>permission</code></th>
    <th rowspan="2"><code>post.visibility</code></th>
    <th colspan="3"><code>permission</code></th>
  </tr>
  <tr>
    <th><code>post.create</code></th>
    <th><code>post.view</code></th>
    <th><code>post.comment.create</code></th>
    <th><code>post.comment.view</code></th>
  </tr>
</thead>
<tbody>
  <!-- >>> timeline.privacy: public -->
  <tr>
    <td rowspan="6"><code>PUBLIC</code></td>
    <td rowspan="2"><code>author</code></td>
    <td rowspan="2"><code>users</code></td>
    <td><code>visible</code></td>
    <td><code>users</code></td>
    <td rowspan="2"><code>users</code></td>
    <td><code>users</code></td>
  </tr>
  <tr>
    <td><code>hidden</code></td>
    <td>
      <code>post.author</code><br>
      <code>timeline.author</code>
    </td>
    <td>
      <code>post.author</code><br>
      <code>timeline.author</code>
    </td>
  </tr>
  <tr>
    <td rowspan="2"><code>group</code></td>
    <td rowspan="2"><code>group.users</code></td>
    <td><code>visible</code></td>
    <td><code>group.users</code></td>
    <td rowspan="2"><code>group.users</code></td>
    <td><code>group.users</code></td>
  </tr>
  <tr>
    <td><code>hidden</code></td>
    <td>
      <code>post.author</code><br>
      <code>timeline.author</code>
    </td>
    <td>
      <code>post.author</code><br>
      <code>timeline.author</code>
    </td>
  </tr>
  <tr>
    <td rowspan="2"><code>org</code></td>
    <td rowspan="2"><code>org.users</code></td>
    <td><code>visible</code></td>
    <td><code>org.users</code></td>
    <td rowspan="2"><code>org.users</code></td>
    <td><code>org.users</code></td>
  </tr>
  <tr>
    <td><code>hidden</code></td>
    <td>
      <code>post.author</code><br>
      <code>timeline.author</code>
    </td>
    <td>
      <code>post.author</code><br>
      <code>timeline.author</code>
    </td>
  </tr>
  <!-- <<< timeline.privacy: public -->
  <!-- >>> timeline.privacy: privateToOrg -->
  <tr>
    <td rowspan="2"><code>PRIVATE_TO_ORG</code></td>
    <td rowspan="2"></td>
    <td rowspan="2"><code>org.users</code></td>
    <td><code>visible</code></td>
    <td><code>org.users</code></td>
    <td rowspan="2">
      <code>timeline.author</code>
      <code>timeline.contributors</code><br>
      <code>timeline.commenters</code>
    </td>
    <td rowspan="2"><code>org.users</code></td>
  </tr>
  <tr>
    <td><code>hidden</code></td>
    <td>
      <code>post.author</code><br>
      <code>timeline.author</code>
    </td>
  </tr>
  <!-- <<< timeline.privacy: privateToOrg -->
  <!-- >>> timeline.privacy: privateToGroup -->
  <tr>
    <td rowspan="2"><code>PRIVATE_TO_GROUP</code></td>
    <td rowspan="2"></td>
    <td rowspan="2"><code>group.users</code></td>
    <td><code>visible</code></td>
    <td><code>group.users</code></td>
    <td rowspan="2">
      <code>timeline.author</code><br>
      <code>timeline.contributors</code><br>
      <code>timeline.commenters</code>
    </td>
    <td rowspan="2"><code>group.users</code></td>
  </tr>
  <tr>
    <td><code>hidden</code></td>
    <td>
      <code>post.author</code><br>
      <code>timeline.author</code>
    </td>
  </tr>
  <!-- <<< timeline.privacy: privateToGroup -->
  <!-- >>> timeline.privacy: private -->
  <tr>
    <td rowspan="2"><code>PRIVATE</code></td>
    <td rowspan="2"></td>
    <td rowspan="2"><code>timeline.author</code></td>
    <td><code>visible</code></td>
    <td rowspan="2"><code>timeline.author</code></td>
    <td rowspan="2"><code>timeline.author</code></td>
    <td rowspan="2"><code>timeline.author</code></td>
  </tr>
  <tr>
    <td><code>hidden</code></td>
  </tr>
  <!-- <<< timeline.privacy: private -->
  <!-- >>> timeline.privacy: privateToUsers -->
  <tr>
    <td rowspan="2"><code>PRIVATE_TO_USERS</code></td>
    <td rowspan="2"></td>
    <td rowspan="2">
      <code>timeline.author</code><br>
      <code>timeline.contributors</code><br>
    </td>
    <td><code>visible</code></td>
    <td rowspan="2">
      <code>timeline.author</code><br>
      <code>timeline.contributors</code><br>
      <code>timeline.commenters</code><br>
      <code>timeline.viewers</code><br>
    </td>
    <td rowspan="2">
      <code>timeline.author</code><br>
      <code>timeline.contributors</code><br>
      <code>timeline.commenters</code><br>
    </td>
    <td rowspan="2">
      <code>timeline.author</code><br>
      <code>timeline.contributors</code><br>
      <code>timeline.commenters</code><br>
      <code>timeline.viewers</code><br>
    </td>
  </tr>
  <tr>
    <td><code>hidden</code></td>
  </tr>
  <!-- <<< timeline.privacy: private -->
</tbody>
</table>


## Data Model

### Timeline

- `timelineId`
- `authorId` references `userId` from `Users`
- `ownerId` references `ownerId` from `Owners`.
- `privacyId` references `privacy` from `Privacies`. 

| `timelineId` | `authorId` | `ownerId` | `privacyId` |
| ------------ | ---------- | --------- | ----------- |
| 1            | 1          | 2         | 2           |
| 3            | 4          | 4         | 1           | 


### TimelineOwners

A timeline `owner` can be either of the `author`, the `group` or the `org`. 

- `ownerId`
- `ownerType`: If `ownerType` is `group` or `org`, then it references the `collectionId` from `Collections`. If it is `author`, then, the `referenceId` is actually the `authorId`.

| `ownerId` | `ownerType` | `referenceId` | 
| --------- | ----------- | ------------- |
| 1         | `group`     | 4             |
| 2         | `org`       | 3             |
| 3         | `author`    | 5             |


### Collections

- A user collection maybe of 3 types: A `group`, an `org` or a `customList`. `referenceId` refers either `groupId` from `Groups` or `orgId` from `Orgs`. For a custom list the `referenceId` is `NULL`.

| `collectionId` | `type`       | `referenceId` |
| -------------- | ------------ | ------------- |
| 4              | `group`      | 2             |
| 2              | `org`        | 6             |
| 5              | `customList` | NULL          | 


### CollectionUsers

This table is for Many-to-Many relation between `Collections` and `Users`.

| `collectionId` | `userId` |
| -------------- | -------- |
| 4              | 1        |
| 4              | 2        |
| 2              | 4        |
| 1              | 1        |
| 3              | 2        |


### Posts

- `postId`
- `authorId`: The `user` who created that `post`.References `userId` from `Users`.
- `timelineId`: references `timelineId` from `Timelines`.
- `privacyId`: references `privacyId` from `Privacies`.
- `visibility`: It can be either of
  - `hidden`
  - `visible`

 | `postId` | `authorId` | `timelineId` | `privacyId` | `visiblity` |
 | -------- | ---------- | ------------ | ----------- | ----------- |
 | 1        | 4          | 3            | 1           | `visible`   | 
 
  
### Comments 

| `commentId` | `authorId` | `postId` | `privacyId` |
| ----------- | ---------- | -------- | ----------- |
| 1           | 1          | 1        | 1           |


### PrivacyCollectionRoles

This table represents the relation between a `UserCollection`, `Role` and `Privacy`. For instance, a `Timeline` has the `Privacy` with `privacyId` 1, it adds a `Collection` ,e.g., a custom list of users and assigned them the role `contributor`.

| `privacyId` | `collectionId` | `roleId` |
| ----------- | -------------- | -------- |
| 1           | 1              | 1        |
| 2           | 3              | 4        |
| 3           | 2              | 7        |

  
### Privacies

This table defines privacy for a `Timeline`, `Post` or a `Comment`.

- `id`
- `entityType`:
  - `Post`
  - `Timeline`
  - `Comment`
- `entityId` 
- `type`:
  - `PUBLIC`
  - `PRIVATE`
  - `PRIVATE_TO_ORG`
  - `PRIVATE_TO_GROUP`
  - `PRIVATE_TO_USERS`

| `privacyId` | `entityType` | `entityId` | `type`             |
| ----------- | ------------ | ---------- | ------------------ |
| 1           | `Timeline`   | 2          | `PRIVATE`          |
| 2           | `Comment`    | 1          | `PRIVATE_TO_ORG`   |
| 3           | `Post`       | 1          | `PRIVATE_TO_USERS` |


### Roles

| `roleId` | `name`                         |
| -------- | ------------------------------ |
| 1        | `timeline.author`              |
| 2        | `timeline.owner`               |
| 3        | `timeline.contributor`         |
| 4        | `timeline.contributor.private` |
| 5        | `post.author`                  |
| 6        | `post.commenter`               |
| 7        | `post.viewer`                  |
| 8        | `post.commenter.private`       |
| 9        | `comment.author`               |

    
### Users

- `id`
- `username`
- `email`
- `password`: A `bcrypt` salted hash of the user password.

| `id` | `username` | `email`            | `password`      |
| ---- | ---------- | ------------------ | --------------- |
| 1    | `isfar`    | `isfar@orunim.com` | `$2a$05$bvI...` |


### Groups

- `groupId`
- `orgId`: References `orgId` from `Orgs`
- `name`

| `groupId` | `orgId` | `name`  |
| --------- | ------- | ------- |
| 1         | 1       | 'dev'   |
| 2         | 1       | 'hr'    |
| 3         | 1       | 'sales' | 


### Orgs

- `orgId`
- `name`

| `orgId` | `name`    |
| ------- | --------- |
| 1       | 'twisker' |
| 2       | 'orunim'  | 


### UserGroups

| `userId` | `groupId` |
| -------- | --------- |
| 1        | 1         |
| 1        | 3         |
| 1        | 4         |
| 2        | 3         |


## API

### Sample Request

1.
```json
{
  action: 'getCommentIdsByPostId',
  postId: 3,
  userId: 1
}
```

2.
```json
{
  permission: `post.create`,
  userId: 1,
  timelineId: 3
}
```


### Sample Response

1.
```json
{
  commentIds: [ 1, 3, 4, 7, 34, 22, 43, 44 ]
}
```

2.
```json
{
  permission: 'denied',
  reason: 'This is some reason why the permission was denied'
}
```

> N.B.: The request-reponse formats, actions etc. will be mapped according to the conventions of *RESTful API* or *GraphQL* standards.


## Queries

### Write

The central concept in the authorization service is the roles a user get for a timeline or a post. We could pick individual users and give them roles for a given timeline, or, we could pick a group and give it a role for the timeline in question. 

Suppose, we are assigning a group as `contributor` for a given timeline. So, we first take the `groupId` from the `Groups` table and assign the corresponding `Collections`'s `collectionId` for the privacy of the timeline. If it's a custom list of users we are assigning roles for, then individual `Collections` rows are created. Likewise, we also store the roles like `timeline.author`, `post.author` etc so that we can query simply.

### Read

Read queries will be built dynamically as we do with *Query Builders*. It is entirely based on the [Permission Map](#Permission-Map). Some examples are given below.


#### Use-case 1 

Can a user view a comment on a post given`timeline.privacy.type` is `PUBLIC`, owned by `group` and `post.visibility` is `hidden`?

From the map, we can see that only `post.author` and `timeline.author` can view this. So the query will be-- 

> Parameters: 
> `$commentId`
> `$userId`

```sql
SELECT 1
FROM `Comments`
  LEFT JOIN `Pivacies` ON `Privacies.entityId` = `$post.postId`
  LEFT JOIN `PrivacyCollectionRoles`
    ON `Privacies.privacyId` = `PrivacyCollectionRoles.privacyId`
  LEFT JOIN `Roles`
    ON `PrivacyCollectionRoles.roleId` = `Role.roleId`
  LEFT JOIN `CollectionsUsers`
    ON `Privacies.collectionId` = `CollectionsUsers.collectionId`
WHERE
  `Comments.postId` = `$commentId`
  `CollectionUsers.userId` = `$userId` AND
  `Privacies.entityType` = 'Comment' AND
  ( 
    `Roles.name` = 'timeline.author' OR 
    `Roles.name` = 'post.author'
  );
```

#### Use-case 2

Can a user create a comment on a `post` that is `hidden` and privacy is `PRIVATE_TO_ORG`? We see from the permission map that `timeline.author`, `post.author`, `timeline.contributors` and `timeline.commenters` have that permission. 

> Parameters: `$postId`, `$userId`
 
```sql
SELECT 1
FROM `Posts`
  LEFT JOIN `Timelines` ON `Posts.timelineId` = `Timelines.timelineId`
  LEFT JOIN `Pivacies` ON `Privacies.entityId` = `$post.postId`
  LEFT JOIN `PrivacyCollectionRoles`
    ON `Privacies.privacyId` = `PrivacyCollectionRoles.privacyId`
  LEFT JOIN `Roles`
    ON `PrivacyCollectionRoles.roleId` = `Role.roleId`
  LEFT JOIN `CollectionsUsers`
    ON `Privacies.collectionId` = `CollectionsUsers.collectionId`
WHERE
  `Privacies.entityType` = 'Comment' AND
  ( 
    `Roles.name` = 'timeline.contributor' OR 
    `Roles.name` = 'post.commenter' OR
    `Roles.name` = 'post.author' OR
    `Roles.name` = 'timeline.author'
  );
```

#### Use-case 3

Get all the comments for a post a user is allowed to view, where post visiblity is `visible` and privacy is `PRIVATE_TO_USERS`. From the Permission Map, we can see that `timeline.author`, `timeline.contributors`, `timeline.commenters` and `timeline.viewers` can see a comment. 

> Parameters
> `$postId`
> `userId`

```sql
SELECT `commentId`
FROM `Comments`
  LEFT JOIN `Privacies`
    ON `Comments.commentId` = `Privacies.entityId`
  LEFT JOIN `PrivacyCollectionRoles`
    ON `Privacies.privacyId` = `PrivacyCollectionRoles.privacyId`
  LEFT JOIN `CollectionUsers`
    ON `PrivacyCollections.collectionId` = `CollectionUsers.collectedId`
  LEFT JOIN `Roles`
    `PrivacyCollectionRoles.roleId` = `Roles.roleId`
WHERE
  `Comments.postId` = `$postId` AND
  `Privacies.entityType` = 'Comment' AND
  `CollectionUsers.userId` = `$userId` AND
  (
    `Roles.name` = 'contributor' OR
    `Roles.name` = 'timeline.author' OR
    `Roles.name` = 'timeline.viewer' OR
    `Roles.name` = 'post.author' OR
    `Roles.name` = 'post.commenter'
  );
  
```

> :exclamation: Note: We have violated normalization rules for a table or two for simplifications. Also note that, the queries are not tested.

  
