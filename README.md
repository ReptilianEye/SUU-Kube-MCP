# Kubernetes controlled by MCP server observability case study - a summer semester SOA project

Project Acronym: MCP-K8

## Introduction

The case study aims to integrate [kubernetes-mcp-server](https://github.com/containers/kubernetes-mcp-server) into a sample Kubernetes application
with non-trivial architecture and demonstrate how LLM can modify cluster configuration and deployment.
Interactions between language model and the K8s runtime should be monitored by Grafana.

For our sample application, we chose [OpenTelemetry Demo Project](https://opentelemetry.io/docs/demo/).
The application is designed to serve as a Docker / Kubernetes deployment example. It showcases a modern
implementation of an online store, with separate microservices for, among other things, product catalog, cart,
payments, reviews, recommendations and provides both web UI and mobile appliation. It also contains some
quality-of-life features for our use-case, like load generator.

## Technology stack

The application stack is dictated by the implementation of OpenTelemetry Demo Project.

The applications is deployed using Kubernetes.

Grafana is the core for observability and tracking LLM activity.

`kubernetes-mcp-server` serves as an interface between Kubernetes cluster and language model of choice (each student's own preference).
