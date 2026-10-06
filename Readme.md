# Local Namespace Is Created By 
```
kubectl apply -f namespace.yaml
```

# Parent Chart Was Created With
`https://helm.sh/docs/helm/helm_create/`

```
helm create lago-wrapper-app
```

# Install Lago Dependency
```
helm repo update
helm dependency update lago-wrapper-app
```

# Other Dependency Charts App Contains

https://hub.docker.com/layers/getlago/postgres-partman/latest
https://artifacthub.io/packages/helm/bitnami/redis?modal=install

> Note partman extension is required in Postgres, https://artifacthub.io/packages/helm/bitnami/postgresql/18.12.4?modal=install does not have it out of the box so I could not use it for local testing


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

# Install AgroCD
https://argo-cd.readthedocs.io/en/stable/getting_started/

## Port Forward ArgoCD to run locally(as per above link)
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Also login using CLI as described in that link and access ArgoCD UI to see apps


## Apply Argocd App(in argo namespace)
`kubectl apply -f argocd-application.yaml`

## After Lago Is Deployed Port Forward it to localhost
```
kubectl get services -n lago-ns

kubectl port-forward svc/<lago-frontend-service-name> 3000:80 -n lago-ns
```

e.g 
`kubectl port-forward svc/lago-wrapper-app-lago-front 3000:80 -n lago-ns`

# Database migrations note
Currently migration flag is true for lago, in prod it should be false and migration should be something handled manually

```
  migrate:
    enabled: true
```
