# source env vars
```
source .secretenv
source .env1
```

# create management cluster
```
kind create cluster
```

# install CAPI & CAPA & helm addon provider
```
clusterctl init --infrastructure aws --addon helm
```


# install argo
```
k apply -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
echo "k port-forward svc/argocd-server 8080:80" | pbcopy

# user: admin
# pass:
k get secrets argocd-initial-admin-secret -o jsonpath={.data.password} | base64 --decode | pbcopy && open http://localhost:8080
```

# create cluster
```
k apply -f argo-app.yaml
```


# show dns records
```
aws route53 list-resource-record-sets --hosted-zone-id=Z00685903NFM1HS4Z3B6N | jq '.ResourceRecordSets'
```

# force reconciliation
```
k patch applications clusters --type=merge -p '{"operation":{"sync":{"syncStrategy":{"apply":{"force":true}}}}}'
```
