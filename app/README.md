# Nginx Deployment with AWS Load Balancer Controller

This directory contains Kubernetes manifests to deploy an Nginx application with an Application Load Balancer (ALB) on
Amazon EKS.

## Prerequisites

- AWS EKS cluster with AWS Load Balancer Controller installed
- AWS CLI configured with appropriate permissions
- `kubectl` configured to connect to your EKS cluster

## Files

- `deployment.yaml` - Nginx deployment configuration
- `service.yaml` - Kubernetes service configuration
- `ingress.yaml` - ALB ingress configuration
- `service-account.yaml` - Service account for AWS Load Balancer Controller

## Deployment Steps

2. **Apply the deployment**:
   ```bash
   kubectl apply -f deployment.yaml
   ```

3. **Apply the service**:
   ```bash
   kubectl apply -f service.yaml
   ```

4. **Apply the ingress**:
   ```bash
   kubectl apply -f ingress.yaml
   ```

## Verification

1. **Check deployment status**:
   ```bash
   kubectl get deployment nginx-deployment
   kubectl get pods -l app=nginx
   ```

2. **Check service status**:
   ```bash
   kubectl get service nginx-service
   ```

3. **Check ingress status**:
   ```bash
   kubectl get ingress nginx-ingress
   ```

4. **Get ALB hostname**:
   ```bash
   kubectl get ingress nginx-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
   ```

5. **Test the application**:
   ```bash
   # Replace <ALB-HOSTNAME> with the actual hostname from the previous command
   curl http://<ALB-HOSTNAME>/
   ```

6. **Check ALB details**:
   ```bash
   # Get detailed information about the ingress
   kubectl describe ingress nginx-ingress
   
   # Check the AWS console for the ALB
   aws elbv2 describe-load-balancers --names $(kubectl get ingress nginx-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}' | cut -d'-' -f1)
   ```

## Cleanup

To remove all resources:

```bash
kubectl delete -f ingress.yaml
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

## Notes

- The ALB may take a few minutes to provision after applying the ingress
- Ensure your VPC subnets are tagged appropriately for the ALB to work:
    - Public subnets: `kubernetes.io/role/elb`
    - Private subnets: `kubernetes.io/role/internal-elb`
- The ingress is configured as internet-facing with IP target type