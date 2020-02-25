Nexus:

```
oc new-app sonatype/nexus3:latest
```

About to do a bunch of updates, pause rollout:

```
oc rollout pause dc nexus3
```

Edit the following in the nexus3 dc:

* spec.strategy.type: Recreate
(NOTE: Can I leave the rollingParams, or must I remove?)

```
spec:
  containers:
  - name: ...
    resources:
      limits:
        cpu: "2"
        memory: "2Gi"
      requests:
        cpu: "500m"
        memory: "1Gi"
```
TODO: Can I turn the above into oc command?

Add liveness / readiness probes:

```
oc set probe dc/nexus3 --readiness --open-tcp=8080
oc set probe dc/nexus3 --liveness --open-tcp=8080
```
TODO: The above isn't great :(

#######################################3

SonarQube

```
oc new-app --template=postgresql-persistent \
  --param POSTGRESQL_USER=sonar \
  --param POSTGRESQL_PASSWORD=sonar \
  --param POSTGRESQL_DATABASE=sonar \
  --param VOLUME_CAPACITY=4Gi \
  --labels=app=sonarqube_db
```
TODO: labels came from solution; do we need this?
