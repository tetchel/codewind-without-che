# Remote Codewind Without Che

This repository provides directions for Codewind developers to deploy a working Codewind instance onto a Kubernetes cluster, outside of an Eclipse Che workspace, which can then be connected to using the `hybrid` branch of [the VS Code extension](https://github.com/eclipse/codewind-vscode/tree/hybrid).

Expands on https://github.com/eclipse/codewind/issues/576#issuecomment-535938363
- Add secret creation instructions
- Add serviceaccount to yaml
- Deleted/renamed some fields that referred to Che or Microclimate or were no longer used

## Install Steps

These steps assume you want to use the namespace `codewind`.

1. Create secret with your Docker registry credentials

```
kubectl create secret docker-registry codewind-registry-secret \
    --docker-username=<username> \
    --docker-password=<password> \
    -n codewind
```

2. Download [`codewind.yaml`](https://raw.githubusercontent.com/tetchel/codewind-without-che/master/codewind.yaml).

2. Edit the CHANGE ME fields in `codewind.yaml`. Some are optional, but you must edit:
- DOCKER_REGISTRY_URL to the same username as `--docker-username` above (for Dockerhub - I haven't tried other registry providers).
- CHE_INGRESS_HOST to `codewind.<cluster IP>.nip.io`.
    - If using Docker for Desktop on Mac, see the prereqs below to determine your cluster IP.
- codewind-ingress spec.rules.host to the same host as above.
- codewind-performance deployment CODEWIND_INGRESS to the same host as above.
- If you plan to use a namespace other than `codewind`, edit KUBE_NAMESPACE (and make sure you created the secret into that namespace, above).

4. Deploy Codewind

`kubectl apply -f codewind.yaml -n codewind`

5. Codewind is now available at the ingress URL provided above.

`kubectl get ing -n codewind`

### Prereqs for Docker for Desktop (Mac)
- [Install ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/):
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml && \
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml
```
- Determine your cluster IP address and add it as an alias of localhost:
```
export CLUSTER_IP=$(kubectl get services --namespace ingress-nginx -o jsonpath='{.items[*].spec.clusterIP}') \
    && echo $CLUSTER_IP \
    && sudo ifconfig lo0 alias $CLUSTER_IP
```
