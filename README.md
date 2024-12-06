
# Advanced Kubernetes Observability with Prometheus

## Overview

This guide provides a comprehensive approach to setting up Prometheus Operators on a Kubernetes cluster, enabling robust observability and monitoring capabilities. It is designed for environments leveraging Role-Based Access Control (RBAC).

The guide is structured into two parts:
- Deploying and configuring Prometheus Operators.
- Adding custom services for monitoring with Prometheus templates.

---

## Repository Access

Clone this repository to get started with your observability setup:

```bash
git clone https://github.com/k8s-ansible-monitor
cd k8s-ansible-monitor
```

---

## Prerequisites

Before diving in, ensure the following are in place:
- Basic understanding of [Ansible](https://en.wikipedia.org/wiki/Ansible_(software)) and a prepared environment for its configuration.
- Access to your Kubernetes cluster via `~/.kube/config`.
- Secrets stored in Vault, specifically an `admin_pass` for Grafana.

The playbook uses `ansible-kubernetes-module` to interact with the Kubernetes cluster and fetch secrets from Vault for secure configurations.

---

## Step 1: Deploying Prometheus Operators

### Configure Secrets
Ensure that the `admin_pass` secret is available in your Vault. Specify its location using the `vault_path` variable.

### Install Prometheus Operators
Run the following command to deploy Prometheus Operators and set up Grafana dashboards:

```bash
ansible-playbook -e vault_path=/mypath setup_prometheus.yml
```

After installation, access Grafana at `http://<kubernetes-server>:30902` to view cluster metrics.

---

## Step 2: Adding Services to Prometheus

### Using the Prometheus-Service Role
The reusable `prometheus-service` Ansible role simplifies adding services to Prometheus. An test application is provided for demonstration.

#### Deploy the test application
Run the following command to deploy a sample app:

```bash
ansible-playbook examples/test_deployment.yml
```

#### Add the Application to Prometheus
Define the service configuration in `setup_services.yml`:

```yaml
---
- hosts: localhost
  gather_facts: no
  tasks:
    - include_role:
        name: ansible-kubernetes-modules
      vars:
        install_python_requirements: no

    - include_role:
        name: prometheus-service
      vars:
        app_name: "test-app"
        port_name: "web"
        port: "8080"
        metrics_path: "/metrics"
        namespace: default
        selector:
          app: "test-app"
```

#### Deploy the Service Configuration
Run the following to integrate the service into Prometheus:

```bash
ansible-playbook setup_services.yml
```

Verify the service NodePort by running:

```bash
kubectl get svc test-app-prom
```

---

## Conclusion

With this setup, you’ll establish a robust monitoring system for your Kubernetes cluster, enabling precise and actionable observability with Prometheus Operators.

---
