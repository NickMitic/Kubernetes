# Setting up the app and db with persistent storage using K8s

## Set-up and Diagram

## The YAML files

### Deployments

These create replica sets, which in turn create pods. Pods contain the containers which actually run the app.

#### App Deployment
```yaml
# YAML is case sensitive
# use spaces not a tab
apiVersion: apps/v1  # specify api to use for deployment
kind: Deployment  # kind of service/object you want to create
metadata:
  name: app-deployment # name the deployment
spec:
  selector:
    matchLabels:
      app: sparta-app  # look for this label/tag to match with k8 service
  # create a replica set of this with instances/pods
  replicas: 3
  template:
    metadata:
      labels:
        app: sparta-app
    spec:
      containers:
      - name: sparta-app
        image: nickmitic/test1:v1 # this image contains runs the sparta app - does not seed
        ports:
        - containerPort: 80 # Opens port 80 (see first bullet below)
        env:
        - name: DB_HOST
          value: mongodb://db-service:27017/posts # connects to the *db-service*, IP is found via the name given in the db-service.yml
        command: [ "sh", "-c", "node seeds/seed.js && node app.js" ] # seeds and runs the app (see second bullet below)
```

* Opening port 80 is here isn't quite right, as our app listens on port 3000. However, this doesn't actually matter as a port **not** being listed here **does not** mean that it is inaccessable within the network. See port section [here](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#ports).
* This isn't the best way to seed. If the app pods go down then a restart will re-seed the db, which we don't really want to do. If set-up in this way this command subsequently needs to be commented, out in order to preserve the data. Clearly this isn't a good approach. It would be far better to leave this out entirely, then seed the db manually. Or, even better, put in some logic to only seed if the posts page is empty.

#### DB Deployment
```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:
  name: db-deployment 
spec:
  selector:
    matchLabels:
      app: sparta-db  
  replicas: 1 # just 1 pod for the db
  template:
    metadata:
      labels:
        app: sparta-db
    spec:
      containers:
      - name: sparta-db
        image: bitnami/mongodb:7.0.6 # correct mongodb image
        ports:
        - containerPort: 27017 # default port mongodb listens on 
        volumeMounts:
          - name: mongodb-storage # must match name in volumes
            mountPath: /bitnami/mongodb # path for this image's storage (will be different for other images - check their docs)
      volumes: # add a volume to this deployment for persistent data storage
      - name: mongodb-storage
        persistentVolumeClaim: # the PVC that will request the storage
          claimName: mongodb-pvc # must match name in pvc.yml

```

### Services

These are how we will connect different parts of the system, and connect it to localhost/the outside world

#### App Service
```yaml
---

apiVersion: v1
kind: Service
metadata:
  name: app-svc
  namespace: default
spec:
  ports:
  - nodePort: 30001 # range is 30000-32768 - this is the port on localhost that the cluster will listen on
    port: 80 - port on service
    targetPort: 3000 - port on pod - the app in this case
  selector:
    app: sparta-app  # this label connects this service to the app deployment
  type: NodePort  # also use LoadBalancer - for local use cluster IP
```

#### DB Service
```yaml
---

apiVersion: v1
kind: Service
metadata:
  name: db-service
spec:
  selector:
    app: sparta-db # connects to db
  ports:
  - port: 27017 # default mongodb port - this could actually be anything, as long as it matches the port in the app ENV
    targetPort: 27017 # default mongodb port
```
* Note that we **did not** make this service a nodeport as there is no need to expose this to localhost/the internet
### PVC
This makes a request to the cluster for storage, where our volume will go.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc # must match claimname in db-deploy.yml
spec:
  accessModes:
    - ReadWriteOnce # usual setting - read/write by a single node
  resources:
    requests:
      storage: 400Mi - estimated from local Docker volumes that did the same job
```

## How to Initialise
1. In the directory containing these YAMLs use ```kubectl create -f pvc.yml``` firstly
   * K8s will not create the db until the PVC is created
   * As we need the db before the app we should create the PVC first
2. ```kubectl create -f db-deploy.yml```
3. ```kubectl create -f db-service.yml```
   * This also needs to be in place before the app - so seeding can take place
4. ```kubectl create -f app-deploy.yml```
5. ```kubectl create -f app-service.yml```
   * App is now available on localhost:30001 and localhost:30001/posts is populated

## Testing the PVC
To make sure the data persists we can delete the db, then restart it. Without the PVC and PV this would result in an empty posts page.
1. Check the /posts page is populated
2. ```kubectl delete -f db-deploy.yml``` - /posts should be completely down now
3. ```kubectl create -f db-deploy.yml```
4. Check /posts again - should be populated with the same posts as in step 1 without any seeding being needed