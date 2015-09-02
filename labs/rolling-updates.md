# Rolling Updates

* Use multiple replication controllers with a single service
* Deploy a canary application for testing

## Send the Canary

```
kubectl run inspector-canary \
  --labels="app=inspector,track=canary" \
  --replicas=1 \
  --image=b.gcr.io/kuar/inspector:2.0.0 \
  --overrides='{
  "apiVersion": "v1",
  "spec": {
    "template": {
      "spec": {
        "containers": [
          {
            "args": [
              "-insecure-host=0.0.0.0",
              "-insecure-port=80"
            ],
            "image": "b.gcr.io/kuar/inspector:2.0.0",
            "name": "inspector-canary"
          }
        ]
      }
    }
  }
}'
```

### Validation

Try hitting the service port on any of the node instances.

#### laptop

```
while true; do curl -s http://nginx | \
  grep -o -e 'Version: Inspector [0-9].[0-9].[0-9]'; sleep 1; done
```

Did you find the canary?

## Rolling Update

Open three terminals

### Terminal 1

#### laptop

```
kubectl get pods --watch
```

### Terminal 2

#### laptop

```
while true; do curl -s http://nginx | \
  grep -o -e 'Version: Inspector [0-9].[0-9].[0-9]'; sleep 1; done
```

### Terminal 3

#### laptop

```
kubectl rolling-update inspector --update-period=3s --image=b.gcr.io/kuar/inspector:2.0.0
```
