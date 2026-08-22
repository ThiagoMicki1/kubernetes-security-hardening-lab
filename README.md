# Kubernetes Security Hardening Lab

[![Kubernetes Security Hardening Lab](https://github.com/ThiagoMicki1/kubernetes-security-hardening-lab/actions/workflows/kubernetes-security.yml/badge.svg)](https://github.com/ThiagoMicki1/kubernetes-security-hardening-lab/actions/workflows/kubernetes-security.yml)

Portfolio learning lab that compares intentionally insecure Kubernetes manifests with a hardened version of the same small web app.

This project is safe to publish. It does not include real secrets, real kubeconfig files, cloud credentials, or automatic deployment steps.

## Why I Built This

I built this to practice Kubernetes security in a way that is realistic but still beginner-friendly. I wanted a repo that shows more than "I can write YAML" by explaining why certain Kubernetes settings matter for least privilege, container hardening, network control, and safer deployments.

## What This Project Demonstrates

- Kubernetes manifest security review
- Container hardening basics
- RBAC and service account least privilege
- NetworkPolicy fundamentals
- Secure defaults such as non-root users and read-only filesystems
- IaC scanning with KubeLinter and Checkov
- GitHub Actions workflow security with read-only permissions
- Clear security documentation and reporting

## Project Scope

This is a local validation lab, not a production Kubernetes platform.

The default workflow validates and scans YAML only. It does not deploy to a cluster, connect to a cloud provider, or apply resources automatically.

If someone chooses to test the manifests later, they should use a local lab cluster such as kind, minikube, Docker Desktop Kubernetes, or another disposable environment.

## Folder Structure

```text
kubernetes-security-hardening-lab/
├── .github/workflows/
│   └── kubernetes-security.yml
├── docs/
│   ├── decisions.md
│   ├── local-kind-walkthrough.md
│   └── security-controls.md
├── manifests/
│   ├── hardened/
│   │   └── hardened-app.yaml
│   └── insecure/
│       └── insecure-app.yaml
├── reports/
│   ├── sample-checkov-output.txt
│   └── sample-kube-linter-output.txt
├── .checkov.yml
├── .gitattributes
├── .gitignore
└── README.md
```

## Insecure vs Hardened Comparison

| Area | Insecure manifest | Hardened manifest |
| --- | --- | --- |
| Container user | Runs as root | Runs as a non-root user |
| Privileged mode | Uses `privileged: true` | Does not use privileged mode |
| Privilege escalation | Allowed | Disabled |
| Linux capabilities | Adds risky capabilities | Drops all capabilities |
| Root filesystem | Writable | Read-only root filesystem |
| Host access | Mounts Docker socket with `hostPath` | Avoids hostPath |
| Resources | No requests or limits | Uses CPU and memory requests/limits |
| Health checks | No probes | Includes readiness and liveness probes |
| Service account | Uses default behavior | Uses a named service account |
| RBAC | No least-privilege example | Restricted Role and RoleBinding |
| Network controls | No NetworkPolicy | Default deny ingress and egress, plus limited ingress |
| Image tag | Uses `latest` | Uses a versioned unprivileged image and documents digest pinning |

## Included Kubernetes Security Controls

- Namespace separation
- Pod Security labels using `restricted`
- Non-root container execution
- Read-only root filesystem
- Dropped Linux capabilities
- Privilege escalation disabled
- Resource requests and limits
- Liveness and readiness probes
- Restricted RBAC
- Named service account with token automount disabled
- NetworkPolicy default deny pattern
- Safe placeholder Secret
- Secret mounted as a read-only file
- ConfigMap for non-sensitive configuration
- Versioned container image tag

More detail is in [docs/security-controls.md](docs/security-controls.md).

Design notes are in [docs/decisions.md](docs/decisions.md).

## Local Validation

Install the tools you want to use locally:

- KubeLinter: https://github.com/stackrox/kube-linter
- Checkov: https://github.com/bridgecrewio/checkov

Run KubeLinter against the hardened manifests:

```bash
kube-linter lint manifests/hardened
```

Run KubeLinter against the intentionally insecure examples:

```bash
kube-linter lint manifests/insecure || true
```

Run Checkov against the hardened manifests:

```bash
checkov -d manifests/hardened --framework kubernetes
```

Run Checkov against the insecure examples:

```bash
checkov -d manifests/insecure --framework kubernetes || true
```

## Optional Local Lab Deployment

Do not apply these manifests to a real cloud cluster unless you understand the impact.

For a disposable local cluster only, you could apply the hardened example:

```bash
kubectl apply -f manifests/hardened/
```

Clean it up afterward:

```bash
kubectl delete namespace hardened-demo
```

The insecure manifests are included for learning and scanner comparison. They are intentionally unsafe.

A safer step-by-step local cluster walkthrough is available in [docs/local-kind-walkthrough.md](docs/local-kind-walkthrough.md).

## GitHub Actions

The workflow in `.github/workflows/kubernetes-security.yml`:

- Checks out the repo
- Installs KubeLinter
- Scans the hardened manifests
- Scans the insecure examples without failing the workflow
- Runs Checkov against the hardened manifests
- Uses `permissions: contents: read`

## Sample Output

Sample scanner outputs are included in:

- [reports/sample-kube-linter-output.txt](reports/sample-kube-linter-output.txt)
- [reports/sample-checkov-output.txt](reports/sample-checkov-output.txt)

## What I Learned

I learned that Kubernetes hardening is mostly about reducing unnecessary privileges. A few settings like `runAsNonRoot`, `allowPrivilegeEscalation: false`, dropping capabilities, and avoiding `hostPath` can make a big difference.

I also learned that scanner output needs human review. A scanner can point out risk, but I still need to understand whether the finding matters for the workload.

## What I Struggled With

The hardest part was keeping the lab simple. It is tempting to add Helm, admission controllers, policy engines, and a full cluster setup, but that would make the repo harder to understand. I kept it to plain YAML so the security controls are easy to see.

## Interview Talking Points

- Why `privileged: true`, `hostPath`, and Docker socket mounts are high-risk.
- How the hardened manifest reduces blast radius with non-root, dropped capabilities, and read-only filesystem settings.
- Why the NetworkPolicy is intentionally strict in the lab and what egress rules a real app might need.

## Future Improvements

- Add a small policy-as-code example with OPA Gatekeeper or Kyverno once the plain YAML version is fully understood.

## Safety Notes

- No real secrets are included.
- No kubeconfig files are included.
- No cloud credentials are required.
- GitHub Actions never runs `kubectl apply`.
- The insecure manifest is intentionally unsafe and should not be deployed outside a disposable lab.
