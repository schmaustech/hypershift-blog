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

~~~bash
Patch a storageclass to default that can be consumed by the Infrastructure Operator

$ oc patch storageclass ocs-storagecluster-ceph-rbd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
storageclass.storage.k8s.io/ocs-storagecluster-ceph-rbd patched
~~~
