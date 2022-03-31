# Red Hat Hypershift on BareMetal

Hub cluster is compact cluster 3 control nodes

Has ODF installed although not required but some kind of storage will be needed for AI
Has community versions Infrastructure Operator for Red Hat OpenShift (Assisted Installer) and Hive installed

Configure AI using the following yaml


~~~bash
cat << EOF > ~/agentserviceconfig.yaml
apiVersion: agent-install.openshift.io/v1beta1
kind: AgentServiceConfig
metadata:
  name: agent
spec:
  databaseStorage:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 10Gi
  filesystemStorage:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 100Gi
  imageStorage:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 10Gi
  mustGatherImages:
    - name: cnv
      openshiftVersion: '4.8'
      url: >-
        registry.redhat.io/container-native-virtualization/cnv-must-gather-rhel8:v2.6.5
    - name: ocs
      openshiftVersion: '4.8'
      url: registry.redhat.io/ocs4/ocs-must-gather-rhel8
    - name: lso
      openshiftVersion: '4.8'
      url: registry.redhat.io/openshift4/ose-local-storage-mustgather-rhel8
  osImages:
    - cpuArchitecture: x86_64
      openshiftVersion: '4.8'
      rootFSUrl: >-
        https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.8/4.8.14/rhcos-live-rootfs.x86_64.img
      url: >-
        https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.8/4.8.14/rhcos-4.8.14-x86_64-live.x86_64.iso
      version: 48.84.202109241901-0
    - cpuArchitecture: x86_64
      openshiftVersion: '4.9'
      rootFSUrl: >-
        https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.9/4.9.0/rhcos-live-rootfs.x86_64.img
      url: >-
        https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.9/4.9.0/rhcos-4.9.0-x86_64-live.x86_64.iso
      version: 49.84.202110081407-0
    - cpuArchitecture: arm64
      openshiftVersion: '4.9'
      rootFSUrl: >-
        https://mirror.openshift.com/pub/openshift-v4/aarch64/dependencies/rhcos/4.9/4.9.0/rhcos-4.9.0-aarch64-live-rootfs.aarch64.img
      url: >-
        https://mirror.openshift.com/pub/openshift-v4/aarch64/dependencies/rhcos/4.9/4.9.0/rhcos-4.9.0-aarch64-live.aarch64.iso
      version: 49.84.202110080947-0
    - cpuArchitecture: x86_64
      openshiftVersion: '4.10'
      rootFSUrl: >-
        https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.10/4.10.3/rhcos-4.10.3-x86_64-live-rootfs.x86_64.img
      url: >-
        https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.10/4.10.3/rhcos-4.10.3-x86_64-live.x86_64.iso
      version: 410.84.202201251210-0
EOF
~~~


Patch a storageclass to default that can be consumed by the Infrastructure Operator
~~~bash
$ oc patch storageclass ocs-storagecluster-ceph-rbd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
storageclass.storage.k8s.io/ocs-storagecluster-ceph-rbd patched
~~~

Also configure the provisioning configuration to watch all namespaces

~~~bash
$ oc patch provisioning provisioning-configuration --type merge -p '{"spec":{"watchAllNamespaces": true}}'
provisioning.metal3.io/provisioning-configuration patched
~~~

~~~bash
NAMESPACE=kni25
CLUSTERNAME=kni25
INFRAENV=kni25infra
BASEDOMAIN=pemlab.rdu2.redhat.com
SSHKEY=~/.ssh/id_rsa.pub
PULLSECRETNAME=${INFRAENV}-pullsecret
MACHINE_CIDR_CLUSTER=10.11.176.0/24
OCP_RELEASE_VERSION="4.10.3"
OCP_ARCH="x86_64"
HYPERSHIFT_IMAGE=quay.io/hypershift/hypershift-operator:latest
~~~

~~~bash
$ mkdir /tmp/kube
$ cp kubeconfig-kni27 /tmp/kubeconfig/
$ cd /tmp/kubeconfig/
~~~

