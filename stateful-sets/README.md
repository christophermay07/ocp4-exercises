NOTE: 'oc create service' does NOT accept --selector flag. Got the 90% solution through the following commands, and fixing labels, selectors, and annotations after:

1. Headless service:

```
oc create service clusterip mongodb-internal \
  --clusterip='None' --tcp=27017 \
  --dry-run -o yaml > mongodb-internal-svc.yaml
```

Modify the output above to accomplish the following:

* Set the following annotation under 'metadata':

```
  annotations:
    service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"
```

* Change the generated label to name=mongodb:

```
  labels:
    name: "mongodb"
```

* Change the generated selector to name=mongodb:

```
  selector:
    name: "mongodb"
```


2. Regular ClusterIP service:

```
oc create service clusterip mongodb \
  --tcp=27017 \
  --dry-run -o yaml > mongodb-svc.yaml
```

Modify the output above to accomplish the following:

* Change the generated label to name=mongodb:

```
  labels:
    name: "mongodb"
```

* Change the generated selector to name=mongodb:

```
  selector:
    name: "mongodb"
```

* Change port name to "mongodb":

```
  ports:
    - name: mongodb
```
