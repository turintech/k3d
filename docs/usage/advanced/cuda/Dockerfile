# https://github.com/k3d-io/k3d/issues/1108
ARG K3S_TAG="v1.27.4-k3s1"
ARG NVIDIA_IMAGE="nvidia/cuda:11.8.0-base-ubuntu22.04"
FROM rancher/k3s:$K3S_TAG as k3s
# nvidia/cuda:11.8.0-base-ubuntu22.04
FROM nvidia/cuda:11.2.2-base-ubuntu20.04

ARG NVIDIA_CONTAINER_RUNTIME_VERSION
ENV NVIDIA_CONTAINER_RUNTIME_VERSION=$NVIDIA_CONTAINER_RUNTIME_VERSION

# Install NVIDIA Container Runtime 
RUN curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | apt-key add -
RUN curl -s -L https://nvidia.github.io/nvidia-container-runtime/ubuntu20.04/nvidia-container-runtime.list | tee /etc/apt/sources.list.d/nvidia-container-runtime.list

RUN apt-get update && \
    apt-get -y install gnupg2 curl nvidia-container-runtime=${NVIDIA_CONTAINER_RUNTIME_VERSION} && \
    chmod 1777 /tmp && \
    mkdir -p /var/lib/rancher/k3s/agent/etc/containerd && \
    mkdir -p /var/lib/rancher/k3s/server/manifests

COPY --from=k3s /bin /bin
COPY --from=k3s /etc /etc

# Provide custom containerd configuration to configure the nvidia-container-runtime
COPY config.toml.tmpl /var/lib/rancher/k3s/agent/etc/containerd/config.toml.tmpl

# Deploy the nvidia driver plugin on startup
COPY device-plugin-daemonset.yaml /var/lib/rancher/k3s/server/manifests/nvidia-device-plugin-daemonset.yaml

VOLUME /var/lib/kubelet
VOLUME /var/lib/rancher/k3s
VOLUME /var/lib/cni
VOLUME /var/log

ENV PATH="$PATH:/bin/aux"
ENV CRI_CONFIG_FILE=/var/lib/rancher/k3s/agent/etc/crictl.yaml

ENTRYPOINT ["/bin/k3s"]
CMD ["agent"]