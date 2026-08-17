# Project Context

## Goal

Teach Kubernetes in Traditional Chinese through hands-on experiments based on the official PHP Guestbook with Redis tutorial. Reuse Guestbook across Levels 1–5 so each new concept extends a familiar application.

Official baseline:
https://kubernetes.io/docs/tutorials/stateless-application/guestbook/

## Learner environment

- Commands run inside Ubuntu WSL 2 unless explicitly marked as PowerShell.
- kubectl context: `minikube`
- kubectl: 1.36.3
- Minikube: 1.38.1
- Kubernetes: 1.35.1
- Container runtime observed on the node: Docker 29.2.1
- Local frontend access: `kubectl port-forward svc/frontend 8080:80`

Verified on 2026-08-17: the Minikube node was Ready, and the baseline Guestbook had 3 healthy Deployments, 3 ReplicaSets, 6 Pods, and 3 application Services.

## Curriculum

1. Deployment, ReplicaSet, Pod, Service, labels, selectors, EndpointSlices.
2. Pod deletion and self-healing, manual scaling from 1 to 5, Service discovery.
3. Image changes, rolling updates, rollout status/history/undo.
4. ConfigMap, Secret, requests/limits, liveness and readiness probes.
5. Namespace, Ingress, HPA, PVC/PV.

## Teaching conventions

- Explain the expected result before asking the learner to run a command.
- Use a cycle of concept, command, observation, deliberate failure, explanation, and checkpoint.
- Prefer read-only inspection first (`get`, `describe`, `logs`) before mutations.
- State which resource is the controller and which is the managed object.
- Use labels/selectors explicitly when explaining Service routing.
- Distinguish desired state from current state.
- Warn before commands that delete the cluster or stored data.
- Do not describe Kubernetes Secret as encrypted by default.
- Explain that Ingress requires an Ingress Controller and HPA requires a metrics pipeline.
- Treat the Guestbook Redis topology as a learning example, not a production HA design.

## Current lesson

Start at Level 1. Use the running baseline resources without recreating them. The first checkpoint is for the learner to explain:

1. Why deleting a Pod does not delete its Deployment.
2. Why a frontend Service can route to three changing Pod IPs.
3. Why ReplicaSet names and Pod names contain generated hashes/suffixes.
