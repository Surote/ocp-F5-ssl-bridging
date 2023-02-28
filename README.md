# F5-SSL bridging OCP

Created: February 27, 2023 1:59 PM
Created By: Surote Wongpaiboon
Last Edited Time: February 28, 2023 11:10 AM
Type: Share-public
tag: F5, shared, ssl-bridging

![Untitled](F5-SSL%20bridging%20OCP%20afe31d01458543669165f4190170c8e8/Untitled.png)

`F5 Configuration VS with iRules do a x-forwarded-for`   

```yaml
root@(demo-lb)(cfg-sync Standalone)(Active)(/Common)(tmos)# list ltm rule xff-client
ltm rule xff-client {
**when HTTP_REQUEST {
    HTTP::header insert X-Forwarded-For-F5 [IP::remote_addr]
}
}**
root@(demo-lb)(cfg-sync Standalone)(Active)(/Common)(tmos)# list ltm virtu
Components:
  virtual           virtual-address
root@(demo-lb)(cfg-sync Standalone)(Active)(/Common)(tmos)# list ltm virtual
**ltm virtual VS-apps-l4-pt-443 {
    creation-time 2023-02-26:19:14:38
    destination 192.168.3.235:https
    ip-protocol tcp
    last-modified-time 2023-02-26:22:46:31
    mask 255.255.255.255
    pool pool-router-ipi
    profiles {
        clientssl-secure {
            context clientside
        }
        http { }
        serverssl-secure {
            context serverside
        }
        tcp { }
    }
    rules {
        xff-client
    }
    serverssl-use-sni disabled
    source 192.168.3.196/32
    source-address-translation {
        type automap
    }
    translate-address enabled
    translate-port enabled
    vs-index 2
}**
root@(demo-lb)(cfg-sync Standalone)(Active)(/Common)(tmos)# list ltm pool pool-router-ipi
**ltm pool pool-router-ipi {
    members {
        192.168.3.104:https {
            address 192.168.3.104
            session monitor-enabled
            state up
        }
        192.168.3.115:https {
            address 192.168.3.115
            session monitor-enabled
            state up
        }
    }
    monitor gateway_icmp
}**
```

![Untitled](F5-SSL%20bridging%20OCP%20afe31d01458543669165f4190170c8e8/Untitled%201.png)

![Untitled](F5-SSL%20bridging%20OCP%20afe31d01458543669165f4190170c8e8/Untitled%202.png)

![Untitled](F5-SSL%20bridging%20OCP%20afe31d01458543669165f4190170c8e8/Untitled%203.png)

`pool-attached VS`

![Untitled](F5-SSL%20bridging%20OCP%20afe31d01458543669165f4190170c8e8/Untitled%204.png)

![Untitled](F5-SSL%20bridging%20OCP%20afe31d01458543669165f4190170c8e8/Untitled%205.png)

`iRules`

![Untitled](F5-SSL%20bridging%20OCP%20afe31d01458543669165f4190170c8e8/Untitled%206.png)

`DNS configuration`

point `hello.lb.rhdemo.local` → `192.168.3.235` (VS IP on F5)

![Untitled](F5-SSL%20bridging%20OCP%20afe31d01458543669165f4190170c8e8/Untitled%207.png)

`Test access application`

![Untitled](F5-SSL%20bridging%20OCP%20afe31d01458543669165f4190170c8e8/Untitled%208.png)

![Untitled](F5-SSL%20bridging%20OCP%20afe31d01458543669165f4190170c8e8/Untitled%209.png)

`Application route with SSL terminate at edge`

![Untitled](F5-SSL%20bridging%20OCP%20afe31d01458543669165f4190170c8e8/Untitled%2010.png)

`Application deployment`

`Deployment`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    alpha.image.policy.openshift.io/resolve-names: '*'
    app.openshift.io/route-disabled: "false"
    deployment.kubernetes.io/revision: "1"
    image.openshift.io/triggers: '[{"from":{"kind":"ImageStreamTag","name":"http-https-echo3:26","namespace":"demo"},"fieldPath":"spec.template.spec.containers[?(@.name==\"http-https-echo3\")].image","pause":"false"}]'
    openshift.io/generated-by: OpenShiftWebConsole
  creationTimestamp: "2023-02-27T06:36:27Z"
  generation: 1
  labels:
    app: http-https-echo3
    app.kubernetes.io/component: http-https-echo3
    app.kubernetes.io/instance: http-https-echo3
    app.kubernetes.io/name: http-https-echo3
    app.kubernetes.io/part-of: flask-api-app
    app.openshift.io/runtime-namespace: demo
  name: http-https-echo3
  namespace: demo
  resourceVersion: "39768089"
  uid: 82a3ed2d-fd90-4ba5-8122-e34f37cb5230
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: http-https-echo3
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      annotations:
        openshift.io/generated-by: OpenShiftWebConsole
      creationTimestamp: null
      labels:
        app: http-https-echo3
        deployment: http-https-echo3
    spec:
      containers:
      - image: image-registry.openshift-image-registry.svc:5000/demo/http-https-echo3@sha256:9d6f63844afa426ee1e606b1268b87bde6092030f213301fca2a6b06bd5b063f
        imagePullPolicy: IfNotPresent
        name: http-https-echo3
        ports:
        - containerPort: 8080
          protocol: TCP
        - containerPort: 8443
          protocol: TCP
        resources:
          limits:
            cpu: 250m
            memory: 25Mi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

`svc`

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    openshift.io/generated-by: OpenShiftWebConsole
  creationTimestamp: "2023-02-27T06:36:27Z"
  labels:
    app: http-https-echo3
    app.kubernetes.io/component: http-https-echo3
    app.kubernetes.io/instance: http-https-echo3
    app.kubernetes.io/name: http-https-echo3
    app.kubernetes.io/part-of: flask-api-app
    app.openshift.io/runtime-version: "26"
  name: http-https-echo3
  namespace: demo
  resourceVersion: "39767803"
  uid: 1f4416e5-10ed-45a2-a6c6-436dd1a8143c
spec:
  clusterIP: 172.30.224.95
  clusterIPs:
  - 172.30.224.95
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: 8080-tcp
    port: 8080
    protocol: TCP
    targetPort: 8080
  - name: 8443-tcp
    port: 8443
    protocol: TCP
    targetPort: 8443
  selector:
    app: http-https-echo3
    deployment: http-https-echo3
  sessionAffinity: None
  type: ClusterIP
```

`route`

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  creationTimestamp: "2023-02-27T03:20:03Z"
  labels:
    app: http-https-echo3
    app.kubernetes.io/component: http-https-echo3
    app.kubernetes.io/instance: http-https-echo3
    app.kubernetes.io/name: http-https-echo3
    app.kubernetes.io/part-of: flask-api-app
    app.openshift.io/runtime-version: "26"
  name: test
  namespace: demo
  resourceVersion: "39855257"
  uid: a6002b50-005a-428b-b7a2-44d458a97f87
spec:
  host: hello.lb.rhdemo.local
  port:
    targetPort: 8080-tcp
  tls:
    insecureEdgeTerminationPolicy: None
    termination: edge
  to:
    kind: Service
    name: http-https-echo3
    weight: 100
  wildcardPolicy: None
```