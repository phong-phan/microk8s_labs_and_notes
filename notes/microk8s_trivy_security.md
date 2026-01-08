# MicroK8s Trivy Security Scanning

The Trivy community addon for MicroK8s comprises the Trivy Operator and the Trivy CLI, both of which can be used to perform security scans on your cluster.

---

## Enabling Trivy Addon

Enable the community and Trivy addons:

```bash
microk8s enable community
microk8s enable trivy
```

Once the operator has been installed, you can verify that it is running by inspecting the pods:

```bash
microk8s kubectl get all -A
```

Or, specifically for the Trivy namespace:

```bash
microk8s kubectl get all -n trivy-system
```

**Sample output:**

```
NAME                                            READY   STATUS      RESTARTS   AGE
pod/node-collector-8c4cd6d69-7wvsm              0/1     Completed   0          13m
pod/scan-vulnerabilityreport-5957c56f9d-xwwmg   0/1     Completed   0          13m
pod/scan-vulnerabilityreport-59856d9fd4-ggfwq   0/1     Completed   0          13m
pod/scan-vulnerabilityreport-599f648876-khphn   0/1     Completed   0          13m
pod/scan-vulnerabilityreport-5ffd4c8987-p9szr   0/1     Completed   0          13m
pod/scan-vulnerabilityreport-6979d4fc9-ffslr    0/1     Completed   0          13m
pod/scan-vulnerabilityreport-6cb94ff55c-m6szj   0/3     Completed   0          13m
pod/scan-vulnerabilityreport-74f67994cc-nhrwf   0/2     Completed   0          13m
pod/scan-vulnerabilityreport-75d8fc986c-f4t7r   0/1     Completed   0          13m
pod/scan-vulnerabilityreport-766bd8947d-bgtt8   0/1     Completed   0          13m
pod/scan-vulnerabilityreport-795469cb9-tvhdv    0/3     Completed   0          13m
pod/trivy-operator-75d59b6b69-sd9kf             1/1     Running     0          14m

NAME                     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/trivy-operator   ClusterIP   None         <none>        80/TCP    14m

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/trivy-operator   1/1     1            1           14m

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/trivy-operator-75d59b6b69   1         1         1       14m

NAME                                            STATUS     COMPLETIONS   DURATION   AGE
job.batch/node-collector-8c4cd6d69              Complete   1/1           14s        13m
job.batch/scan-vulnerabilityreport-5957c56f9d   Complete   1/1           99s        13m
job.batch/scan-vulnerabilityreport-59856d9fd4   Complete   1/1           3m17s      13m
job.batch/scan-vulnerabilityreport-599f648876   Complete   1/1           112s       13m
job.batch/scan-vulnerabilityreport-5ffd4c8987   Complete   1/1           118s       13m
job.batch/scan-vulnerabilityreport-6979d4fc9    Complete   1/1           99s        13m
job.batch/scan-vulnerabilityreport-6cb94ff55c   Complete   1/1           2m36s      13m
job.batch/scan-vulnerabilityreport-74f67994cc   Complete   1/1           2m36s      13m
job.batch/scan-vulnerabilityreport-75d8fc986c   Complete   1/1           116s       13m
job.batch/scan-vulnerabilityreport-766bd8947d   Complete   1/1           2m20s      13m
job.batch/scan-vulnerabilityreport-795469cb9    Complete   1/1           3m16s      13m

---

## Using Trivy Operator

The Trivy Operator runs Trivy security tools and incorporates their output into Kubernetes CRDs (Custom Resource Definitions).  
From there, security reports are accessible through the Kubernetes API, making it easy for users to find and view the risks that relate to different resources in a Kubernetes-native way.

---

## Scanning Resources

By default, Trivy is configured to scan resources in all namespaces.

To test this, create a deployment:

```bash
microk8s kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```

---

## Viewing Vulnerability Reports

Vulnerability reports are available within the cluster.  
A simple status can be found by running:

```bash
microk8s kubectl get vulnerabilityreports --all-namespaces -o wide
```

To retrieve more details:

```bash
microk8s kubectl describe vulnerabilityreports --all-namespaces
```

---

## Running a Configuration Audit

Configuration audits are also exposed through the Kubernetes API.  
Run the following command:

```bash
microk8s kubectl get configauditreports --all-namespaces -o wide
```

**Sample output:**

```
NAMESPACE            NAME                                                           SCANNER   AGE   CRITICAL   HIGH   MEDIUM   LOW
container-registry   replicaset-registry-579865c76c                                 Trivy     23h   0          0      2        10
container-registry   service-registry                                               Trivy     23h   0          0      0        0
default              replicaset-kubernetes-bootcamp-9bc58d867                       Trivy     22h   0          0      2        11
default              replicaset-nginx-deployment-789494bcc4                         Trivy     23h   0          0      2        11
default              service-kubernetes                                             Trivy     23h   0          0      0        1
default              service-nginx-test                                             Trivy     23h   0          0      0        1
default              service-webserver                                              Trivy     23h   0          0      0        1
demo                 replicaset-demo-app-5996977bd4                                 Trivy     23h   0          0      2        6
demo                 replicaset-microbot-2-85d8655fb9                               Trivy     23h   0          0      2        10
demo                 service-demo-service                                           Trivy     23h   0          0      0        0
demo                 service-microbot-2                                             Trivy     23h   0          0      0        0
ingress              daemonset-nginx-ingress-microk8s-controller                    Trivy     23h   0          1      3        8
kube-system          daemonset-calico-node                                          Trivy     23h   0          4      8        25
kube-system          replicaset-calico-kube-controllers-5947598c79                  Trivy     23h   0          0      3        10
kube-system          replicaset-coredns-79b94494c7                                  Trivy     23h   0          0      3        5
kube-system          replicaset-dashboard-metrics-scraper-5bd45c9dd6                Trivy     23h   0          0      2        8
kube-system          replicaset-hostpath-provisioner-c778b7559                      Trivy     23h   0          0      4        10
kube-system          replicaset-kubernetes-dashboard-57bc5f89fb                     Trivy     23h   0          0      2        8
kube-system          replicaset-metrics-server-7dbd8b5cc9                           Trivy     23h   0          0      2        7
kube-system          service-dashboard-metrics-scraper                              Trivy     23h   0          0      1        0
kube-system          service-kube-dns                                               Trivy     23h   0          0      1        0
kube-system          service-kube-prom-stack-kube-prome-coredns                     Trivy     23h   0          0      1        0
kube-system          service-kube-prom-stack-kube-prome-kube-controller-manager     Trivy     23h   0          0      1        0
kube-system          service-kube-prom-stack-kube-prome-kube-etcd                   Trivy     23h   0          0      1        0
kube-system          service-kube-prom-stack-kube-prome-kube-proxy                  Trivy     23h   0          0      1        0
kube-system          service-kube-prom-stack-kube-prome-kube-scheduler              Trivy     23h   0          0      1        0
kube-system          service-kube-prom-stack-kube-prome-kubelet                     Trivy     23h   0          0      1        0
kube-system          service-kubernetes-dashboard                                   Trivy     23h   0          0      1        0
kube-system          service-metrics-server                                         Trivy     23h   0          0      1        0
metallb-system       daemonset-speaker                                              Trivy     23h   0          1      2        8
metallb-system       replicaset-controller-7ffc454778                               Trivy     23h   0          0      0        8
metallb-system       service-webhook-service                                        Trivy     23h   0          0      0        0
observability        daemonset-kube-prom-stack-prometheus-node-exporter             Trivy     23h   0          2      2        10
observability        daemonset-loki-promtail                                        Trivy     23h   0          0      2        8
observability        replicaset-kube-prom-stack-grafana-c5b5bf7d                    Trivy     23h   0          0      6        26
observability        replicaset-kube-prom-stack-kube-prome-operator-95f875dd6       Trivy     23h   0          0      0        9
observability        replicaset-kube-prom-stack-kube-state-metrics-86b8bcd658       Trivy     23h   0          0      2        10
observability        service-alertmanager-operated                                  Trivy     23h   0          0      0        0
observability        service-kube-prom-stack-grafana                                Trivy     23h   0          0      0        0
observability        service-kube-prom-stack-kube-prome-alertmanager                Trivy     23h   0          0      0        0
observability        service-kube-prom-stack-kube-prome-operator                    Trivy     23h   0          0      0        0
observability        service-kube-prom-stack-kube-prome-prometheus                  Trivy     23h   0          0      0        0
observability        service-kube-prom-stack-kube-state-metrics                     Trivy     23h   0          0      0        0
observability        service-kube-prom-stack-prometheus-node-exporter               Trivy     23h   0          0      0        0
observability        service-loki                                                   Trivy     23h   0          0      0        0
observability        service-loki-headless                                          Trivy     23h   0          0      0        0
observability        service-loki-memberlist                                        Trivy     23h   0          0      0        0
observability        service-prometheus-operated                                    Trivy     23h   0          0      0        0
observability        service-tempo                                                  Trivy     23h   0          0      0        0
observability        statefulset-757589978                                          Trivy     23h   0          0      0        8
observability        statefulset-loki                                               Trivy     23h   0          0      1        9
observability        statefulset-prometheus-kube-prom-stack-kube-prome-prometheus   Trivy     23h   0          0      0        11
observability        statefulset-tempo                                              Trivy     23h   0          0      4        18
trivy-system         replicaset-trivy-operator-75d59b6b69                           Trivy     23h   0          0      1        7
trivy-system         service-trivy-operator                                         Trivy     23h   0          0      0        0
```




