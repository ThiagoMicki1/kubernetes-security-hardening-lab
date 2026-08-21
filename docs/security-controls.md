# Kubernetes Security Controls

This lab compares risky Kubernetes YAML against a hardened version of the same small web app. The goal is not to make a perfect production template. The goal is to show that I understand common Kubernetes misconfigurations and how to reduce risk with readable manifests.

## Namespace Separation

The hardened app runs in its own `hardened-demo` namespace. Namespaces help organize resources and make it easier to apply policies to one workload without affecting everything else in the cluster.

The hardened namespace also includes Pod Security labels set to `restricted`. In a real cluster, these labels help prevent pods from using risky settings like privileged containers or host namespace access.

## Non-Root Container User

The hardened deployment sets:

```yaml
runAsNonRoot: true
runAsUser: 10001
runAsGroup: 10001
```

Running as root inside a container increases the impact of a container escape or misconfiguration. Running as a non-root user limits what the process can do.

## Read-Only Root Filesystem

The hardened container sets:

```yaml
readOnlyRootFilesystem: true
```

This makes it harder for an attacker or compromised process to write tools, scripts, or modified files into the container image filesystem. The deployment uses small `emptyDir` volumes only where nginx needs writable runtime paths.

## Dropped Linux Capabilities

The hardened container drops all Linux capabilities:

```yaml
capabilities:
  drop:
    - ALL
```

Capabilities split root-level privileges into smaller permissions. Dropping them reduces what a compromised container can do.

## Privilege Escalation Disabled

The hardened container sets:

```yaml
allowPrivilegeEscalation: false
```

This helps prevent a process from gaining more privileges than it started with.

## No Privileged Container

The insecure deployment sets `privileged: true`, which gives the container broad access to the host. The hardened deployment does not use privileged mode.

## No hostPath Volume

The insecure deployment mounts `/var/run/docker.sock` from the host. That is intentionally unsafe because access to the container runtime socket can lead to host-level compromise.

The hardened deployment avoids `hostPath` and uses temporary `emptyDir` volumes only for container runtime paths.

## Resource Requests and Limits

The hardened deployment includes CPU and memory requests and limits. This helps prevent one workload from consuming too many cluster resources and affecting other workloads.

## Readiness and Liveness Probes

The hardened deployment includes readiness and liveness probes.

Readiness probes tell Kubernetes when the app is ready to receive traffic. Liveness probes help Kubernetes restart the container if it becomes unhealthy.

## Restricted RBAC

The hardened app uses a named service account and a small Role that can only `get` one ConfigMap.

This demonstrates least privilege: the workload should only have the Kubernetes API permissions it actually needs.

## Avoid Default Service Account Usage

The hardened deployment uses:

```yaml
serviceAccountName: webapp-sa
automountServiceAccountToken: false
```

Avoiding the default service account makes permissions easier to reason about. Disabling automatic token mounting helps reduce unnecessary credential exposure inside pods.

## NetworkPolicy

The hardened manifests include a default deny NetworkPolicy and a second policy that allows traffic to the web app only from pods in the same namespace.

NetworkPolicy is useful because Kubernetes networking is often open by default unless a compatible network plugin enforces policies.

## Safe Placeholder Secret

The Secret uses a fake placeholder value only:

```yaml
API_TOKEN: "REPLACE_WITH_LOCAL_LAB_VALUE_ONLY"
```

This project never uses real secrets. In real environments, secrets should be managed through a secure secret manager or a controlled deployment process.

The hardened deployment mounts the Secret as a read-only file instead of passing it as an environment variable. This is a safer pattern because environment variables are often easier to expose through debug output, process inspection, or crash reports.

## Image Tag Pinning

The insecure manifest uses `nginx:latest`, which can change unexpectedly.

The hardened manifest uses a versioned image tag:

```yaml
nginxinc/nginx-unprivileged:1.29-alpine
```

A digest pin would be stronger for production because it locks the image to an exact build. I kept a readable version tag here because this is a beginner portfolio lab.
