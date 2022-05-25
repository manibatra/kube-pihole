# Kube PiHole

A lean configuration for running Pi Hole in a local Kubernetes cluster. Ideal for homelabs. Inspired by [docker pi-hole](https://github.com/pi-hole/docker-pi-hole)


## Installation 

We assume a running K8s/K3s cluster.

1. Create volumes to be used by the pi-hole deployment : 

```bash
for $v in vol1 vol2; do
  mkdir -p /mnt/disks/$v
  sudo mount -t tmpfs $v /mnt/disks/$v
done
```
2. Create the admin password for pi-hole, base64 encode it and it to `./pi-hole/k3s/secret.yaml`
3. Apply the pi-hole YAMLs : `kubectl apply -f ./pi-hole/k3s/`

The repository comes with manifests to set up [ingress-nginx](https://kubernetes.github.io/ingress-nginx/) services to access the pi-hole admin interface and forward the DNS queries to the pi-hole service.
1. Install the helm chart with overrides : `helm upgrade --namespace ingress-nginx --repo https://kubernetes.github.io/ingress-nginx ingress-nginx ingress-nginx -f ./ingress-ngnx/helm/values/overrides.yaml`
2. Apply the YAMLs for the udp service and load balancer : `k apply -f ./k3s/`
3. Disable systemd-resolve service to stop it from listening on port 53 : `systemctl disable systemd-resolve`
4. Set IP of one of your nodes as your DNS server. 


## Notes 
- We create a separate load balancer for the UDP protocol because the service type `LoadBalancer` does not support a multi protocol setup. though the support is in the [pipeline](https://github.com/kubernetes/enhancements/issues/1435). 
- The UDP service is not exposed using helm chart values because doing so forces helm to add the service to the TCP load balancer leading to a failure due to the reason mentioned above. 

## TO DO
- [ ] IPv6 support
