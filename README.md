# Kubernetes Node Ansible Role
Configures a Kubernetes node.

SSL is enabled by default and requires certificates to be configured.
See the [SSL Configuration](#ssl-configuration) section for more information.

## Configuration
| Key | Description |
|-----|-------------|
| ``kube_docker_registry``   | The base Docker registry used to pull Kubernetes related images. |
| ``kube_apiserver_url``     | The URL used to access to apiserver. |
| ``kube_cluster_dns``       | The DNS server to be used for pods. Defaults to 10.0.0.10 as used by the ``jumperfly.coredns`` role. |
| ``kube_cluster_domain``    | The cluster domain to be used. |
| ``kube_node_kubelet_args`` | The kubelet arguments passed to the ``jumperfly.kubelet`` role as the ``kubelet_args`` variable. |
| ``kube_pod_cidr_range``    | The IP range to be used for pods. |

## Configuration (dependencies)
Some of the relevant configuration from dependencies is shown below.
| Key | Dependency |
|-----|------------|
| ``kube_version`` | jumperfly.kubelet |

## SSL Configuration
Keys and certificates are loaded from the locations shown below.
The ``jumperfly.ssl_cert`` role can be used to generate them if required. For example, the following will work with the default configuration:
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
| ``kube_key``     | The location of the private key. Defaults to ``/srv/kubernetes/kubelet-{{ ansible_hostname }}-key.pem``. |
| ``kube_cert``    | The location of the certificate. Defaults to ``/srv/kubernetes/kubelet-{{ ansible_hostname }}.pem``. |
| ``kube_ca_cert`` | The location of the CA certificate chain. Defaults to ``/srv/kubernetes/kubelet-{{ ansible_hostname }}-chain.pem`` |
