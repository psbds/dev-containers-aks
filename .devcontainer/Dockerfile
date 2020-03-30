#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

FROM debian:9

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Verify git, process tools installed
    && apt-get -y install git iproute2 procps \
    #
    # Install the Azure CLI
    && apt-get install -y apt-transport-https curl gnupg2 lsb-release \
    && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $(lsb_release -cs) main" > /etc/apt/sources.list.d/azure-cli.list \
    && curl -sL https://packages.microsoft.com/keys/microsoft.asc | apt-key add - 2>/dev/null \
    && apt-get update \
    && apt-get install -y azure-cli \
    #
    # Install AKS Preview Extension
    && az extension add --name aks-preview \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Install Jq
    && sudo apt-get install jq -y \
    # Install uuid-runtime
    && sudo apt-get install uuid-runtime \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*


RUN cd ~ \
    && mkdir Downloads

# Install istioctl latest release
RUN cd ~/Downloads \
    && curl -L https://istio.io/downloadIstio | sh - \
    && cd "$(ls | grep istio)" \
    && mv bin/istioctl /usr/local/bin

# Install kubectl latest release
RUN cd ~/Downloads \
    && curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt  --insecure`/bin/linux/amd64/kubectl --insecure \
    && chmod +x ./kubectl \
    && sudo mv ./kubectl /usr/local/bin/kubectl

# Install veleroctl 1.2.0
RUN cd ~/Downloads \
    && curl -LO https://github.com/vmware-tanzu/velero/releases/download/v1.2.0/velero-v1.2.0-linux-amd64.tar.gz \
    && tar -xvf velero-v1.2.0-linux-amd64.tar.gz \
    && mv velero-v1.2.0-linux-amd64/velero /usr/local/bin

# Install Helm 3.1.0
RUN cd ~/Downloads \
    && curl -LO https://get.helm.sh/helm-v3.1.0-linux-amd64.tar.gz \
    && tar -zxvf helm-v3.1.0-linux-amd64.tar.gz \
    && mv linux-amd64/helm /usr/local/bin/helm

# Installing Kubectx and Kubens
RUN cd ~/Downloads \
    && curl -L https://github.com/ahmetb/kubectx/archive/v0.8.0.tar.gz -o kubectx.tar.gz \
    && tar -zxvf kubectx.tar.gz \
    && mv kubectx-0.8.0/kubectx /usr/local/bin \
    && mv kubectx-0.8.0/kubens /usr/local/bin 

RUN cd ~/Downloads \
    && curl -L https://github.com/jonmosco/kube-ps1/archive/v0.7.0.tar.gz -o kube-ps1.tar.gz \
    && tar -zxvf kube-ps1.tar.gz \
    && mv kube-ps1-0.7.0/kube-ps1.sh /usr/local/bin/kube-ps1 \
    && echo "source kube-ps1 && KUBE_PS1_SYMBOL_DEFAULT=\"\" && KUBE_PS1_SEPARATOR=\"\" && PS1='\u@\h \w \$(kube_ps1)\$ '" >> ~/.bashrc

RUN rm -rf ~/Downloads

#https://github.com/ahmetb/kubectx/archive/v0.8.0.zip
# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog