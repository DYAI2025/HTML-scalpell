# Architecture model contract

Use this model whenever an architecture must be visualized, edited, exported, or handed off to a coding agent.

## Root object

```json
{
  "project": {
    "name": "Project name",
    "domain": "software | product | api | workflow | data | agent | mixed",
    "repo": "optional repository or workspace",
    "timestamp": "ISO-8601 timestamp",
    "description": "short architecture purpose"
  },
  "nodes": [],
  "edges": [],
  "changes": []
}
```

## Node object

```json
{
  "id": "stable-kebab-case-id",
  "x": 500,
  "y": 420,
  "type": "entry | ui | service | api | data | agent | workflow | external | security | infra | domain | custom",
  "icon": "◇",
  "label": "Short label",
  "sub": "short subtitle",
  "detail": {
    "title": "Full title",
    "subtitle": "layer · responsibility",
    "tags": ["api", "frontend"],
    "flow": ["step one", "step two"],
    "apis": [],
    "schemas": [],
    "notes": "implementation context",
    "constraints": [],
    "risks": []
  }
}
```

## API route object

```json
{
  "method": "GET",
  "path": "/api/resource/:id",
  "desc": "what the endpoint does",
  "params": ["userId: string", "date: ISO-8601"],
  "requestSchema": { "type": "object", "properties": {} },
  "responseSchema": { "type": "object", "properties": {} },
  "consumers": ["dashboard"],
  "auth": "unknown | public | session | bearer | service",
  "status": "planned | active | deprecated | removed"
}
```

## Edge object

```json
{
  "from": "source-node-id",
  "to": "target-node-id",
  "style": "primary | secondary | api | event | data | dependency | risk | ownership",
  "label": "optional edge label",
  "relation": "user-flow | api-call | event-flow | data-flow | dependency | deployment | ownership | risk-control",
  "protocol": "HTTP | queue | file | human | internal | unknown",
  "data": "optional payload or domain object"
}
```

## Change object

```json
{
  "id": "c_stable_or_random",
  "timestamp": "ISO-8601 timestamp",
  "type": "modify | add | delete | reroute | deprecate",
  "target": "node | edge | api | flow | schema | notes | tags | constraint | risk",
  "nodeId": "optional-node-id",
  "edgeIndex": 0,
  "apiIndex": 0,
  "before": {},
  "after": {},
  "description": "human-readable exact edit"
}
```

## Normalization rules

- Use kebab-case IDs.
- Preserve original endpoint casing and path syntax.
- Use `Unknown - search required` instead of inventing file paths.
- Keep deleted nodes in the model with `_deleted: true` until export is complete so the brief can reference them.
- Sort endpoints by path, then method.
- Sort nodes by known runtime order, otherwise by layer, then label.
