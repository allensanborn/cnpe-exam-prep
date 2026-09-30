# The lab's tooling on Debian. The cluster runs in the docker-in-docker feature
# (devcontainer.json), so kind, LoadBalancer IPs, /etc/hosts and localhost
# port-forwards behave as they do on a Linux host.
FROM jdxcode/mise:2026.9.15@sha256:54bc09dd85c4af5b198e998b666852a1d018f9d513462859fd7e848047a7514c AS mise

FROM mcr.microsoft.com/devcontainers/base:trixie@sha256:1f851004adcd3dff3776b4a1da86727cf52280b0fdc9d5b0568c9c7d74274286

# python3-yaml for 94-fix-cp-metrics.sh; fonttools and brotli for `make fonts`.
RUN apt-get update && apt-get install -y --no-install-recommends \
      bash-completion make procps python3 python3-brotli python3-fonttools \
      python3-yaml skopeo unzip wget \
 && rm -rf /var/lib/apt/lists/*

# Log rotation for the inner dockerd: kind node logs fill the disk otherwise.
RUN mkdir -p /etc/docker && printf '%s\n' \
      '{ "log-driver": "json-file", "log-opts": { "max-size": "10m", "max-file": "3" } }' \
      > /etc/docker/daemon.json

COPY --from=mise /usr/local/bin/mise /usr/local/bin/mise

USER vscode
ENV MISE_TRUSTED_CONFIG_PATHS=/workspaces
WORKDIR /workspaces/cnpe-exam-prep
# Installing here bakes the locked CLIs into an image layer; the workspace mount
# replaces these two files with the live ones.
COPY --chown=vscode:vscode mise.toml mise.lock ./
RUN mise install --locked && mise run shell
