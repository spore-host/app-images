# OpenRefine — built from the official upstream release (no canonical image exists).
FROM eclipse-temurin:21-jre
ARG OPENREFINE_VERSION=3.10.1
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl ca-certificates \
 && rm -rf /var/lib/apt/lists/*
WORKDIR /opt
RUN curl -fsSL "https://github.com/OpenRefine/OpenRefine/releases/download/${OPENREFINE_VERSION}/openrefine-linux-${OPENREFINE_VERSION}.tar.gz" -o or.tgz \
 && tar -xzf or.tgz \
 && rm or.tgz \
 && mv "openrefine-${OPENREFINE_VERSION}" openrefine
RUN useradd -m refine && mkdir -p /data && chown refine /data
USER refine
WORKDIR /opt/openrefine
EXPOSE 3333
# Bind all interfaces (so the published port is reachable) + no auth — access is
# gated by spore-host/spawn's :443 proxy token.
CMD ["./refine","-i","0.0.0.0","-p","3333","-d","/data","-m","4g"]
