# kgateway (kgateway)
kgateway is the most widely deployed gateway in Kubernetes for microservices and AI agents. It is a feature-rich, fast, and flexible Kubernetes-native ingress controller and next-generation API gateway built on top of Envoy proxy and the Kubernetes Gateway API. Originally born as Gloo in 2018, kgateway provides a resilient and performance-oriented control plane that scales from lightweight microgateway deployments between services to massively parallel centralized gateways handling billions of API calls. The project provides custom resource definitions for traffic management, backend routing (including AI providers like OpenAI, Azure, and Gemini), direct responses, gateway extensions for external auth and rate limiting, and infrastructure configuration.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/kgateway/refs/heads/main/apis.yml)

## Scope

- **Type:** Index 
- **Position:** Consuming 
- **Access:** 3rd-Party 

## Tags:

 - Gateways

## Timestamps

- **Created:** 2026-01-02 
- **Modified:** 2026-03-16 

## APIs

### kgateway Kubernetes Gateway API
The kgateway Kubernetes Gateway API provides custom resource definitions (CRDs) under the gateway.kgateway.dev/v1alpha1 API group for managing traffic policies, backends, direct responses, gateway extensions, gateway parameters, HTTP listener policies, and AI backends. kgateway is built on Envoy proxy and implements the Kubernetes Gateway API for microservices and AI agent traffic management.

**Human URL:** [https://kgateway.dev/docs/envoy/latest/reference/api/](https://kgateway.dev/docs/envoy/latest/reference/api/)


#### Tags:

 - Gateways, Kubernetes, Envoy, API Gateway, AI Gateway, Traffic Management

#### Properties

- [Documentation](https://kgateway.dev/docs/envoy/latest/reference/api/)
- [OpenAPI](openapi/kgateway-kubernetes-gateway-api-openapi.yml)
- [JSONSchema](json-schema/traffic-policy.json)
- [JSONSchema](json-schema/backend.json)
- [JSONSchema](json-schema/direct-response.json)
- [JSONSchema](json-schema/gateway-extension.json)
- [JSONSchema](json-schema/gateway-parameters.json)
- [JSONSchema](json-schema/http-listener-policy.json)
- [JSONSchema](json-schema/ai-backend.json)
- [JSONLD](json-ld/kgateway-context.jsonld)

## Common Properties

- [Website](https://kgateway.dev/)
- [Videos](https://kgateway.dev/resources/videos/)
- [Blog](https://kgateway.dev/blog/)
- [Documentation](https://kgateway.dev/docs/envoy/latest/)
- [GitHub](https://github.com/kgateway-dev/kgateway)

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
