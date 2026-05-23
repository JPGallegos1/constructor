# ADR: Frontend Transformations and Data Boundaries

## Context

Frontend applications often need to transform data for rendering: filtering visible rows, mapping API responses into view models, sorting already-loaded cards, validating selected UI items, or grouping information for display.

These transformations are valid and useful. The problem is not using `.filter()`, `.map()`, `.sort()`, or client-side state. The problem is using the frontend as the source of truth for business decisions, permissions, search, matching, ranking, pricing, or access to large/sensitive datasets.

As a general rule, frontend code should operate over data that has already been scoped by the backend. The backend should remain responsible for operations over domain-owned data, user/customer-owned data, multi-tenant datasets, sensitive records, and any operation that affects business behavior.

## Decision

Frontend transformations are allowed only over already-scoped UI datasets.

Any operation over user-owned, customer-owned, tenant-owned, or business-critical datasets must run server-side through API services, backend tools, or domain services.

In other words:

> The problem is not using `.filter()` in the frontend.  
> The problem is using the frontend as the search, matching, ranking, permission, or decision engine over business data.

## Principles

### 1. The frontend can transform what it already has

The frontend may filter, map, sort, group, or validate data that is already loaded, already authorized, and already scoped to the current view.

Examples:

- Filtering a visible list of 6 selected items.
- Sorting cards already returned by the API.
- Grouping loaded notifications by date.
- Mapping API results into UI-friendly labels.
- Validating that a selected ID exists in the currently rendered list.
- Applying local search over a small, already-loaded settings list.

### 2. The backend owns business decisions

The backend must handle operations that determine what data the user is allowed to see, what records match a query, what result is ranked higher, or what action is valid.

Examples:

- Searching across thousands of records.
- Filtering records by business rules.
- Ranking, scoring, matching, or recommendations.
- Validating ownership or permissions.
- Calculating prices, eligibility, quotas, limits, or availability.
- Comparing entities using complete domain data.
- Executing AI tools, embeddings, RAG, or semantic search.
- Applying mutations that affect persisted state.

### 3. The frontend should not receive more data than it needs

The frontend should only receive the subset required for the current screen or interaction.

Avoid sending large datasets to the browser just so the frontend can filter them locally. This creates performance issues, increases memory usage, leaks unnecessary information, and makes the frontend responsible for decisions it should not own.

### 4. Client-side validation is UX; server-side validation is authority

Client-side validation is allowed for fast feedback, but it must never be the only validation layer.

Examples:

- The frontend can disable a submit button if required fields are missing.
- The backend must still validate the submitted payload.
- The frontend can check whether a selected item exists in the current list.
- The backend must still validate ownership, permissions, and business rules before executing the action.

## Implementation Guidelines

Use frontend transformations when all of the following are true:

- The dataset is already loaded in the current UI state.
- The dataset is small enough to process without noticeable performance cost.
- The data has already been authorized and scoped by the backend.
- The transformation only affects presentation or local interaction.
- Bypassing the transformation would not create a security or business integrity issue.

Use server-side services or tools when any of the following are true:

- The operation queries the real domain dataset.
- The dataset may grow significantly.
- The dataset contains sensitive, private, customer-owned, or tenant-owned data.
- The result depends on permissions, ownership, or access control.
- The result affects persisted state or business behavior.
- The operation involves matching, ranking, scoring, recommendations, AI, or semantic search.
- The same capability may be reused by other clients, screens, tools, or automations.

## Concrete Examples

### Allowed in frontend

```ts
// Valid: the API already returned this scoped list for the current view.
const visibleItems = currentItems.filter((item) => item.status !== "archived");
```

```ts
// Valid: local lookup over a small list already visible to the user.
const selectedItemsById = new Map(visibleItems.map((item) => [item.id, item]));

const selectedItems = selectedIds
  .map((id) => selectedItemsById.get(id))
  .filter(Boolean);
```

```ts
// Valid: formatting data for presentation.
const rows = apiResults.map((result) => ({
  id: result.id,
  title: result.name,
  subtitle: `${result.role} · ${result.location}`,
}));
```

### Should be server-side

```ts
// Avoid: loading all records into the browser and filtering locally.
const matches = allCustomerRecords.filter((record) =>
  record.skills.includes("React"),
);
```

Preferred:

```ts
// Better: query scoped results from the backend.
const matches = await api.searchRecords({
  query: "React",
  filters: { seniority: "senior" },
  limit: 20,
});
```

```ts
// Avoid: deciding permissions in the browser.
const canEdit = record.ownerId === currentUser.id;
```

Preferred:

```ts
// Better: backend enforces permissions before returning or mutating data.
await api.updateRecord(recordId, payload);
```

```ts
// Avoid: ranking business results in the frontend using incomplete data.
const ranked = candidates.sort((a, b) => b.score - a.score);
```

Preferred:

```ts
// Better: backend ranks with complete data, rules, and observability.
const ranked = await api.rankCandidatesForRole({
  roleId,
  limit: 10,
});
```

## Agent Reference

When implementing a feature, apply this decision checklist:

1. Is the data already loaded, authorized, and scoped to the current UI?
   - Yes: frontend transformation is acceptable.
   - No: use backend/API/tooling.

2. Could this operation expose private, tenant-owned, or business-sensitive data?
   - Yes: backend.

3. Could bypassing this frontend code break permissions or business rules?
   - Yes: backend.

4. Is the operation search, matching, ranking, scoring, recommendation, pricing, eligibility, or persistence?
   - Yes: backend.

5. Is this only presentation logic over a small visible dataset?
   - Yes: frontend is acceptable.

## Consequences

This decision keeps frontend code simple and responsive while preserving backend authority over business logic, security, scalability, and data ownership.

It also avoids two common extremes:

- Over-engineering every small UI transformation as an API call.
- Turning the frontend into an accidental business engine over large or sensitive datasets.
