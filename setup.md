# Demo setup instructions

Install [microk8s](https://microk8s.io/docs/getting-started):

```console
snap install microk8s --classic
sudo usermod -a -G microk8s $USER
echo "alias mk='microk8s kubectl'; complete -o default -F __start_kubectl mk" >> ~/.bashrc
newgrp microk8s
```

Test the deployment with:

```conole
mk create deployment nginx --image=nginx
# ovverride needed to specify a specific nodeport rather than random one
mk expose deployment nginx --type=NodePort --port 80 --overrides '{ "apiVersion": "v1","spec":{"ports": [{"port":80,"protocol":"TCP","targetPort":80,"nodePort":30080}]}}'

curl localhost:30080

mk delete svc/nginx deploy/nginx
```

Install [Argo CD](https://argo-cd.readthedocs.io/en/stable/getting_started/):

```console
mk create namespace argocd
mk apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Patch argocd-server to expose it with Nodeport on known ports:
```console
mk -n argocd patch svc/argocd-server -p '{"spec":{"ports":[{"port":80,"nodePort":32080}, {"port":443,"nodePort":32443}], "type": "NodePort"}}'
```

Patch the repo server to refresh more often:
```console
mk -n argocd set env deploy/argocd-repo-server "ARGOCD_RECONCILIATION_TIMEOUT=10s"
```

Extract argo cd initial random password to login to http://localhost:32080:
```console
mk -n argocd get secret/argocd-initial-admin-secret -o json |jq ".data | map_values(@base64d)"
```

## Debug

If your IPs changed from the initial microk8s setup (e.g. you switched WiFi network), refresh the CA certs
([SO](https://stackoverflow.com/questions/63451290/),
[argo docs](https://argo-cd.readthedocs.io/en/stable/operator-manual/tls/)):

```console
sudo microk8s refresh-certs -e ca.crt
```
