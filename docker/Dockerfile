# syntax=docker/dockerfile:1
# https://docs.docker.com/engine/reference/builder/

ARG ARGOCD_VERSION="v3.1.0"
FROM quay.io/argoproj/argocd:${ARGOCD_VERSION}

ARG YQ_VERSION="v4.47.1"
ARG JQ_VERSION="1.8.1"
ARG VAULT_VERSION="1.20.2"
ARG VAULT_PLUGIN_VERSION="1.18.1"

ENV YQ_BINARY="https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64"
ENV JQ_BINARY="https://github.com/stedolan/jq/releases/download/jq-${JQ_VERSION}/jq-linux64"

ENV VAULT_TARBALL="https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip"
ENV VAULT_TARBALL_SHASUM="https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_SHA256SUMS"
ENV VAULT_PLUGIN_BINARY=https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v${VAULT_PLUGIN_VERSION}/argocd-vault-plugin_${VAULT_PLUGIN_VERSION}_linux_amd64

# Switch to root for the ability to perform install
USER root

# Install tools needed for your repo-server to retrieve & decrypt secrets, render manifests 
# (e.g. curl, awscli, gpg, sops)
RUN apt-get update \
    && apt-get install -y software-properties-common \
    && add-apt-repository -y ppa:deadsnakes/ppa \
    && apt-get update -y \
    && apt-get install -y \
        curl unzip python3.10 python3-pip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN echo "Configure YQ" \
  && export FILE_NAME="yq" \
  && curl -L --fail ${YQ_BINARY} -o /usr/local/bin/${FILE_NAME} \
  && chmod a+x /usr/local/bin/${FILE_NAME}

RUN echo "Configure JQ" \
  && export FILE_NAME="jq" \
  && curl -L --fail ${JQ_BINARY} -o /usr/local/bin/${FILE_NAME} \
  && chmod a+x /usr/local/bin/${FILE_NAME}

RUN echo "Installing Vault" \
  && cd /tmp \
  && export FILE_NAME=$(basename ${VAULT_TARBALL}) \
  && curl -L --fail ${VAULT_TARBALL} -o ${FILE_NAME} \
  && curl -L --fail ${VAULT_TARBALL_SHASUM} | grep ${FILE_NAME} > ${FILE_NAME}.sha \
  && sha256sum -c ${FILE_NAME}.sha \
  && unzip ${FILE_NAME} -d /usr/bin/ \
  && rm -f ${FILE_NAME}*

RUN echo "Configure Vault Plugin" \
  && export FILE_NAME="argocd-vault-plugin" \
  && curl -L --fail ${VAULT_PLUGIN_BINARY} -o /usr/local/bin/${FILE_NAME} \
  && chmod a+x /usr/local/bin/${FILE_NAME}

# Switch back to non-root user
USER 999
