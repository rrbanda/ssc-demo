# AI coding-agent runtime -- OpenCode flavor.
# Used by: lw-opencode-write-tests pipeline task.
# Contains: JDK 17 + Maven, OpenCode (coding agent), git + glab + gh (SCM), bash.
#
# Build & push:
#   podman build --platform linux/amd64 -t quay.io/<org>/ai-agent-maven-opencode:v1.0.0 \
#     -f images/ai-agent-maven-opencode/Dockerfile images/ai-agent-maven-opencode
#   podman push quay.io/<org>/ai-agent-maven-opencode:v1.0.0

FROM registry.access.redhat.com/ubi9/ubi-minimal:latest

USER 0

ARG MAVEN_DIST_VERSION=3.9.9
ARG GLAB_VERSION=1.48.0
ARG GH_VERSION=2.63.2

# JDK 17 + bash + build tools
RUN microdnf install -y \
      java-17-openjdk-devel \
      bash git tar gzip which findutils ca-certificates jq \
    && microdnf clean all
ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk

# Maven
RUN set -eux; \
    for url in \
      "https://repo1.maven.org/maven2/org/apache/maven/apache-maven/${MAVEN_DIST_VERSION}/apache-maven-${MAVEN_DIST_VERSION}-bin.tar.gz" \
      "https://archive.apache.org/dist/maven/maven-3/${MAVEN_DIST_VERSION}/binaries/apache-maven-${MAVEN_DIST_VERSION}-bin.tar.gz" ; do \
      if curl -fsSL --retry 5 --retry-delay 3 --retry-all-errors "${url}" -o /tmp/maven.tar.gz; then break; fi; \
    done; \
    test -s /tmp/maven.tar.gz; \
    tar -xzf /tmp/maven.tar.gz -C /opt; \
    ln -s "/opt/apache-maven-${MAVEN_DIST_VERSION}/bin/mvn" /usr/local/bin/mvn; \
    rm -f /tmp/maven.tar.gz; \
    mvn -version

# OpenCode (coding agent) — headless mode for CI/pipeline use
RUN curl -fsSL https://opencode.ai/install | bash; \
    OPENCODE_BIN=$(find / -name opencode -type f -executable 2>/dev/null | head -1); \
    if [ -z "$OPENCODE_BIN" ]; then echo "ERROR: opencode not found" && exit 1; fi; \
    cp "$OPENCODE_BIN" /usr/local/bin/opencode; \
    opencode --version

# glab (GitLab CLI)
RUN ARCH=$(uname -m | sed 's/x86_64/amd64/; s/aarch64/arm64/') \
    && curl -fsSL "https://gitlab.com/gitlab-org/cli/-/releases/v${GLAB_VERSION}/downloads/glab_${GLAB_VERSION}_linux_${ARCH}.tar.gz" \
      -o /tmp/glab.tar.gz \
    && tar -xzf /tmp/glab.tar.gz -C /usr/local bin/glab \
    && rm -f /tmp/glab.tar.gz \
    && test -x /usr/local/bin/glab

# gh (GitHub CLI)
RUN ARCH=$(uname -m | sed 's/x86_64/amd64/; s/aarch64/arm64/') \
    && curl -fsSL "https://github.com/cli/cli/releases/download/v${GH_VERSION}/gh_${GH_VERSION}_linux_${ARCH}.tar.gz" \
      -o /tmp/gh.tar.gz \
    && tar -xzf /tmp/gh.tar.gz -C /tmp \
    && cp "/tmp/gh_${GH_VERSION}_linux_${ARCH}/bin/gh" /usr/local/bin/gh \
    && rm -rf /tmp/gh.tar.gz "/tmp/gh_${GH_VERSION}_linux_${ARCH}" \
    && test -x /usr/local/bin/gh

# Tekton runs as non-root
ENV HOME=/tekton/home
RUN mkdir -p /tekton/home /.m2 \
    && chgrp -R 0 /tekton/home /.m2 \
    && chmod -R g=u /tekton/home /.m2

USER 1001
