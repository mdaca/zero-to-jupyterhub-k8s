# FIXME: use of alpine:3.19 makes iptables work in "nf_tables" mode instead of
#        "legacy" mode, and then our init container breaks.
#
#        ref: https://github.com/jupyterhub/zero-to-jupyterhub-k8s/issues/3368
#
FROM 201959883603.dkr.ecr.us-east-2.amazonaws.com/mdaca/base-images/ironbank-alpine:3.20.2

# VULN_SCAN_TIME=2024-09-16_05:05:41

RUN apk add --no-cache iptables
