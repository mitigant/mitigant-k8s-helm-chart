# Mitigant KSPM Operator Helm Charts

Helm charts for deploying the Mitigant Kubernetes Security Posture Management (KSPM) operator stack.

## Overview

The KSPM operator deploys a suite of Kubescape-based microservices into a Kubernetes cluster to provide:

- **Configuration scanning** — cluster-wide compliance and misconfiguration detection
- **Vulnerability scanning** — container image CVE analysis via Grype
- **Runtime observability** — eBPF-based workload monitoring via the node agent
- **Network policy generation** — automated network policy recommendations

## Repository Structure

```
charts/
├── kspm-operator/            Main application chart
│   ├── templates/            Kubernetes manifests for all components
│   ├── assets/               Static config assets (cronjob templates, egress rules)
│   └── tests/                Helm snapshot tests
└── dependency_chart/         CRD sub-charts
    ├── operatorcommand-crds/ OperatorCommand CRD
    └── servicescanresult-crds/ ServiceScanResult CRD
```

## Components

| Component | Kind | Description |
|---|---|---|
| kubescape | Deployment | Configuration and compliance scanner |
| operator | Deployment | Scan orchestrator and scheduler |
| kubevuln | Deployment | Image vulnerability scanner |
| kollector | StatefulSet | Data collector, reports to Mitigant cloud |
| gateway | Deployment | Relays cloud notifications to the operator |
| storage | Deployment + APIService | Aggregated API server for internal state |
| node-agent | DaemonSet | eBPF runtime observability |
| synchronizer | Deployment | Syncs cluster data with the Mitigant backend |
| otel-collector | Deployment | OpenTelemetry metrics pipeline |
| grype-offline-db | Deployment | Offline Grype vulnerability database |

## Prerequisites

- Kubernetes 1.24+
- Helm 3.x

## Installation

```bash
helm install kspm-operator ./charts/kspm-operator \
  --namespace mitigant \
  --create-namespace \
  --set clusterName="$(kubectl config current-context)" \
  --set account=<YOUR_ACCOUNT_ID> \
  --set accessKey=<YOUR_ACCESS_KEY> \
  --set server=<MITIGANT_SERVER_URL>
```

## Configuration

Key values to configure at install time:

| Parameter | Description | Default |
|---|---|---|
| `clusterName` | Name of the cluster (required) | `cluster` |
| `account` | Mitigant account GUID | — |
| `accessKey` | Mitigant agent access key | — |
| `server` | Mitigant backend server URL | — |
| `ksNamespace` | Namespace for all components | `mitigant` |
| `capabilities.configurationScan` | Enable config scanning | `enable` |
| `capabilities.vulnerabilityScan` | Enable image vuln scanning | `enable` |
| `capabilities.runtimeObservability` | Enable eBPF runtime monitoring | `enable` |
| `capabilities.networkPolicyService` | Enable network policy generation | `enable` |
| `global.networkPolicy.enabled` | Deploy NetworkPolicy resources | `false` |
| `global.httpsProxy` | HTTPS proxy URL | — |
| `openshift` | Enable OpenShift SCC support | `false` |

See [charts/kspm-operator/values.yaml](charts/kspm-operator/values.yaml) for the full list of configurable values.

## Upgrading

```bash
helm upgrade kspm-operator ./charts/kspm-operator --namespace mitigant
```

## Uninstalling

```bash
helm uninstall kspm-operator --namespace mitigant
```

CRDs are not removed automatically. To clean them up:

```bash
kubectl delete crd operatorcommands.kubescape.io
kubectl delete crd servicescanresults.kubescape.io
```

## Origin

This project is derived from the [Kubescape Helm Charts](https://github.com/kubescape/helm-charts) by the Kubescape Authors, adapted for the Mitigant KSPM product.

## License

This project is licensed under the Apache License 2.0 — see the [LICENSE](LICENSE) file for details.

Original work: Copyright 2022-2025 The Kubescape Authors
Modifications: Copyright 2024-2026 Mitigant GmbH
