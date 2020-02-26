Nexus:

```
oc new-app sonatype/nexus:latest
```

About to do a bunch of updates, pause rollout:

```
oc rollout pause dc nexus
```

Edit the following in the nexus dc:

* spec.strategy.type: Recreate
(NOTE: Can I leave the rollingParams, or must I remove?)

```
oc set resources dc nexus --limits=cpu=2,memory=2Gi --requests=cpu=500m,memory=1G
```

Add liveness / readiness probes:

```
oc set probe dc/nexus --readiness --open-tcp=8080
oc set probe dc/nexus --liveness --open-tcp=8080
```
TODO: The above isn't great :(

Apply PVC (in repo):
```
oc create -f nexus-pvc.yaml
```

Set volume to use PVC:
```
      volumes:
      - name: nexus-volume-1
        persistentVolumeClaim:
          claimName: nexus-data
```
TODO: Update to use oc set volume command!

Resume rollout:

```
oc rollout resume dc nexus
```

#######################################

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
