# K8SLAB
K8S Lab Test

## Install Grafana / Prometheus / Alert Manager /DingDing Hook

### SetUp 


### Install Grafana / Prometheus / Alert Manager


## DingDing Hook
### 在`Kubernetes`集群中运行
第一步建议将钉钉机器人TOKEN创建成`Secret`资源对象：
```shell
$ kubectl create secret generic dingtalk-secret --from-literal=token=<钉钉群聊的机器人TOKEN> --from-literal=secret=<钉钉群聊机器人的SECRET> -n kube-ops
secret "dingtalk-secret" created
```

然后定义`Deployment`和`Service`资源对象：(dingtalk-hook.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dingtalk-hook
  namespace: kube-ops
spec:
  selector:
    matchLabels:
      app: dingtalk-hook
  template:
    metadata:
      labels:
        app: dingtalk-hook
    spec:
      containers:
      - name: dingtalk-hook
        image: cnych/alertmanager-dingtalk-hook:v0.3.6
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 5000
          name: http
        env:
        - name: PROME_URL
          value: prometheus.local
        - name: LOG_LEVEL
          value: debug
        - name: ROBOT_TOKEN
          valueFrom:
            secretKeyRef:
              name: dingtalk-secret
              key: token
        - name: ROBOT_SECRET
          valueFrom:
            secretKeyRef:
              name: dingtalk-secret
              key: secret
        resources:
          requests:
            cpu: 50m
            memory: 100Mi
          limits:
            cpu: 50m
            memory: 100Mi

---
apiVersion: v1
kind: Service
metadata:
  name: dingtalk-hook
  namespace: kube-ops
spec:
  selector:
    app: dingtalk-hook
  ports:
  - name: hook
    port: 5000
    targetPort: http
```

直接创建上面的资源对象即可：
```shell
$ kubectl create -f dingtalk-hook.yaml
deployment.apps "dingtalk-hook" created
service "dingtalk-hook" created
$ kubectl get pods -n kube-ops
NAME                            READY     STATUS      RESTARTS   AGE
dingtalk-hook-c4fcd8cd6-6r2b6   1/1       Running     0          45m
......
```

最后在`AlertManager`中 webhook 地址直接通过 DNS 形式访问即可：
```yaml
receivers:
- name: 'webhook'
  webhook_configs:
  - url: 'http://dingtalk-hook.kube-ops.svc.cluster.local:5000'
    send_resolved: true
```

## 参考文档
* [钉钉自定义机器人文档](https://open-doc.dingtalk.com/microapp/serverapi2/qf2nxq)
* [AlertManager 的使用](https://www.qikqiak.com/k8s-book/docs/57.AlertManager%E7%9A%84%E4%BD%BF%E7%94%A8.html)
 
