# Grafana Dashboards

With introduction dashboard provisioning in [grafana v5](http://docs.grafana.org/guides/whats-new-in-v5/#dashboards), you can have dashboards in a git repository for version control and you can clone it during startup.

## Kubernetes Use

One time clone with gitRepo volume

```
apiVersion: v1
kind: Pod
metadata:
  name: grafana
spec:
  containers:
  - image: grafana/grafana:5.0.3
    name: grafana
    env:
    - name: GF_PATHS_PROVISIONING
      value: "/etc/grafana/provisioning"
    volumeMounts:
    - name: dashboard-config
      mountPath: "/etc/grafana/provisioning/dashboards"
    - mountPath: /var/lib/grafana/dashboards
      name: grafana-dashboard-volume
  volumes:
  - name: dashboard-config
    configMap:
      name: grafana-dashboard-config
  - name: grafana-dashboard-volume
    gitRepo:
      repository: "git@github.com:oba11/grafana-dashboards.git"
      revision: "22f1d8406d464b0c0874075539c1f2e96c253775"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboard-config
  namespace: kube-system
data:
  default.yaml: |
    apiVersion: 1
    providers:
    - name: 'default'
      orgId: 1
      folder: ''
      type: file
      disableDeletion: false
      editable: true
      options:
        path: /var/lib/grafana/dashboards
```

Git-Sync Sidecar

```
apiVersion: v1
kind: Pod
metadata:
  name: grafana
spec:
  containers:
  - image: grafana/grafana:5.0.3
    name: grafana
    env:
    - name: GF_PATHS_PROVISIONING
      value: "/etc/grafana/provisioning"
    volumeMounts:
    - name: dashboard-config
      mountPath: "/etc/grafana/provisioning/dashboards"
    - mountPath: /var/lib/grafana/dashboards
      name: grafana-dashboard-volume
  - image: k8s.gcr.io/git-sync:v2.0.6
    name: git-sync
    args:
    - "-repo=https://github.com/oba11/grafana-dashboards.git"
    - "-branch=master"
    - "-root=/tmp/git"
    - "-wait=30"
    volumeMounts:
    - mountPath: /git
      name: grafana-dashboard-volume
  volumes:
  - name: dashboard-config
    configMap:
      name: grafana-dashboard-config
  - name: grafana-dashboard-volume
    emptyDir: {}
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboard-config
  namespace: kube-system
data:
  default.yaml: |
    apiVersion: 1
    providers:
    - name: 'default'
      orgId: 1
      folder: ''
      type: file
      disableDeletion: false
      editable: true
      options:
        path: /var/lib/grafana/dashboards
```



## Special Thanks for Awesome Dashboards

* [prometheus-operator dashboards](https://github.com/coreos/prometheus-operator/blob/master/contrib/kube-prometheus/manifests/grafana/grafana-dashboard-definitions.yaml) by CoreOS
* [Kubernetes cluster monitoring](https://grafana.com/dashboards/1621) by cjdc