~~~bash
$ alias hypershift="podman run --net host --rm --entrypoint /usr/bin/hypershift -e KUBECONFIG=/working_dir/kubeconfig-kni27 -v $HOME/.ssh:/root/.ssh -v /tmp/kube:/working_dir $HYPERSHIFT_IMAGE"
~~~

~~~bash
$ hypershift install --hypershift-image $HYPERSHIFT_IMAGE
WARN[0000] error mounting subscriptions, skipping entry in /usr/share/containers/mounts.conf: getting host subscription data: failed to read subscriptions from "/usr/share/rhel/secrets": open /usr/share/rhel/secrets/rhsm/rhsm.conf: permission denied
created PriorityClass /hypershift-control-plane
created PriorityClass /hypershift-etcd
created PriorityClass /hypershift-api-critical
applied Namespace /hypershift
applied ServiceAccount hypershift/operator
applied ClusterRole /hypershift-operator
applied ClusterRoleBinding /hypershift-operator
applied Role hypershift/hypershift-operator
applied RoleBinding hypershift/hypershift-operator
applied Deployment hypershift/operator
applied Service hypershift/operator
applied Role hypershift/prometheus
applied RoleBinding hypershift/prometheus
applied ServiceMonitor hypershift/operator
applied PrometheusRule hypershift/metrics
applied CustomResourceDefinition /clusterresourcesetbindings.addons.cluster.x-k8s.io
applied CustomResourceDefinition /clusterresourcesets.addons.cluster.x-k8s.io
applied CustomResourceDefinition /clusterclasses.cluster.x-k8s.io
applied CustomResourceDefinition /clusters.cluster.x-k8s.io
applied CustomResourceDefinition /machinedeployments.cluster.x-k8s.io
applied CustomResourceDefinition /machinehealthchecks.cluster.x-k8s.io
applied CustomResourceDefinition /machinepools.cluster.x-k8s.io
applied CustomResourceDefinition /machines.cluster.x-k8s.io
applied CustomResourceDefinition /machinesets.cluster.x-k8s.io
applied CustomResourceDefinition /agentclusters.capi-provider.agent-install.openshift.io
applied CustomResourceDefinition /agentmachines.capi-provider.agent-install.openshift.io
applied CustomResourceDefinition /agentmachinetemplates.capi-provider.agent-install.openshift.io
applied CustomResourceDefinition /awsclustercontrolleridentities.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /awsclusterroleidentities.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /awsclusters.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /awsclusterstaticidentities.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /awsclustertemplates.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /awsfargateprofiles.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /awsmachinepools.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /awsmachines.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /awsmachinetemplates.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /awsmanagedmachinepools.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /azureclusteridentities.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /azureclusters.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /azuremachines.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /azuremachinetemplates.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /ibmpowervsclusters.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /ibmpowervsmachines.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /ibmpowervsmachinetemplates.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /ibmvpcclusters.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /ibmvpcmachines.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /ibmvpcmachinetemplates.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /kubevirtclusters.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /kubevirtmachines.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /kubevirtmachinetemplates.infrastructure.cluster.x-k8s.io
applied CustomResourceDefinition /awsendpointservices.hypershift.openshift.io
applied CustomResourceDefinition /hostedclusters.hypershift.openshift.io
applied CustomResourceDefinition /hostedcontrolplanes.hypershift.openshift.io
applied CustomResourceDefinition /nodepools.hypershift.openshift.io
~~~

~~~bash
$ PATCH='[
{"op":"add","path":"/rules/-","value":{"apiGroups":["extensions.hive.openshift.io"],"resources":["agentclusterinstalls"],"verbs":["*"]}}, {"op":"add","path":"/rules/-","value":{"apiGroups":["hive.openshift.io"],"resources":["clusterdeployments"],"verbs":["*"]}},
{"op":"add","path":"/rules/-","value":{"apiGroups":["cluster.open-cluster-management.io"],"resources":["managedclustersets/join"],"verbs":["create"]}}
 ]'
~~~

