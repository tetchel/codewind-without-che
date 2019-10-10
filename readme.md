# Remote Codewind Without Che

This repository provides directions for Codewind developers to deploy a working Codewind instance onto a K8s cluster, outside of an Eclipse Che workspace.

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

2. Edit the CHANGE ME fields in `codewind.yaml`. Some are optional, but you must edit:
- DOCKER_REGISTRY_URL to the same username as `--docker-username` above
- CHE_INGRESS_HOST to `codewind.<cluster IP>.nip.io`.
    - If using Docker for Desktop on Mac, see the prereqs below to determine your cluster IP.
- codewind-ingress spec.rules.host to the same host as above.
- codewind-performance deployment CODEWIND_INGRESS to the same host as above.
- If you plan to use a namespace other than `codewind`, edit KUBE_NAMESPACE (and make sure you created the secret into the same namespace, above).

3. Download `codewind.yaml` and deploy Codewind

`kubectl apply -f codewind.yaml -n codewind`

4. Codewind is now available at the ingress URL provided above.

`kubectl get ing -n codewind`

### Prereqs for Docker for Desktop (Mac)
- Install ingress-nginx:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml
```
- Determine your cluster IP address and add it as an alias of localhost:
```
export CLUSTER_IP=$(kubectl get services --namespace ingress-nginx -o jsonpath='{.items[*].spec.clusterIP}') \
    && echo $CLUSTER_IP \
    && sudo ifconfig lo0 alias $CLUSTER_IP
```
