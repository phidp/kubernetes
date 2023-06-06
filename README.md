# kubernetes
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

