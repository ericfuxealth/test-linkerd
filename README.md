We are having issues in the integration cluster with
- Linkerd
- Pod Security Group
- Cilium network policy

This is to reproduce it in a standalone EKS cluser

# Steps

1. Create an EKS cluster

One is created in the dev account called ericfu-test4
https://us-west-2.console.aws.amazon.com/eks/home?region=us-west-2#/clusters/ericfu-test4?selectedTab=cluster-configuration-tab



2. Install Cilium

```bash
helm upgrade --install cilium cilium/cilium --version 1.11.3 \
  --namespace kube-system \
  --set cni.chainingMode=aws-cni \
  --set enableIPv4Masquerade=false \
  --set tunnel=disabled \
  --set hubble.relay.enabled=true \
  --set hubble.ui.enabled=true
```

3. Install Linkerd

```bash
linkerd check --pre
linkerd install | kubectl apply -f -
linkerd check
```

4. Run the nginx app

```bash
kubectx create namespace test2
kubectl -n test2  apply -f nginx/nginx.yaml

```

# Nginx app

In the app, you should see these configured

- Linkerd
- Pod Security Group
- Cilium network policy

The pod works with just network policy and Pod SG.
The pod stops working once Linkerd is added. Error

```bash
Readiness probe failed: Get "http://10.10.37.71:4191/ready": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
```

# What works (and not)

| Linkerd | Pod SG | Network policy | Works |
| ------- | ------ | -------------- | ----- |
| X       | X      | X              | NO    |
|         | X      | X              | YES   |
| X       | X      |                | YES   |
| X       |        | X              | YES   |



