# EKS CD via Azure Pipelines (ALB Ingress)

This pack contains:
- `azure-pipelines-cd.yml` – CI/CD pipeline that authenticates to AWS, targets your EKS cluster, applies manifests, and prints a clickable URL.
- `k8s/deployment.yaml` – 2‑replica Deployment of your app (container on port 8080).
- `k8s/service.yaml` – ClusterIP Service (Ingress routes to it).
- `k8s/ingress.yaml` – AWS ALB Ingress (internet-facing), optional TLS by ACM.

## Quick start

1. **Prereqs**
   - EKS is running and your IAM principal (or assumed role) has kubectl access (aws-auth mapped).
   - **AWS Load Balancer Controller** installed on the cluster (IRSA). Verify:
     ```bash
     kubectl -n kube-system get deploy aws-load-balancer-controller
     ```

2. **Edit variables** in `azure-pipelines-cd.yml` (or set them as pipeline variables in Azure DevOps):
   - `DOCKER_IMAGE` e.g. `menaosman/github-action-app`
   - `IMAGE_TAG` e.g. `latest` or your build tag
   - `AWS_REGION`, `EKS_CLUSTER_NAME`, `K8S_NAMESPACE` (e.g., `prod`)
   - Optional: `ACM_CERT_ARN` for HTTPS via ALB

3. **Create secrets in Azure Pipelines** (Pipeline > Edit > Variables or Library):
   - `AWS_ACCESS_KEY_ID` (secret)
   - `AWS_SECRET_ACCESS_KEY` (secret)
   - Optional: `AWS_ROLE_ARN` (secret) if you prefer assuming a role
   - Optional: `ACM_CERT_ARN` (secret) to enable HTTPS

4. **Commit & run**
   - Push these files to your repo root.
   - Create pipeline from existing YAML (`azure-pipelines-cd.yml`).
   - Run. When finished, job summary shows **Open your app: https://<ALB-hostname>**.

5. **Troubleshooting**
   - Ingress pending hostname? Check controller:
     ```bash
     kubectl -n kube-system logs deploy/aws-load-balancer-controller | tail -n 50
     kubectl -n prod get ingress,myapp -o wide
     ```
   - 403/5xx? Verify healthcheck path `/`, target port 8080, and security groups opened by the ALB.
