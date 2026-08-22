# Local kind Walkthrough

This walkthrough is optional. The default project workflow is still scan-only and does not deploy anything.

Use this only with a disposable local cluster, never a shared or production cluster.

## Prerequisites

- Docker running locally
- `kind`
- `kubectl`

## Create A Disposable Cluster

```bash
kind create cluster --name k8s-hardening-lab
```

## Apply The Hardened Example

```bash
kubectl apply -f manifests/hardened/
```

Check the resources:

```bash
kubectl get all -n hardened-demo
kubectl get networkpolicy -n hardened-demo
kubectl describe pod -n hardened-demo -l app=hardened-webapp
```

## What To Look For

- The pod uses `webapp-sa`, not the default service account.
- `automountServiceAccountToken` is disabled.
- The container runs as UID `10001`, not root.
- The container drops Linux capabilities and disables privilege escalation.
- The root filesystem is read-only.
- The default-deny NetworkPolicy includes both ingress and egress.

## Cleanup

```bash
kind delete cluster --name k8s-hardening-lab
```

## Notes

The insecure manifest exists for scanner comparison. I would not apply it unless I was using a throwaway local lab and specifically testing scanner behavior.
