# Nginx Ingress Controller Module (mis flavor)

[![Module Version](https://img.shields.io/badge/version-0.1-blue.svg)](./facets.yaml)

## Overview

This Terraform module deploys and configures an Nginx Ingress Controller on Kubernetes clusters. The module provides comprehensive ingress management with enhanced service naming to prevent resource conflicts, supporting multiple cloud providers (AWS, Azure, GCP) and various ingress features.

## Environment Awareness

This module is environment-aware and adapts its behavior based on:
- **Cloud Provider**: Different load balancer configurations for AWS, Azure, and GCP
- **Environment Namespace**: Resource naming includes environment context to prevent conflicts
- **Domain Management**: Base domain creation can be disabled per environment
- **SSL/TLS**: Environment-specific certificate management

## Key Fixes in This Version

### 🔧 Service Naming Collision Resolution

The original error `services "ext-dtrack-dependency-track-api-server-dependency-track" already exists` has been resolved through:

1. **Enhanced External Service Naming**: Added instance name and rule key prefixes to external service names:
   ```terraform
   service_name = substr("${var.instance_name}-ext-${v.service_name}-${lookup(v, "namespace", var.environment.namespace)}-${k}", 0, 63)
   ```

2. **Unique Default Backend Services**: Improved naming for header-based routing default backends:
   ```terraform
   backend_key = "${var.instance_name}-${lookup(lookup(value, "header_based_routing", {}), "default_backend", "")}-${lookup(lookup(value, "header_based_routing", {}), "default_backend_namespace", var.environment.namespace)}-${key}"
   ```

3. **Service Name Truncation**: Added `substr()` function to ensure Kubernetes service names don't exceed 63 characters

4. **Better Collision Detection**: Enhanced logic to detect and handle naming collisions for cross-namespace service references

## Resources Created

The module creates and manages the following Kubernetes resources:

- **Helm Release**: Nginx Ingress Controller deployment with customizable values
- **Ingress Resources**: Individual ingress rules for each configured service
- **External Services**: ExternalName services for cross-namespace service references
- **Secrets**: Basic authentication secrets and custom TLS certificates
- **ConfigMaps**: Custom error page configurations
- **Route53 Records**: DNS entries for AWS-hosted domains (when applicable)

## Security Considerations

- **TLS/SSL Encryption**: Automatic certificate management via cert-manager or custom certificates
- **Basic Authentication**: Optional HTTP basic auth with auto-generated passwords
- **Private Load Balancers**: Support for internal-only load balancers
- **CORS Configuration**: Cross-origin resource sharing controls
- **Custom Headers**: Security header injection and manipulation
- **Network Policies**: Ingress-level traffic control and routing rules

## Key Features

- **Multi-Cloud Support**: Works with AWS, Azure, GCP, and generic Kubernetes
- **Header-Based Routing**: Advanced routing based on HTTP headers
- **Session Affinity**: Cookie-based session persistence
- **Custom Error Pages**: Branded error page configuration
- **Resource Management**: CPU and memory limits/requests configuration
- **Pod Disruption Budgets**: High availability settings
- **Auto-scaling**: Horizontal pod autoscaler configuration
- **Monitoring**: Prometheus metrics and monitoring integration
