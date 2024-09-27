# compile the code to an executable using an intermediary image
FROM golang:1.23.1-alpine3.20 AS builder

# VULN_SCAN_TIME=

WORKDIR /build
COPY *.mod \
     *.go \
     *.sum \
     .

RUN CGO_ENABLED=0 GOOS=linux go build -ldflags '-w -s' -installsuffix cgo -a -o out/image-awaiter

# present the result within a slimmed image
FROM 201959883603.dkr.ecr.us-east-2.amazonaws.com/mdaca/base-images/ironbank-alpine:3.20.2 as final

RUN apk add --no-cache \
    shadow bash

# Copy the pause binary from the build stage, set ownership and permissions
COPY --from=builder --chown=nobody /build/out/image-awaiter /image-awaiter

# Set the binary as executable
RUN chmod 750 /image-awaiter

# Switch to non-root user
USER nobody:nobody

# Set the default entrypoint to the pause binary
ENTRYPOINT ["/image-awaiter"]
