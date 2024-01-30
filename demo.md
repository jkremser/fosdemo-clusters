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


# check the app
```
curl http://roundrobin.demo.k8s.kremser.dev
```

# force reconciliation
```
k patch applications clusters --type=merge -p '{"operation":{"sync":{"syncStrategy":{"apply":{"force":true}}}}}'
```
k8gb-us-east-2-demo.k8s.kremser.dev

# domain cleanup
```
_DO="demo.k8s.kremser.dev."
_HZ="Z00685903NFM1HS4Z3B6N"
for r in us-east-2 eu-west-2; do
  for rec in k8gb-${r}-ns-${_DO} \
          k8gb-${r}-${_DO} \
          k8gb-${r}-gslb-ns-${r}-${_DO} \
          k8gb-${r}-a-gslb-ns-${r}-${_DO} \
          gslb-ns-${r}-${_DO} \
          ${_DO} ;do
    RS=$(aws route53 --hosted-zone-id=${_HZ} list-resource-record-sets | jq '.ResourceRecordSets[] | select(.Name=="'${rec}'")' 2> /dev/null)
    [ -z "${RS}" ] && echo "${rec} is already gone, skipping.." && continue
    cat <<DNS > dns.json
{
  "Comment": "resetting dns records",
  "Changes": [
    {
      "Action": "DELETE",
      "ResourceRecordSet": ${RS}
    }
  ]
}
DNS
    aws route53 --hosted-zone-id=Z00685903NFM1HS4Z3B6N change-resource-record-sets --change-batch file://dns.json
    echo "deleted ${rec} .."
    rm -rf dns.json
  done
done
```