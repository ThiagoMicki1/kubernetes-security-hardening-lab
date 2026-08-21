# Project Decisions

This lab stays intentionally small: plain Kubernetes YAML, one insecure example, one hardened example, and scanner output to compare them.

## Plain YAML instead of Helm

I skipped Helm because the point of this project is to make the security settings easy to see. Templates would be useful for a real reusable chart, but they would hide the controls a beginner should be able to explain.

## No automatic deployment

The GitHub Action only scans manifests. It does not run `kubectl apply`, use a kubeconfig, or connect to a cloud cluster. That keeps the project safe for a public portfolio.

## Default deny is strict on purpose

The hardened NetworkPolicy blocks ingress and egress by default. That is a useful demo posture, but a real application would normally need explicit egress for DNS, APIs, package mirrors, or observability endpoints.

## Secrets are placeholders

The Secret is fake sample data. The hardened manifest mounts it read-only to show a safer pattern, but a real deployment should use a managed secret workflow instead of committing secret values to Git.
