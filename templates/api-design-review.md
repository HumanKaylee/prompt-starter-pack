---
name: API Design Review
use_case: Review or design a REST/GraphQL/RPC API before implementation
variables:
  - API_DESCRIPTION: the API design (OpenAPI spec, endpoint list, or prose description)
  - API_TYPE: REST, GraphQL, gRPC, or other
  - CONSUMERS: who calls this API (internal services, mobile clients, third-party developers, etc.)
  - CONSTRAINTS: versioning policy, auth mechanism, existing conventions to follow
---

# API Design Review Prompt

## Prompt

```
You are a senior API designer reviewing this {{API_TYPE}} API. Your goal: make it easy to use correctly and hard to use incorrectly.

**Consumers:** {{CONSUMERS}}
**Constraints:** {{CONSTRAINTS}}

**API design to review:**
{{API_DESCRIPTION}}

Review across these dimensions:

### Naming and Consistency
Are resource names, action names, and field names consistent? Are they intuitive to a developer who hasn't read the documentation?

### Request/Response Design
Are requests minimal (no required fields the server can infer)? Are responses predictable (no fields that appear/disappear based on state)?

### Error Handling
Are errors distinguishable? Can a client know how to recover from each error type? Are HTTP status codes used correctly?

### Pagination, Filtering, and Sorting
Are large collections paginated? Is pagination cursor-based (preferred) or offset-based? Are filters and sort fields consistent?

### Versioning and Evolution
How will breaking changes be communicated? What's the deprecation path? Are there any design decisions that make future changes harder?

### Security
Is authentication required where it should be? Is authorization enforced at the resource level? Are there over-exposed fields?

### Developer Experience
Would a developer find this API intuitive? Where will they get confused? What should the quickstart look like?

Rate each dimension: ✅ Good / ⚠️ Needs Work / ❌ Blocking Issue. For each ⚠️ or ❌, give the specific fix.
```

## Worked Example

**API:** A REST API for a task management system with endpoints for creating tasks, assigning users, and marking complete

**What a good response looks like:** Names rating is ⚠️ because `POST /tasks/create` should be `POST /tasks`; error handling is ❌ because all errors return 200 with an error field (a common but wrong pattern); versioning is ⚠️ because no version in the URL or headers; developer experience notes that the authentication flow requires 3 calls before you can do anything useful.

## Tuning Notes

- For greenfield APIs: run this on a draft spec before writing any implementation code. The cost to change is near zero at design time.
- Ask for "the three most important changes" if you want to prioritize ruthlessly.
- For internal APIs: relax the DX requirements but increase the focus on consistency (internal API inconsistency is a maintenance tax forever).
- Add: "How would you design the webhook/event equivalent of these endpoints?" to think through async patterns early.
