---
layout: post
title: "GraphQL vs REST: When We Chose GraphQL (And When We Didn't)"
date: 2025-10-28 10:00:00 +0800
categories: api-design graphql rest
author: Casey Liu
---

The GraphQL hype is real. But is it always the right choice? Here's our pragmatic take after running both in production.

## The GraphQL Promise

- **No overfetching**: Get exactly what you need
- **No underfetching**: One request, all your data
- **Strongly typed**: Schema as contract
- **Self-documenting**: GraphQL Playground

Sounds perfect, right?

## When We Chose GraphQL

### Use Case: Mobile App Dashboard

**Problem**: Mobile clients needed data from 7 different REST endpoints. Latency killed UX.

**REST Approach**:
```
GET /api/user/profile
GET /api/user/stats
GET /api/user/recent-orders
GET /api/user/notifications
GET /api/user/recommendations
GET /api/user/settings
GET /api/user/wallet
```

7 round trips. On 4G: ~3.5 seconds.

**GraphQL Approach**:
```graphql
query DashboardData {
  user {
    profile { name, avatar }
    stats { totalOrders, points }
    recentOrders(limit: 5) { id, status, total }
    notifications(unreadOnly: true) { id, message }
    recommendations { products { id, name, price } }
    settings { theme, language }
    wallet { balance }
  }
}
```

One request. 800ms. Win.

### Implementation

```javascript
const typeDefs = gql`
  type User {
    profile: Profile
    stats: Stats
    recentOrders(limit: Int): [Order]
    notifications(unreadOnly: Boolean): [Notification]
    recommendations: Recommendations
    settings: Settings
    wallet: Wallet
  }

  type Query {
    user: User
  }
`;

const resolvers = {
  User: {
    profile: (parent, args, { dataSources }) =>
      dataSources.profileAPI.getProfile(parent.id),

    recentOrders: (parent, { limit }, { dataSources }) =>
      dataSources.orderAPI.getOrders(parent.id, limit),

    // DataLoader for batching
    notifications: (parent, { unreadOnly }, { dataSources }) =>
      dataSources.notificationAPI.getNotifications(parent.id, unreadOnly),
  },
};
```

### N+1 Problem (The Gotcha)

Without careful planning, GraphQL causes the N+1 query problem:

```graphql
query {
  orders {
    user { name }  # Queries user table for EACH order
  }
}
```

**Solution**: DataLoader for batching.

```javascript
const userLoader = new DataLoader(async (userIds) => {
  const users = await db.query(
    'SELECT * FROM users WHERE id = ANY($1)',
    [userIds]
  );
  return userIds.map(id => users.find(u => u.id === id));
});

const resolvers = {
  Order: {
    user: (order, args, { userLoader }) =>
      userLoader.load(order.userId)  // Batches queries
  }
};
```

## When We Stuck with REST

### Use Case: Public API

**Why REST won**:

1. **Caching**: CDN + HTTP caching just works
   ```
   GET /api/products/123
   Cache-Control: public, max-age=3600
   ```
   GraphQL queries with POST? Harder to cache.

2. **Rate limiting**: Simple with REST
   ```
   Rate limit: 100 requests/hour per endpoint
   ```
   With GraphQL, one query can fetch 1 field or 1000. How do you rate limit fairly?

3. **Monitoring**: Every REST endpoint is a metric
   ```
   GET /api/products - 50ms avg
   GET /api/users - 120ms avg
   ```
   GraphQL: All hits go to `/graphql`. Harder to track which queries are slow.

4. **Third-party integration**: Most tools expect REST

### Use Case: Webhooks

Webhooks push data to clients. They don't query. REST makes sense.

```javascript
POST /webhooks/order-created
{
  "orderId": "123",
  "status": "created",
  "total": 99.99
}
```

No need for GraphQL complexity.

## Hybrid Approach

We run both:
- **GraphQL**: Client-facing APIs (web, mobile)
- **REST**: Public APIs, webhooks, third-party integrations

## GraphQL Challenges We Faced

### 1. Error Handling

GraphQL returns HTTP 200 even with errors.

```json
{
  "data": { "user": null },
  "errors": [
    { "message": "User not found", "path": ["user"] }
  ]
}
```

Clients must check `errors` array. Many developers miss this.

### 2. File Uploads

No standard. We used `graphql-upload`.

```graphql
mutation UploadAvatar($file: Upload!) {
  uploadAvatar(file: $file) {
    url
  }
}
```

Works, but feels hacky compared to `multipart/form-data` in REST.

### 3. Schema Stitching (Microservices)

Combining schemas from multiple services is complex. We use Apollo Federation:

```javascript
// User service
extend type Order @key(fields: "userId") {
  userId: ID! @external
  user: User
}

// Order service
type Order @key(fields: "id") {
  id: ID!
  userId: ID!
  total: Float
}
```

Works, but adds operational overhead.

## Performance Comparison

**GraphQL Wins**:
- Mobile apps (fewer round trips)
- Complex, nested data requirements
- Rapidly changing client needs

**REST Wins**:
- Simple CRUD
- Public APIs
- Heavy caching needs
- Webhooks

## Tools We Use

- **Apollo Server**: GraphQL server
- **DataLoader**: Batching queries
- **GraphQL Code Generator**: TypeScript types from schema
- **Apollo Studio**: Monitoring GraphQL queries

## Bottom Line

GraphQL isn't a silver bullet:
- Use it when clients need flexible, efficient data fetching
- Stick with REST for simple APIs, caching, and third-party integrations
- Don't rewrite working REST APIs just for the hype

We're happy with our hybrid approach.

---

*GraphQL vs REST? What's your experience? Let's discuss!*
