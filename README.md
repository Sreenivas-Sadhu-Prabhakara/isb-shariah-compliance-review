# Shariah Compliance Review

BIAN Service Domain microservice — part of the [islamic-platform](../../islamic-platform/) landscape.

| | |
|---|---|
| **Business Area** | Shariah Governance and Compliance |
| **Business Domain** | Shariah Governance |
| **Functional Pattern** | Assess |
| **Asset Type** | Shariah Compliance Case |
| **Control Record** | Shariah Compliance Case Assessment |
| **K8s Namespace** | `isb-governance` |
| **Stack** | Java 21 · Spring Boot 3 · Resilience4j · Cilium mesh |

> ⚠️ **Phase 1 (shallow):** real REST API over an in-memory store. Phase 2 replaces the store with per-domain persistence and real domain logic. This repo was stamped from `islamic-platform/generator` — regenerate rather than hand-editing boilerplate.

## BIAN Semantic API

| Method | Path | BIAN action term |
|---|---|---|
| GET | `/v1/service-domain` | — (SD metadata) |
| POST | `/v1/shariah-compliance-case-assessment/initiate` | Initiate |
| GET | `/v1/shariah-compliance-case-assessment` | Retrieve (list) |
| GET | `/v1/shariah-compliance-case-assessment/{crId}/retrieve` | Retrieve |
| PUT | `/v1/shariah-compliance-case-assessment/{crId}/update` | Update |
| PUT | `/v1/shariah-compliance-case-assessment/{crId}/control` | Control — body `{"action": "suspend"\|"resume"\|"terminate"}` |

OpenAPI UI: `/swagger-ui.html` · Health: `/actuator/health` · Metrics: `/actuator/prometheus`

**API contract:** [`api/openapi.yaml`](api/openapi.yaml) — owned by **this repo** (contract-per-repo; no central contracts repo). The runtime spec at `/v3/api-docs` must stay compatible with it; Phase 2 adds contract tests that enforce this.

## Run locally

```bash
mvn spring-boot:run
curl localhost:8080/v1/service-domain

# lifecycle smoke test
curl -X POST localhost:8080/v1/shariah-compliance-case-assessment/initiate -H 'content-type: application/json' -d '{"note":"hello"}'
```

## Build & containerize

```bash
mvn -B verify
docker build -t isb/isb-shariah-compliance-review:0.1.0 .
```

## Deploy (Helm → K8s with Cilium mesh)

```bash
helm upgrade --install isb-shariah-compliance-review ./helm -n isb-governance
```

Exposed through the platform Gateway at path prefix `/isb-shariah-compliance-review` (Cilium Gateway API). Mesh policy (`CiliumNetworkPolicy`) allows: gateway ingress, same-area peers, Prometheus — everything else denied.
