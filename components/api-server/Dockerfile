FROM registry.access.redhat.com/ubi9/go-toolset:1.26.5-1786023237@sha256:5d26ff5606bd6590930e7cfc202b510e3fe2c7a7a1720860f444ab49c45128cb AS builder

USER root

WORKDIR /workspace

COPY go.mod go.sum ./
RUN go mod edit -dropreplace github.com/openshift-online/rh-trex-ai && go mod download

COPY cmd/ cmd/
COPY pkg/ pkg/
COPY plugins/ plugins/
COPY openapi/ openapi/

ARG GIT_VERSION=unknown
ARG BUILD_TIME=unknown
RUN go build -mod=mod -ldflags="-s -w \
  -X github.com/openshift-online/hypershell/components/api-server/pkg/api.Version=${GIT_VERSION} \
  -X 'github.com/openshift-online/hypershell/components/api-server/pkg/api.BuildTime=${BUILD_TIME}'" \
  -o hypershell ./cmd/hypershell

FROM registry.access.redhat.com/ubi9/ubi-minimal:9.8-1786323074@sha256:57c8151c51445a07e503dab9dc9211dc3cdeac9d45ed81a10954b7d770659b3b

WORKDIR /

COPY --from=builder /workspace/hypershell /usr/local/bin/

USER 1001
EXPOSE 8000
EXPOSE 9000

ENTRYPOINT ["/usr/local/bin/hypershell"]
CMD ["serve"]

LABEL name="hypershell" \
      version="0.0.1" \
      summary="HyperShell API Server" \
      description="REST + gRPC API server for HyperShell fleet management"
