# env_var_demo

Creating a production environment in Killercoda playground
--------------
## Create Prod Environment configurations

### define production namespace
`kubectl create namespace production`

### configure env-specific configmaps

`kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --namespace=production`

### define sensitive info as secrets

`kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=prodpassword123 \
  --namespace=production`

### view all secrets per enfironment 

`kubectl get secrets -n production`

### to view metadata of secrets

`kubectl describe secret app-secret -n production`

### To decode secrets

`kubectl get secret app-secret -n production -o jsonpath="{.data.DB_PASSWORD}" | base64 --decode`

----------------
## Deploying deployments to production environment

### create a generic deployment YAML template. 
Save it as app-deployment-template.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-app
      namespace: PLACEHOLDER_NAMESPACE
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx-app
      template:
        metadata:
          labels:
            app: nginx-app
        spec:
          containers:
          - name: nginx
            image: nginx
            env:
            - name: APP_ENV
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: APP_ENV
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secret
                  key: DB_PASSWORD
              
### Replace generic yaml template and deploy

`sed -e 's/PLACEHOLDER_NAMESPACE/production/' \
    -e 's/replicas: .*$/replicas: 3/' app-deployment-template.yaml > production-deployment.yaml`

`kubectl apply -f production-deployment.yaml`

### expose the production environment

`kubectl expose deployment nginx-app \
  --type=NodePort \
  --name=nginx-service \
  --port=80 \
  --target-port=80 \
  --namespace=production`

### verify deployment and access

`kubectl get pods -n production
kubectl get service -n production`

`kubectl exec -it <pod-name> -n production -- /bin/bash`

`echo $APP_ENV
echo $DB_PASSWORD`
