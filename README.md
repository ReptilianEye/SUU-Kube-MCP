# Kubernetes controlled by MCP server observability case study - a summer semester SOA project

Project Acronym: MCP-K8

---

## Contents

1. [Introduction](#1-introduction)
2. [Theoretical background / technology stack](#2-theoretical-background--technology-stack)
3. [Case study concept description](#3-case-study-concept-description)
4. [Case study high level architecture](#4-case-study-high-level-architecture)
5. [Case study detailed architecture](#5-case-study-detailed-architecture)
6. [Environment configuration description](#6-environment-configuration-description)
7. [Installation method](#7-installation-method)
8. [Demo deployment steps](#8-demo-deployment-steps)
9. [Demo description](#9-demo-description)
10. [Summary – conclusions](#10-summary--conclusions)
11. [References](#11-references)

---

## 1. Introduction

The case study aims to integrate [kubernetes-mcp-server](https://github.com/containers/kubernetes-mcp-server) into a sample Kubernetes application with non-trivial architecture and demonstrate how LLM can modify cluster configuration and deployment. Interactions between language model and the K8s runtime should be monitored by Grafana.

For our sample application, we chose [OpenTelemetry Demo Project](https://opentelemetry.io/docs/demo/). The application is designed to serve as a Docker / Kubernetes deployment example. It showcases a modern implementation of an online store, with separate microservices for, among other things, product catalog, cart, payments, reviews, recommendations and provides both web UI and mobile application. It also contains some quality-of-life features for our use-case, like load generator.

---

## 2. Theoretical background / technology stack

The application stack is dictated by the implementation of [OpenTelemetry Demo Project](https://opentelemetry.io/docs/demo/):

- **OpenTelemetry** – Collection and export of traces, metrics, and logs from application services via OTLP.
- **Prometheus** – Time-series metrics storage; collects information from services and the OpenTelemetry Collector.
- **OpenSearch** – Log storage and search; receives logs from the OpenTelemetry Collector.
- **Jaeger** – Distributed tracing backend.
- **Grafana** – Visualization; reads from Prometheus, OpenSearch, and Jaeger.
- **Kubernetes** – Orchestration of the demo application.
- **kubernetes-mcp-server** – Interface between the Kubernetes cluster and the language model.

**Logs in Grafana:** Application services emit logs via the OpenTelemetry SDK. The OpenTelemetry Collector receives them (OTLP) and exports to OpenSearch. Grafana connects to OpenSearch as a datasource and displays logs in dashboards and Explore.

---

## 3. Case study concept description

The core of this case study is **updating Kubernetes configuration using the Kubernetes MCP server**, integrated with Claude Code or Cursor IDE. Through the chat, you can diagnose issues, observe the application, and scale it up—all driven by natural language prompts.

### Connecting the Kubernetes MCP to Cursor IDE or Claude Code

1. **Add the Kubernetes MCP server** to your IDE’s MCP configuration (e.g. Cursor Settings → Tools & MCP, or `.cursor/mcp.json` / Claude Code MCP config). Use the [kubernetes-mcp-server](https://github.com/containers/kubernetes-mcp-server) configuration from its documentation.
2. **Prerequisites:** Ensure `kubectl` is installed and a cluster is running (e.g. Minikube). The MCP server uses your default kubeconfig (`~/.kube/config`) and current context.
3. **Restart the IDE** so it picks up the new MCP server. The AI assistant can then use the server’s tools to work with the cluster.

### Using the chat to diagnose, observe, and scale up

Once connected, you can use the chat to:

- **Diagnose** – e.g. “List pods in the default namespace”, “Describe the frontend deployment”, “Show recent events for failing pods”. The assistant uses MCP tools to run equivalent `kubectl` commands and interpret the output.
- **Observe** – e.g. “How many replicas are running for each deployment?”. You can cross-check with Grafana dashboards (http://localhost:8080/grafana) for metrics and logs.
- **Scale up** – e.g. “Scale the frontend deployment to 3 replicas” or “Scale up the cart service”. The assistant uses MCP tools to update the deployment configuration.

All changes made via the MCP server are reflected in the cluster and can be verified in Grafana.

---

## 4. Case study high level architecture

The demo deploys application microservices (frontend, cart, payment, etc.), the OpenTelemetry Collector, Prometheus, OpenSearch, Jaeger, and Grafana on Kubernetes. Telemetry flows from services to the collector and then to the appropriate backends; Grafana queries these backends for visualization.

---

## 5. Case study detailed architecture

(To be expanded in the full report with diagrams, component descriptions, and data flow.)

---

## 6. Environment configuration description

- **Runtime:** Minikube (Kubernetes)
- **Resources:** 8 GB RAM, 4 CPUs (required for the OpenTelemetry Demo)
- **Namespaces:** `otel-demo` (observability stack), `default` (application services)

---

## 7. Installation method

### Prerequisites

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

---

## 8. Demo deployment steps

### a. Configuration set-up

1. Start Minikube with sufficient resources:
   ```bash
   minikube start --memory=8192 --cpus=4
   ```
   or with max resources:

   ```bash
   minikube start --memory=max --cpus=max
   ```

2. Deploy the OpenTelemetry Demo:

   ```bash
   kubectl apply -f opentelemetry-demo/kubernetes/opentelemetry-demo.yaml
   ```

### b. Data preparation

(To be expanded in the full report: any required ConfigMaps, secrets, or pre-loading steps.)

---

## 9. Demo description

### a. Execution procedure

1. Ensure Minikube is running and the demo is deployed (see §8).
2. Start port-forwarding the frontend-proxy:

   ```bash
   kubectl port-forward svc/frontend-proxy 8080:8080 -n otel-demo
   ```

   (If the service is in `default`, use `-n default` instead.)

3. Access the application:
   - **Web store:** http://localhost:8080
   - **Grafana:** http://localhost:8080/grafana

### b. Results presentation

(To be expanded in the full report: all prompts used with AI models, screenshots from Grafana dashboards.)

---

## 10. Summary – conclusions

(To be expanded in the full report.)

---

## 11. References

- [OpenTelemetry Demo Project](https://opentelemetry.io/docs/demo/)
- [kubernetes-mcp-server](https://github.com/containers/kubernetes-mcp-server)
- [Grafana](https://grafana.com/)
- [Prometheus](https://prometheus.io/)