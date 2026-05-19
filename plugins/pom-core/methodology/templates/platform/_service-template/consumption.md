# Service Consumption — {{service-name}}

How to integrate with this service.

## Integration pattern

{{Describe the canonical integration shape: client library, API, message queue, etc.}}

```{{language}}
{{Concrete example: the smallest piece of code or config that consumes the service successfully.}}
```

## Configuration

| Setting | Required? | Default | Notes |
|---|---|---|---|
| {{setting}} | yes/no | {{default}} | {{notes}} |

## Failure modes & fallbacks

What consumers should do when the service is degraded or unavailable.

| Failure mode | Symptom | Fallback |
|---|---|---|
| Service unavailable | {{HTTP 503, timeout, etc.}} | {{what consumers do}} |
| Degraded performance | {{latency > X}} | {{what consumers do}} |
| Data inconsistency | {{symptom}} | {{what consumers do}} |

## Versioning & deprecation

{{How the service versions its API; how breaking changes are announced; deprecation timelines.}}

## Observability hooks consumers should add

- {{metric to emit, e.g., "request count to service X"}}
- {{alert recommendation, e.g., "page on > N% errors over 5 minutes"}}
