# Horizontal Pod Autoscaler

Horizontal Pod Autoscaler automatically scales the number of Pods in a replication controller, deployment, replica set or stateful set based on observed CPU utilization.

To test this first you have to create a simple deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  replicas: 1
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
```

Then -if you're using minikube-, enable the metrics-addon using the command:
 
```sh
[younesbe@node-1 ~]$ minikube addons enable metrics-server
```
Then create the HPA using the command:

```sh
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

To check the current status of autoscaler, run:

```sh
kubectl get hpa
```
Output:

```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache/scale   0% / 50%  1         10        1          18s
```

Now we're all set if we generate a load to this pod that it will pass the 50% cpu usage, the HPA will automatically scale out the deployment so it can handle the load.

Simultaneously, if the load decreased below 50% the HPA will scale in so only one pod would remain up and running.

![HPA](/images/hpa.jpg)

Hope you found it useful!