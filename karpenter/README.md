# Karpenter Configuration

This directory contains Karpenter manifests managed by ArgoCD, including NodePool and EC2NodeClass configurations.

## Prerequisites

- Karpenter Helm release must be deployed (via Terraform with IAM roles configured)
- AWS IAM roles and policies must be properly set up (KarpenterControllerRole, KarpenterNodeRole)
- Subnets and security groups must be tagged with `karpenter.sh/discovery: "true"` for auto-discovery

## Manifests

### namespace.yaml
Creates the `karpenter` namespace for all Karpenter resources.

### ec2-node-class.yaml
Defines the EC2NodeClass with:
- AL2 AMI family
- 32GB gp3 EBS volumes
- Discovery-based subnet and security group selection
- Production tags for resource tracking

### node-pool.yaml
Defines the NodePool with:
- **Instance Type**: t4g.medium (Graviton2 ARM64)
- **Architecture**: arm64
- **Capacity Type**: on-demand
- **Scaling Limits**: 
  - CPU limit: 1000
  - Memory limit: 1000Gi
  - 10% consolidation budget
- **Disruption**: Cost-based consolidation with 30-second consolidation intervals
- **Node Taints**: workload-type=general:NoSchedule (workloads must tolerate this)

### kustomization.yaml
Kustomize configuration for applying all resources together, with common labels and annotations.

## ArgoCD Integration

Simply point your ArgoCD Application to this `karpenter` folder. ArgoCD will automatically discover and apply all manifests using the `kustomization.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: karpenter-config
  namespace: argocd
spec:
  project: default
  source:
    repoURL: <your-git-repo>
    targetRevision: HEAD
    path: karpenter
  destination:
    server: https://kubernetes.default.svc
    namespace: karpenter
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

The `kustomization.yaml` ensures all resources (namespace, EC2NodeClass, and NodePool) are applied together with proper labels and annotations. ArgoCD will detect changes automatically when you push updates to this folder.

## Customization

Update the following in `node-pool.yaml` if needed:
- Instance types in `spec.template.spec.requirements[].values`
- Scaling limits in `spec.limits`
- Consolidation settings in `spec.disruption`
- Node taints and labels in `spec.template.metadata.labels` and `spec.template.spec.taints`

## Notes

- Nodes managed by Karpenter will be automatically created and deleted based on pod resource requests
- Workloads must tolerate the `workload-type=general:NoSchedule` taint to be scheduled on Karpenter nodes
- Monitor Karpenter controller logs for scaling decisions: `kubectl logs -n karpenter -l app.kubernetes.io/name=karpenter -f`
