### install flux
```shell
# https://fluxcd.io/legacy/flux/references/fluxctl/
brew install fluxcd/tap/flux
brew install fluxctl

# connect fluxctl to the daemon
fluxctl --k8s-fwd-ns=<namespace> list-workloads
# allow flux to generate ssh-key
fluxctl identity
```
### create repo
```shell
gh repo create FluxCDExample -d "FLUX CD"  --public
```
### export ENV
```shell
export GITHUB_USER=donwany
export GITHUB_TOKEN="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"
```
### create namespace
```shell
kubectl create ns flux-system
```

### Check your Kubernetes cluster
```shell
flux check --pre
```
```shell
kubectl create ns flux

export GHUSER="donwany"

fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-url=git@github.com:${GHUSER}/flux-get-started \
--git-path=namespaces,workloads \
--namespace=flux | kubectl apply -f -

# check rollout status
kubectl -n flux rollout status deployment/flux
# Giving write access
fluxctl identity --k8s-fwd-ns flux
# setup ssh key
fluxctl identity --k8s-fwd-ns flux
# syn repo: manually
fluxctl sync --k8s-fwd-ns flux
```

### Install Flux onto your cluster, Create fleet-infra repository
```shell
flux bootstrap github \
--owner=$GITHUB_USER \
--repository=FluxCDExample \
--branch=main \
--personal=true \
--path=dev \
--token-auth \
--private=false
```
```shell
git clone https://github.com/$GITHUB_USER/FluxCDExample
cd FluxCDExample

flux create source git fluxpodinfo \
--url=https://github.com/donwany/FluxCDExample \
--branch=main \
--interval=30s \
--export > fluxpodinfo-source.yaml

flux create source git podinfo \
--url=https://github.com/stefanprodan/podinfo \
--branch=master \
--interval=30s \
--export > podinfo-source.yaml

# push to repo
git add -A && git commit -m "Add podinfo GitRepository"
git push

flux create kustomization podinfo \
--target-namespace=default \
--source=podinfo \
--path="./kustomize" \
--prune=true \
--interval=5m \
--export > podinfo-kustomization.yaml

git add -A && git commit -m "Add podinfo Kustomization"
git push

flux get kustomizations --watch
kubectl -n default get deployments,services
```