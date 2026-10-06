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
helm dependency update lago-wrapper-app/
```

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