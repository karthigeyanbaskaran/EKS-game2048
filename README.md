# EKS-game2048
## Create an EKS Cluster
## Create a new EKS cluster named "game" in the "ap-south-1" region using Fargate:
```
eksctl create cluster --name game --region ap-south-1 --fargate
```

## Update your kubeconfig to add the new context for the EKS cluster:

```
aws eks update-kubeconfig --name game --region ap-south-1
# Output: Added new context arn:aws:eks:ap-south-1:590183981315:cluster/game to /Users/path/.kube/config
```

## Create a Fargate profile for the cluster:

```
eksctl create fargateprofile \
    --cluster game \
    --region ap-south-1 \
    --name alb-game-app \
    --namespace game-2048
```

## Deploy the Application
## Apply the Kubernetes configuration for the 2048 game:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

## Handling AWS CLI Issues on macOS
## If you encounter the following AWS CLI error on macOS:

```
error: exec plugin: invalid apiVersion "client.authentication.k8s.io/v1alpha1"
```

## Or:

```
Unable to connect to the server: getting credentials: decoding stdout: no kind "ExecCredential" is registered for version "client.authentication.k8s.io/v1alpha1" in scheme "pkg/client/auth/exec/exec.go:62"
```
## This is a known issue with the AWS CLI. To resolve it, try upgrading or downgrading the AWS CLI. If the issue persists, consider using an EC2 instance for this process.

## To install AWS CLI v2:

```
# Download AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
# Uncompress the archive
unzip awscliv2.zip
# Run the installer
sudo ./aws/install
```

## Associate IAM OIDC Provider
## Associate the IAM OIDC provider with your cluster:

```
eksctl utils associate-iam-oidc-provider --cluster game --approve
```

## Create IAM Policy and Service Account
## Download the IAM policy document:
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

## Create an IAM policy:

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

## Create an IAM service account:

```
eksctl create iamserviceaccount \
  --cluster=game \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::590183981315:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

## Install AWS Load Balancer Controller
## Add the EKS Helm chart repository:
```
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
```

## Install the AWS Load Balancer Controller using Helm:


```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=game \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-1 \
  --set vpcId=vpc-0903b6272944f1722
```

## Verify Ingress Creation
## To check if the Ingress has created a Load Balancer, run:

```
kubectl get ingress -n game-2048
```
## Delete the Cluster
## To delete the cluster, use the following command:

```
eksctl delete cluster --region=ap-south-1 --name=game
```
