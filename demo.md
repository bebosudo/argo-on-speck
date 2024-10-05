# Demo day

## Prerequisites

Set up microk8s and Argo CD as per [setup.md](setup.md).

## Demo steps

### Argo CD Applications demo

- [ ] Create a classic Argo CD Application:
      `mk apply -f argo-applications/hello-world/my-cluster1/application.yaml`
- [ ] Check the application in the Argo CD UI: http://localhost:32080
- [ ] Check the application in the browser: http://localhost:30080
- [ ] Bump the image version in the `hello-world` application at `manifests/hello-world/base/deployment.yaml`
- [ ] Watch Argo CD sync it: may need a bump on the UI or CLI to trigger the sync.
- [ ] Wait for the sync to finish, or force it with: `mk -n argocd patch app/hello-world -p '{"operation": {"initiatedBy": {"username": "alberto"}, "sync": {"syncStrategy": {"hook": {}}}}}' --type merge`
- [ ] Check the new version of the application at http://localhost:30080

### Argo CD Application Sets demo

- [ ] Delete the `hello-world` application:
      `mk delete -f argo-applications/hello-world/my-cluster1/application.yaml`
- [ ] Create an Argo CD Application Set:
      `mk apply -f argo-application-sets/hello-world.yaml`
- [ ] Repeat the image version bump
- [ ] Explain that the bump would have applied to all the applications in the set
- [ ] After the demo is complete, delete the `hello-world` appset:
      `mk delete -f argo-applications/hello-world/my-cluster1/application.yaml`
