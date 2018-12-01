# Kubernetes Node Ansible Role
Configures a Kubernetes node.

When configured as a master the role includes `jumperfly.etcd_node`.
This is not declared as a dependency to allow kubelet to be configured before
running the etcd pod.

SSL is enabled by default and requires certificates to be configured.
See the [SSL Configuration](#ssl-configuration) section for more information.

## Configuration
| Key | Description |
|-----|-------------|
| `kube_docker_registry`   | The base Docker registry used to pull Kubernetes related images. |
| `kube_apiserver_url`     | The URL used to access to apiserver. |
| `kube_cluster_dns`       | The DNS server to be used for pods. Defaults to 10.0.0.10 as used by the `jumperfly.coredns` role. |
| `kube_cluster_domain`    | The cluster domain to be used. |
| `kube_node_kubelet_args` | The kubelet arguments passed to the `jumperfly.kubelet` role as the `kubelet_args` variable. |
| `kube_pod_cidr_range`    | The IP range to be used for pods. |

## Configuration (master)
| Key | Description |
|-----|-------------|
| `kube_master`                      | Whether to configure the node as a master node (installs apiserver, etc.) |
| `kube_master_taint_node`           | Whether to taint the master node to prevent pods being scheduled on the master. Generally the default of `yes` should be used unless running a single node cluster. |
| `kube_apiserver_iface`             | The network interface used to listen for apiserver requests. Defaults to the Ansible default as described by the default_ipv4 fact. |
| `kube_apiserver_etcd_group`        | The Ansible group containing all etcd server hosts. This is used to configure apiserver to with etcd connection information. |
| `kube_apiserver_admission_plugins` | The list of apiserver [admission plugins](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) to be enabled. |
| `kube_controller_manager_iface`    | The network interface used to listen for controller manager requests. Defaults to the Ansible default as described by the default_ipv4 fact. |

## Configuration (dependencies)
Some of the relevant configuration from dependencies is shown below.
| Key | Dependency |
|-----|------------|
| `kube_version` | jumperfly.kubelet |

## SSL Configuration
Keys and certificates are loaded from the locations shown below.
The `jumperfly.ssl_cert` role can be used to generate them if required. For example, the following will work with the default configuration:
```
  - role: jumperfly.ssl_cert
    ssl_cert_type: client
    ssl_cert_file_base_dir: /srv/kubernetes
    ssl_cert_name: "kubelet-{{ ansible_hostname }}"
    ssl_cert_ca_delegate_name: kubernetes-ca
    ssl_cert_subject_common_name: "{{ ansible_hostname }}"
```

| Key | Description |
|-----|-------------|
| `kube_cert_dir` | The base directory for all certificates. All locations listed below must fall within this directory. |
| `kube_key`      | The location of the private key. Defaults to `/srv/kubernetes/kubelet-{{ ansible_hostname }}-key.pem`. |
| `kube_cert`     | The location of the certificate. Defaults to `/srv/kubernetes/kubelet-{{ ansible_hostname }}.pem`. |
| `kube_ca_cert`  | The location of the CA certificate chain. Defaults to `/srv/kubernetes/kubelet-{{ ansible_hostname }}-chain.pem` |

## SSL Configuration (master)
For master nodes, keys and certificates are additionally loaded from the locations shown below.
The `jumperfly.ssl_cert` role can be used to generate them if required. For example, the following will work with the default configuration:
```
  - role: jumperfly.ssl_cert
    ssl_cert_type: client
    ssl_cert_file_base_dir: /srv/kubernetes
    ssl_cert_name: apiserver-{{ ansible_hostname }}-etcd
    ssl_cert_ca_delegate_name: etcd-ca
    ssl_cert_subject_common_name: "kubernetes"
  - role: jumperfly.ssl_cert
    ssl_cert_type: server
    ssl_cert_file_base_dir: /srv/kubernetes
    ssl_cert_name: apiserver-{{ ansible_hostname }}-kube
    ssl_cert_ca_delegate_name: kubernetes-ca
    ssl_cert_subject_common_name: kubernetes
    ssl_cert_hosts:
      - kubernetes
      - kubernetes.default
      - kubernetes.default.svc
      - kubernetes.default.svc.cluster
      - kubernetes.default.svc.cluster.local
      - 10.0.0.1
      - "{{ ansible_default_ipv4.address }}"
```

| Key | Description |
|-----|-------------|
| `kube_apiserver_etcd_key`     | The location of the etcd client private key. Defaults to `/srv/kubernetes/apiserver-{{ ansible_hostname }}-etcd-key.pem`. |
| `kube_apiserver_etcd_cert`    | The location of the etcd client certificate. Defaults to `/srv/kubernetes/apiserver-{{ ansible_hostname }}-etcd.pem`. |
| `kube_apiserver_etcd_ca_cert` | The location of the etcd CA certificate chain. Defaults to `/srv/kubernetes/apiserver-{{ ansible_hostname }}-etcd-chain.pem` |
| `kube_apiserver_key`          | The location of the apiserver private key. Defaults to `/srv/kubernetes/apiserver-{{ ansible_hostname }}-kube-key.pem`. |
| `kube_apiserver_cert`         | The location of the apiserver certificate. Defaults to `/srv/kubernetes/apiserver-{{ ansible_hostname }}-kube.pem`. |
| `kube_apiserver_ca_cert`      | The location of the apiserver CA certificate chain. Defaults to `/srv/kubernetes/apiserver-{{ ansible_hostname }}-kube-chain.pem` |
