# KUBERNETES

## TLS
Use CFSSL to generate certificates

More about [CFSSL here]("https://github.com/cloudflare/cfssl")

```
cd kubernetes\admissioncontrollers\introduction

docker run -it --rm -v ${PWD}:/work -w /work debian bash

apt-get update && apt-get install -y curl &&
curl -L https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssl_1.5.0_linux_amd64 -o /usr/local/bin/cfssl && \
curl -L https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssljson_1.5.0_linux_amd64 -o /usr/local/bin/cfssljson && \
chmod +x /usr/local/bin/cfssl && \
chmod +x /usr/local/bin/cfssljson

#generate ca in /tmp
cfssl gencert -initca ./tls/ca-csr.json | cfssljson -bare /tmp/ca

#generate certificate in /tmp
cfssl gencert \
  -ca=/tmp/ca.pem \
  -ca-key=/tmp/ca-key.pem \
  -config=./tls/ca-config.json \
  -hostname="example-webhook,example-webhook.default.svc.cluster.local,example-webhook.default.svc,localhost,127.0.0.1" \
  -profile=default \
  ./tls/ca-csr.json | cfssljson -bare /tmp/example-webhook

#make a secret
cat <<EOF > ./tls/example-webhook-tls.yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-webhook-tls
type: Opaque
data:
  tls.crt: $(cat /tmp/example-webhook.pem | base64 | tr -d '\n')
  tls.key: $(cat /tmp/example-webhook-key.pem | base64 | tr -d '\n') 
EOF
```
For this lab, the certificates already created in path ./tls
## JENKINS
In order to run Jenkins agent, go to below steps:
```
k edit sa jenkins -n devsecops -o yaml
```
Embed field: automountServiceAccountToken: false

Result should be like:
```
apiVersion: v1
automountServiceAccountToken: false
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: devsecops
```

## KYVERNO
Should add this following annotation to values.yaml `kyverno.crds.annotations` to avoid CRD failure:
```
argocd.argoproj.io/sync-options: Replace=true
```

The secret should be created first. Ex: create secret for authenticating to docker.io
```
DOCKER_REGISTRY_SERVER=docker.io
DOCKER_USER=Type your dockerhub username, same as when you `docker login`
DOCKER_EMAIL=Type your dockerhub email, same as when you `docker login`
DOCKER_PASSWORD=Type your dockerhub pw, same as when you `docker login`

kubectl create secret docker-registry phidp-image-cred \
  --docker-server=$DOCKER_REGISTRY_SERVER \
  --docker-username=$DOCKER_USER \
  --docker-password=$DOCKER_PASSWORD \
  --docker-email=$DOCKER_EMAIL
```