~~~bash
cat << EOF > ~/hypershift-$CLUSTERNAME-hostedcluster.yaml
apiVersion: hypershift.openshift.io/v1alpha1
kind: HostedCluster
metadata:
  name: ${CLUSTERNAME}
  namespace: ${NAMESPACE}
spec:
  release:
    image: "quay.io/openshift-release-dev/ocp-release:${OCP_RELEASE_VERSION}-${OCP_ARCH}"
  pullSecret:
    name: ${PULLSECRETNAME}
  sshKey:
    name: "${SSHKEY}"
  networking:
    serviceCIDR: "172.31.0.0/16"
    podCIDR: "10.132.0.0/14"
    machineCIDR: "${MACHINE_CIDR_CLUSTER}"
  platform:
    agent:
      agentNamespace: ${NAMESPACE}
    type: Agent
  dns:
    baseDomain: ${BASEDOMAIN}
  services:
  - service: APIServer
    servicePublishingStrategy:
      nodePort:
        address: api.${CLUSTERNAME}.${BASEDOMAIN}
      type: NodePort
  - service: OAuthServer
    servicePublishingStrategy:
      nodePort:
        address: api.${CLUSTERNAME}.${BASEDOMAIN}
      type: NodePort
  - service: OIDC
    servicePublishingStrategy:
      nodePort:
        address: api.${CLUSTERNAME}.${BASEDOMAIN}
      type: None
  - service: Konnectivity
    servicePublishingStrategy:
      nodePort:
        address: api.${CLUSTERNAME}.${BASEDOMAIN}
      type: NodePort
  - service: Ignition
    servicePublishingStrategy:
      nodePort:
        address: api.${CLUSTERNAME}.${BASEDOMAIN}
      type: NodePort
EOF
~~~

~~~bash
cat << EOF > ~/hypershift-$CLUSTERNAME-nodepool.yaml
apiVersion: hypershift.openshift.io/v1alpha1
kind: NodePool
metadata:
  name: ${CLUSTERNAME}
  namespace: ${NAMESPACE}
spec:
  clusterName: ${CLUSTERNAME}
  nodeCount: 0
  management:
    autoRepair: false
    upgradeType: Replace
  platform:
    type: Agent
  release:
    image: "quay.io/openshift-release-dev/ocp-release:${OCP_RELEASE_VERSION}-${OCP_ARCH}"
EOF
~~~
