# Ingress Controller
[Refer Here](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/)

- Create OIDC provider
    ```
        eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster roboshop-dev --approve
    ```

- Download IAM policy of cluster with any other regin (for us gov & china select respective ones)
    ```
        curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v3.6.0/docs/install/iam_policy.json
    ```

- Create IAM policy **AWSLoadBalancerControllerIAMPolicy**
    ```
        aws iam create-policy \
            --policy-name AWSLoadBalancerControllerIAMPolicy \
            --policy-document file://iam-policy.json
    ```
  ***Note***- note down policy arn 
    eg: arn:aws:iam::055610219795:policy/AWSLoadBalancerControllerIAMPolicy

- Create service account and IAM role, also map above policy
    ```
        eksctl create iamserviceaccount \
        --cluster=roboshop-dev \
        --namespace=kube-system \
        --name=aws-load-balancer-controller \
        --attach-policy-arn=arn:aws:iam::055610219795:policy/AWSLoadBalancerControllerIAMPolicy \
        --override-existing-serviceaccounts \
        --region us-east-1 \
        --approve
    ```

## Install Drivers
***Note*** - Install helm using below commands
    ```
        curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
        chmod 700 get_helm.sh
        ./get_helm.sh
    ```

- add eks charts
    ```
        helm repo add eks https://aws.github.io/eks-charts
    ```
- Install aws-load-balancer-controller
    ```
        helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=roboshop-dev --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
    ```
- Check drivers installed or not using below command
    ```
        kubectl get pods -n kube-system
    ```

We have created all drivers are required for aws lb controller

Once above done we need ingress resource yaml to create rules, target groups and listeners

annotations are important in ingress resource
Defined below annotations
    - alb.ingress.kubernetes.io/listen-ports: as we use https point to 443
    - alb.ingress.kubernetes.io/certificate-arn: create a certificate in cert manager and copy its arn
    - alb.ingress.kubernetes.io/scheme: as we want alb to be public like keep it as internet-facing
    - alb.ingress.kubernetes.io/tags: we can provide tags
    - alb.ingress.kubernetes.io/target-type: it is required for communication with pods
    - alb.ingress.kubernetes.io/group.name: route traffic from the Application Load Balancer (ALB) to your Kubernetes cluster. ALB routes network traffic directly to the Pod's private IP address