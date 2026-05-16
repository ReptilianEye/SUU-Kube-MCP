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

### OpenTelemetry
Acts as the foundation of modern observability, providing a unified standard and a set of tools to generate and transmit telemetry data (logs, metrics, and traces). By utilizing the OTLP (OpenTelemetry Protocol) and the Collector component, the project enables seamless integration of services written in various programming languages. In this demo, it serves as the "central nervous system" that aggregates data from microservices and routes it to the appropriate backend systems.
### Prometheus
Collects performance metrics from both the Kubernetes infrastructure layer and directly from application business logic via pull mechanisms or OpenTelemetry Collector integration. It allows for precise tracking of resource utilization (CPU, RAM) and application-specific indicators, which is critical for the LLM's decision-making process during scaling operations.
### OpenSearch
Stores structured text data received from the collector, enabling rapid indexing and searching. Through Grafana integration, it allows users and AI agents to correlate log errors with metric anomalies, significantly accelerating root cause analysis in a distributed microservices environment.
### Jaeger 
Handles distributed tracing in microservice-based systems. It makes it possible to follow requests across multiple services and helps identify latency issues, failed calls, and dependencies between components. In the OpenTelemetry Demo environment, this allows end-to-end request flows to be inspected more easily.
### Grafana
Provides a unified interface for visualizing and exploring observability data such as metrics, logs, and traces. By integrating multiple data sources, it supports both dashboard-based monitoring and ad hoc analysis. This makes it a convenient entry point for observing telemetry collected across the environment.
### Kubernetes 
Enables automated deployment, management, and scaling of containerized applications. It supports service orchestration, workload recovery, and resource control, which makes it well suited to microservice-based environments. In this case, it provides the runtime layer for the demo application and the supporting observability components.
### kubernetes-mcp-server
Serves as a bridge between the language model and the Kubernetes cluster. It gives AI assistants structured access to cluster resources and makes it possible to inspect the environment or perform selected operations through MCP-compatible tools. As a result, cluster interaction can be performed through natural-language prompts rather than through dashboards or manual command-line operations.

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
- **Scale up** – e.g. “Scale the frontend deployment to 3 replicas” or “Scale up the cart service”. The assistant should use the **Kubernetes MCP** scaling tools (for example `kubectl_scale` on the deployment), not ad hoc `kubectl scale` in a terminal, so scaling stays part of the MCP-driven workflow and matches the case study.

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
- **Namespaces:** `otel-demo` (application services and observability stack)

---

## 7. Installation method

