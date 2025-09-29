# Demo setup instructions

Install [microk8s](https://microk8s.io/docs/getting-started):

```console
sudo ln -s /var/lib/snapd/snap /snap # required by microk8s
sudo snap install microk8s --classic
sudo usermod -a -G microk8s $USER
echo "alias mk='microk8s kubectl'; complete -o default -F __start_kubectl mk" >> ~/.bashrc
```

Now either reboot to get the new group to work, or force a shell to load with that group with `newgrp microk8s`.

Test the deployment with:

```console
# fire a nginx server
mk create deployment nginx --image=nginx
# create a service resource with specific nodeport rather than random one
mk expose deployment nginx --type=NodePort --port 80 --overrides '{ "apiVersion": "v1","spec":{"ports": [{"port":80,"protocol":"TCP","targetPort":80,"nodePort":30080}]}}'
# test the nginx server at the specified port
curl localhost:30080
# cleanup the test resources created
mk delete svc/nginx deploy/nginx
```

Install [Argo CD](https://argo-cd.readthedocs.io/en/stable/getting_started/):

```console
mk create namespace argocd
mk apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.1.7/manifests/install.yaml
```

Patch argocd-server to expose it with Nodeport on known ports:
```console
mk -n argocd patch svc/argocd-server -p '{"spec":{"ports":[{"port":80,"nodePort":32080}, {"port":443,"nodePort":32443}], "type": "NodePort"}}'
```

Patch the repo server to refresh more often and speed up the demo:
```console
mk -n argocd set env deploy/argocd-repo-server "ARGOCD_RECONCILIATION_TIMEOUT=10s"
```

Wait for all pods to be running, then extract the Argo CD initial random password to login to http://localhost:32080:
```console
mk -n argocd get secret/argocd-initial-admin-secret -o json |jq ".data | map_values(@base64d)"
```

## Troubleshooting

If your IPs changed from the initial microk8s setup (e.g. you switched WiFi network), refresh the CA certs
([SO](https://stackoverflow.com/questions/63451290/),
[argo docs](https://argo-cd.readthedocs.io/en/stable/operator-manual/tls/)):

```console
sudo microk8s refresh-certs -e ca.crt
```
