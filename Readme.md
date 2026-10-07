# Prerequisites
Install and run https://docs.docker.com/desktop/   
Install `kubectl` https://kubernetes.io/docs/tasks/tools/.  
Install `helm`https://helm.sh/docs/intro/install/
Install `minikube` https://minikube.sigs.k8s.io/docs/start and have it activated with 

# 1. Create Namespace For Lago Resources To Be Deployed To
```
kubectl apply -f namespace.yaml
```

# How Parent Chart Was Created
`https://helm.sh/docs/helm/helm_create/`

you do not need to run this one it's just for reference

```
helm create lago-wrapper-app
```

# 2. Install Lago Dependency And Update charts folder to have lago(also useful for updating chart.lock after undate to new chart versions)
> Note lago subchart was created using https://getlago.com/docs/guide/lago-self-hosted/kubernetes guide, but used as Helm SubChart 
```
helm repo update
helm dependency update lago-wrapper-app
```

# Other Dependency Charts App Contains

https://artifacthub.io/packages/helm/bitnami/redis?modal=install

> Note partman extension is required in Postgres https://artifacthub.io/packages/helm/bitnami/postgresql/18.12.4?modal=install so this chart count not be used,  
> Had to use https://hub.docker.com/layers/getlago/postgres-partman/latest image that lago uses in their own test setup, lago-wrapper-app/templates/postgresql.yaml has it.


# Secrets
The following secrets in values is generated as per `https://getlago.com/docs/guide/lago-self-hosted/kubernetes`

```
    encryption:
      key: "71669fdeee5b8776"
      salt: "76f5b3f8b147293"

    # Generate rsa with: openssl genrsa 2048 | openssl base64 -A
    # Generate hmac with: openssl rand -base64 16
    signing:
      rsa: "..."
      hmac: "+q1OyAJUB0ytlWeHFuzhnw=="
```
Don't use it in prod it's just for local testing

# 3. Install AgroCD
https://argo-cd.readthedocs.io/en/stable/getting_started/

Follow the guide to install `argocd` cli and them follow `Port Forwarding` section to port forward to `https://localhost:8080`.  
Then follow Login Using The CLI section to login and update your argo password

> Note that in `argocd login <ARGOCD_SERVER>` <ARGOCD_SERVER> is https://localhost:8080 you port forwarded before it's also your argo UI which you can access in browser


## Apply Argocd App(in argo namespace) to add our wrapper application to Argo
`kubectl apply -f argocd-application.yaml`

> Note `argocd-application.yaml` has https://github.com/MarkiyanPyts/lago-play public repo as source of truth for deployments, you can change it with your own fork of this repo you can control, but mind that if you make this repo private you will have to follow https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/ to give Argo rights to access repo

## 4. After Lago Is Deployed Port Forward it to localhost
```
kubectl get services -n lago-ns

kubectl port-forward svc/<lago-frontend-service-name> 3000:80 -n lago-ns
```

e.g 
`kubectl port-forward svc/lago-wrapper-app-lago-front 3000:80 -n lago-ns`

# 5. Database migrations note
You need to manually run database migration job after argo app is deployed

```
kubectl exec -n lago-ns deployment/lago-wrapper-app-lago-api -- bundle exec rake db:create 2>&1 | tail -5

kubectl exec -n lago-ns deployment/lago-wrapper-app-lago-api -- bundle exec rails db:migrate 2>&1 | tail -10

kubectl exec -n lago-ns deployment/lago-wrapper-app-lago-api -- bundle exec rails roles:seed_predefined 2>&1
```

or 

```
kubectl exec -n lago-ns deployment/lago-wrapper-app-lago-api -- bash scripts/migrate.sh
```

related issue: https://github.com/getlago/lago/issues/708

## Command explanation:

| Part | Explanation |
|------|-------------|
| `kubectl exec` | Run a command inside a running container |
| `-n lago-ns` | In the `lago-ns` namespace |
| `deployment/lago-wrapper-app-lago-api` | Target the API deployment (kubectl picks one of its pods automatically) |
| `--` | Separator — everything after this is the command to run inside the container |
| `bundle exec` | Run the following command using the exact gem versions defined in `Gemfile.lock` |
| `rails db:migrate` | Rails task that runs all pending database migrations |

# 6. Port Forward Lago FE and API
```
kubectl port-forward svc/lago-wrapper-app-lago-api 3001:80 -n lago-ns
kubectl port-forward svc/lago-wrapper-app-lago-front 3000:80 -n lago-ns
```