### How to configure custom params for the kube-scheduler when using kubeadm.

If you would like to use [kind](https://sigs.k8s.io/kind) to try this out you can use the [config](./config) file provided.

The config assumes that `scheduler-config.yaml` is located in `/tmp/`

Make sure you copy it there first.


example:

```
$ cp scheduler-config.yaml /tmp/
$ kind create cluster --config config
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.14.2) 🖼
 ✓ Preparing nodes 📦
 ✓ Creating kubeadm config 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing StorageClass 💾
Cluster creation complete. You can now use the cluster with:

export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
kubectl cluster-info

```

The `config` file describes a patch that will populate the file on the control-plane node and configure the scheduler to use it.

When using the config you need to provide only the config not any cli args to scheduler.

Here is an example configuration that is modified from the output of `kube-scheduler --write-config-to scheduler-config.yaml`

I needed to add the path to the `kubeconfig` for this configuration to work correctly.

```
algorithmSource:
  provider: DefaultProvider
apiVersion: kubescheduler.config.k8s.io/v1alpha1
bindTimeoutSeconds: 600
clientConnection:
  acceptContentTypes: ""
  burst: 100
  contentType: application/vnd.kubernetes.protobuf
  kubeconfig: "/etc/kubernetes/scheduler.conf"
  qps: 50
disablePreemption: false
enableContentionProfiling: false
enableProfiling: false
failureDomains: kubernetes.io/hostname,failure-domain.beta.kubernetes.io/zone,failure-domain.beta.kubernetes.io/region
hardPodAffinitySymmetricWeight: 1
healthzBindAddress: 0.0.0.0:10251
kind: KubeSchedulerConfiguration
leaderElection:
  leaderElect: true
  leaseDuration: 15s
  lockObjectName: kube-scheduler
  lockObjectNamespace: kube-system
  renewDeadline: 10s
  resourceLock: endpoints
  retryPeriod: 2s
metricsBindAddress: 0.0.0.0:10251
percentageOfNodesToScore: 0
schedulerName: default-scheduler
```


The important bits are:
```
scheduler:
  extraArgs:
    config: /etc/kubernetes/scheduler-config.yaml
  extraVolumes:
  - hostPath: /etc/kubernetes/scheduler-config.yaml
    mountPath: /etc/kubernetes/scheduler-config.yaml
    name: scheduler-config
    pathType: File
    readOnly: true

```

This configures the pod that the scheduler will use to mount a new file at `/etc/kubernetes/scheduler-config.yaml`
Then configures kube-scheduler to use it.

The complete kubeadm.conf is provided as an example.

Thanks!

Hit me up on slack.k8s.io or twitter I am @mauilion :)
