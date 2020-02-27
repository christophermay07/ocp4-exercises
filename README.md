# ocp4-exercises
Exercises for OCP 4 App Deploy Boot Camp


TODO: Sublinks

TODO: Capture the below in subdirs:



Implement Chained Builds

```
oc import-image jorgemoralespou/s2i-go --confirm
```

```
oc new-build s2i-go~https://github.com/tonykay/ose-chained-builds \
   --context-dir=/go-scratch/hello_world --name=builder
```

```
oc new-build --name=runtime \
   --source-image=builder \
   --source-image-path=/opt/app-root/src/go/src/main/main:. \
   --dockerfile=$'FROM scratch\nCOPY main /main\nEXPOSE 8080\nUSER 1000\nENTRYPOINT ["/main"]'
```

```
oc new-app runtime --name=my-application
oc expose svc/my-application

curl $(oc get route my-application --template='{{ .spec.host }}')
```

