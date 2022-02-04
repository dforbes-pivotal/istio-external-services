# Service Mesh Patterns with Mesh-External HTTPS Services

This repo demonstrates an approach to accessing external services from inside a service mesh,
and applying some of the traffic management approaches to those services to apps running inside the mesh.

The external services are assumed to be available over HTTPS, proxied by an Istio egress gateway
which makes those services available inside the mesh over HTTP, although traffic inside the mesh
would be secured according to the TLS configuration of the mesh. This allows an Istio `VirtualService`
to manage the traffic to the external services from the point of view of the mesh workloads, enabling
traffic shifting, fault injection, timeouts and retries, and so on.

## Deploying

### Target services

Included are two apps that can be deployed to Cloud Foundry, where we assume access to those apps is
secured with TLS. The Kubernetes manifests that define the workloads that connect to these Cloud Foundry
services refer to those apps as `hello-istio-v1.apps.coachella.cf-app.com` and `hello-istio-v2.apps.coachella.cf-app.com`;
substitute the actual names for these apps in the Kubernetes manifest.

For details of deploying these Cloud Foundry apps, see the [README for the Cloud Foundry apps](cf/README.md).

### Consuming services

To demonstrate connecting to external services and managing traffic, I've included several Kubernetes
manifests for three main groupings of resources:

1. A proxy, that accepts incoming web requests and returns the result of calling one of the external services
1. A caller, that makes regular requests to the proxy
1. A set of Istio configuration for re-exposing the external services inside the mesh and applying traffic
management patterns

For details of deploying these Kubernetes resources, see the [README for the Kubernetes manifests](k8s/README.md).

## Istio

This demo relies on Istio being installed in a Kubernetes cluster. It's been tested with Istio 1.12.1
and Kubernetes 1.22.3.

For this demo, Istio must have the egress gateway enabled, as it is when installed using the `demo` profile.

Istio may be installed with external service passthrough _disabled_ by default (set `meshConfig.outboundTrafficPolicy.mode`
to `REGISTRY_ONLY`). This will allow us to validate that our explicit configuration is allowing access
to the external services, rather than access to those services being available by default.

This demo has relied heavily on Istio documentation, in particular:

- [Accessing External Services](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-control/)
- [Egress Gateways](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway/)
- [Egress TLS Origination](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-tls-origination/)

These together describe the patterns implemented here, which is:

1. Allow access to the external services via a `ServiceEntry`, and define it inside the mesh on port 80
1. Create an egress `Gateway` to route access to the service from inside the mesh
1. Use a `DestinationRule` to set up TLS origination for the external service
1. Define a `VirtualService` to direct traffic from the sidecars to the egress gateway and from the egress
gateway to the external service

This allows an application inside the mesh to reference an external service by an arbitrary name, and have
its requests to that service traverse the mesh via the Envoy sidecars and be subject to any traffic management
or other rules that may be configured.

The result is a traffic flow that looks like this:

`traffic` --> `envoy` --> `service-that-calls-externally` --> `app-that-calls-externally` --> `envoy` --> `istio-egressgateway` --> external service

## Validation

To check that this demo is working as intended, tail the logs of the `traffic` pod(s) (`kubectl logs deploy/traffic -f`).
With the traffic shifting that is set up by default with these Kubernetes resources, the output should
come from a combination of the v1 and v2 services running on Cloud Foundry, with a few occurrences of
HTTP 418 failures (these show up as "`fault filter abort`"):

```text
2022-02-04T10:34:33+00:00       Hello K8s, from CF v1 service!
2022-02-04T10:34:34+00:00       Hello K8s, from CF v2 service!
2022-02-04T10:34:35+00:00       Hello K8s, from CF v1 service!
2022-02-04T10:34:36+00:00       Hello K8s, from CF v2 service!
2022-02-04T10:34:37+00:00       fault filter abort
2022-02-04T10:34:38+00:00       Hello K8s, from CF v2 service!
2022-02-04T10:34:39+00:00       Hello K8s, from CF v2 service!
2022-02-04T10:34:40+00:00       Hello K8s, from CF v1 service!
2022-02-04T10:34:41+00:00       Hello K8s, from CF v1 service!
2022-02-04T10:34:42+00:00       Hello K8s, from CF v2 service!
2022-02-04T10:34:43+00:00       Hello K8s, from CF v2 service!
2022-02-04T10:34:44+00:00       fault filter abort
2022-02-04T10:34:45+00:00       Hello K8s, from CF v1 service!
```
