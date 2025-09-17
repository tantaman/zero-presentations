---
marp: true
size: 16:9
paginate: true
theme: "zero-alpha2"
---

# Zero

## Synced Queries & Custom Mutators

<img src="./mascot-jumping.png" class="mascot center" />

---

# Synced Query

- Like today's queries
- `[name, args]` tuple sent to server instead of an AST
- Advantages
  - Locks down the server
  - Custom code on read path
  - Custom authorization
- Disadvantages
  - More complex deployment

---

# Synced Query (Shared)

<div class="two-col">
  <div class="col">

```typescript
// Loaded on both the client and server.
// Defined once.
const todoList = syncedQuery(
  'todoList',
  ({context, id}: {context: Session, id: string}) =>
    builder
      .todo
      .where('listId', id)
      .whereExists(
        'collaborators',
        q => q.where('userId', context.userId)
      )
);
```
  </div>
</div>

<img src="./mascot-running.png" class="mascot-sm bottom-left" />

---

# Synced Query (Divergent 1)

<div class="two-col">
  <div class="col">

## Client

```typescript
const todoList = syncedQuery(
  'todoList',
  ({context, id}: {context: Session, id: string}) =>
    builder
      .todo
      .where('listId', id)
);
```
  </div>
  <div class="col">

## Server

```typescript
const todoList = syncedQuery(
  'todoList',
  ({context, id}: {context: Session, id: string}) =>
    // call client impl to share base query
    clientTodoList
      // append permission check to only
      // run server side
      .whereExists(
        'collaborators',
        q => q.where('userId', context.userId)
      )
);
```
  </div>
</div>

<img src="./mascot-blocking.png" class="mascot-sm bottom-left" />

---

# Synced Query (Divergent 2)

<div class="two-col">
  <div class="col">

  ## Client

```typescript
const todoList = syncedQuery(
  'todoList',
  ({id}: {id: string}) => builder
    .todo
    .where('listId', id)
);
```
  </div>
  <div class="col">

## Server

```ts
const todoList = syncedQuery(
  'todoList',
  z.object({
    id: z.string(),
  }),
  async ({context, id}) => {
    // load row via drizzle
    const user = await db.query.users.findFirst({
      where: eq(users.id, context.userId),
    });

    // call Polar or some 3rd party authz service
    const allowed = await checkListAccess(
      user,
      id,
    );

    // share code / call into client impl
    const q = clientTodoList({id});
    return allowed ? q : q.where(alwaysFalse);
  }
);
```
  </div>
</div>

<img src="./mascot-jumping.png" class="mascot-sm bottom-left" />

---

# Reviving Ad-Hoc Queries

Synced queries, a more powerful primitive than today's queries.

```ts
// Gist: AST is the arg to the query
const adHoc = syncedQuery(
  'adHoc',
  astSchema,
  ({ast}) => newQueryFromAst(ast), // coming soon
  // Note: could inject RLS rules by walking the AST
);

// usage:
useQuery(adHoc(builder.todo.where('listId', id).ast))
```


<div class="note">⚠️ Notional example</div>

---

# Custom Mutators

- Formalized version of CRUD mutators

---


# Custom Mutator

todo: code example

---

# Why?

todo: questions we asked ourselves
- How to use POLAR, Google RBAC thing
  - External systems
- HTTP session cookies
- Lock down server


- Ad-hoc
- Local-only
- no more permissions system
- fewer concepts?
- no more