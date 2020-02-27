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

Create service for registry (port 5000):
```
oc create service clusterip nexus-registry --tcp=5000
```

Create a route to expose port 5000 using edge termination:
```
oc create route edge nexus-registry --service=nexus-registry --port=5000
```


#######################################

SonarQube

Create the backing database:
```
oc new-app --template=postgresql-persistent \
  --param POSTGRESQL_USER=sonar \
  --param POSTGRESQL_PASSWORD=sonar \
  --param POSTGRESQL_DATABASE=sonar \
  --param VOLUME_CAPACITY=4Gi \
  --labels=app=sonarqube_db
```
TODO: labels came from lab solution doc; do we need this?

Deploy SonarQube:
```
oc new-app --docker-image=quay.io/gpte-devops-automation/sonarqube:7.9.1 -e SONARQUBE_JDBC_USERNAME=sonar -e SONARQUBE_JDBC_PASSWORD=sonar -e SONARQUBE_JDBC_URL=jdbc:postgresql://postgresql/sonar
```

Pause rollout (see prior instructions)

Create a route for SonarQube:
```
oc expose svc sonarqube
```

Create a PVC:
(TODO: Same as above; oc create, but also find oc cli way to do)

Add PVC to dc:
(TODO: Same as above, +need oc command)

Add limits/resource:
```
oc set resources dc sonarqube --limits=cpu=2,memory=3Gi --requests=cpu=1,memory=2G
```

Readiness/Liveness probes:
```
oc set probe dc/sonarqube --readiness --open-tcp=9000
oc set probe dc/sonarqube --liveness --open-tcp=9000
```
TODO: Better probes?


Annotate the pod through the dc (TODO: oc annotate works directly against pod; is there a way to do w/o oc patch dc?)
```
spec:
  template:
    metadata:
      annotations:
        tuned.openshift.io/elasticsearch: "true"
```
TODO: Verify that the above is correct; not 100% sure at time of writing


################################

Jenkins:

oc new-app --template=jenkins-persistent -p MEMORY_LIMIT=2Gi -p VOLUME_CAPACITY=4Gi -p DISABLE_ADMINISTRATIVE_MONITORS=true

Add limits/resource:
```
oc set resources dc jenkins --limits=cpu=2 --requests=cpu=500m,memory=1Gi
```