### Prerequisites

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Docker](https://docs.docker.com/get-docker/)

---

## 8. Demo deployment steps

### a. Configuration set-up

1. Ensure that Docker is running.

   Minikube requires a container or virtual machine driver to start the Kubernetes cluster. In this setup, Docker is used as the Minikube driver, so Docker Desktop should be running before starting Minikube.

2. Start Minikube with sufficient resources:

   ```bash
   minikube start --memory=8192 --cpus=4
   ```

   or with max resources:

   ```bash
   minikube start --memory=max --cpus=max
   ```

3. Deploy the OpenTelemetry Demo:

   ```bash
   kubectl create -n otel-demo -f opentelemetry-demo/kubernetes/opentelemetry-demo.yaml
   ```

4. Enable `kube-state-metrics` service (for grafana k8s monitoring):

   ```bash
   kubectl apply -n otel-demo -f kube-metrics/
   ```
   Later you can import [this dashboard](https://grafana.com/grafana/dashboards/15661-k8s-dashboard-en-20250125/) in grafana (copying ID is the simplest way).

### b. Data preparation

The demo does not require a manually prepared dataset. Telemetry data is generated automatically by the OpenTelemetry Demo application after deployment.

The deployed microservices emit logs, metrics, and traces through the OpenTelemetry SDK. The built-in `load-generator` continuously produces sample traffic for the online store, so Grafana can display changing observability data during the demo.

Before continuing with the demo scenario, wait until the application and observability components are ready:

```bash
kubectl get pods -n otel-demo -w
```
On the first run, pulling container images and starting all services can take approximately **5–20 minutes**, depending on the machine and network connection. Most pods should eventually reach the Running status.

---

## 9. Demo description

### a. Execution procedure

This section describes the MCP-K8 demo scenario executed after completing the deployment and preparation steps from section 8. At this point, the Minikube cluster should be running, the OpenTelemetry Demo should be deployed in the `otel-demo` namespace, and the required pods should be ready.

#### Step 1: Verify that the environment is deployed

Before starting the demo scenario, verify that the application and observability components are running:

```bash
kubectl get pods -n otel-demo
```

Expected result:

- All required pods should be in the `Running` status.
- The output should include application pods such as `frontend-*`, `cart-*`, `checkout-*`, and `product-catalog-*`.
- Observability-related pods such as `grafana-*`, `prometheus-*`, `opensearch-*`, and `otel-collector-*` should also be present.

Pod names contain generated suffixes, so exact names will differ between deployments.

#### Step 2: Start port-forwarding

In a separate terminal window, start port-forwarding for the `frontend-proxy` service:

```bash
kubectl port-forward svc/frontend-proxy 8080:8080 -n otel-demo
```

This terminal must remain open during the demo, because it provides local access to the web store and Grafana.

Expected result:

```text
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

#### Step 3: Verify local access

Open the web store in a browser:

```text
http://localhost:8080/
```

Open Grafana in a browser:

```text
http://localhost:8080/grafana/
```

Expected result:

- The Astronomy Shop web UI should be available at `http://localhost:8080/`.
- Grafana should be available at `http://localhost:8080/grafana/`.

#### Step 4: List pods through MCP in Cursor IDE

Prerequisite: `kubernetes-mcp-server` must be configured in Cursor IDE under **Cursor Settings → Tools & MCP**. The MCP server should use the current Kubernetes context, which points to the local Minikube cluster.

In the Cursor IDE MCP chat, use the following prompt:

```text
List all pods in the otel-demo namespace
```

Expected result:

- The MCP server should return a list of pods from the `otel-demo` namespace.
- Pod names and statuses should be visible in the response.
- The response should include application pods such as `frontend-*`, `cart-*`, `checkout-*`, and `product-catalog-*`.
- Observability-related pods such as `grafana-*`, `prometheus-*`, `jaeger-*`, `opensearch-*`, and `otel-collector-*` should also be present.

#### Step 5: List services through MCP in Cursor IDE

In the Cursor IDE MCP chat, use the following prompt:

```text
List all services in the otel-demo namespace
```

Expected result:

- The MCP server should return a list of services from the `otel-demo` namespace.
- Service names and exposed ports should be visible in the response.
- The response should include `frontend-proxy`.
- The `frontend-proxy` service should expose port `8080`.
- Other expected services include application services such as `frontend`, `cart`, `checkout`, `payment`, and `product-catalog`, as well as observability services such as `grafana`, `prometheus`, `jaeger`, `opensearch`, and `otel-collector`.

#### Step 6: Open Grafana and import Kubernetes dashboard

Navigate to Grafana:

```text
http://localhost:8080/grafana/
```

Import the Kubernetes dashboard:

1. Open **Dashboards**.
2. Click **New** and then **Import**.
3. Enter the dashboard ID:

   ```text
   15661
   ```

4. Click **Load**.
5. Select the appropriate Prometheus data source.
6. Click **Import**.

After importing the dashboard, verify that it displays Kubernetes cluster metrics such as workload count, pod count, and node count.

Record the baseline pod count before scaling. This value will be compared with the pod count after scaling the `frontend` deployment.

The total pod count may differ depending on the OpenTelemetry Demo version and the current cluster state. The baseline deployment is expected to start with approximately one replica per deployment.

#### Step 7: Scale the frontend deployment through MCP

The `frontend` deployment is the main Next.js Astronomy Shop UI and is used as the scaling target in this demo.

In the Cursor IDE MCP chat, use the following prompt:

```text
Scale the frontend deployment in the otel-demo namespace to 3 replicas
```

Expected result:

- The assistant should use the Kubernetes MCP server to update the `frontend` deployment.
- The response should confirm that the `frontend` deployment was scaled to `3` replicas.
- Scaling should be performed through the MCP tool, not manually through a terminal command.

#### Step 8: Verify the scale-up through MCP

In the Cursor IDE MCP chat, use the following prompt:

```text
List pods in the otel-demo namespace
```

Expected result:

- There should now be **3 running `frontend-*` pods**.
- Other application and observability pods should remain running.

Example expected result:

```text
frontend-xxxxxxxxxx-aaaaa              Running
frontend-xxxxxxxxxx-bbbbb              Running
frontend-xxxxxxxxxx-ccccc              Running
```

#### Step 9: Verify the scale-up in Grafana

Return to Grafana:

```text
http://localhost:8080/grafana/
```

Refresh the Kubernetes dashboard.

Expected result:

- The panel showing pod count or replica count should reflect the increased number of frontend replicas.
- The total pod count should increase by approximately `2`, because the `frontend` deployment was scaled from `1` replica to `3` replicas.

Expected comparison:

```text
Before scaling:
total pod count: base_val

After scaling:
total pod count: base_val + 2
```

Exact numbers may vary depending on the OpenTelemetry Demo version and cluster state.

### b. Results presentation

This section should be completed after executing the demo scenario described in section 9a.

The final results should include screenshots and observations from:

- Kubernetes pod and service listing,
- Cursor IDE MCP chat responses,
- Grafana Kubernetes dashboard before scaling,
- Grafana Kubernetes dashboard after scaling.

The results should clearly show that the `frontend` deployment was scaled from `1` to `3` replicas through the Kubernetes MCP server, and that the change was visible in both MCP output and Grafana.

---

## 10. Summary – conclusions

(To be expanded in the full report.)

---

## 11. References

- [OpenTelemetry Demo Project](https://opentelemetry.io/docs/demo/)
- [Jaeger](https://www.jaegertracing.io/docs/latest/)
- [Grafana](https://grafana.com/docs/grafana/latest/)
- [Kubernetes](https://kubernetes.io/docs/home/)
- [kubernetes-mcp-server](https://github.com/containers/kubernetes-mcp-server)
- [Grafana](https://grafana.com/)
- [Prometheus](https://prometheus.io/)
