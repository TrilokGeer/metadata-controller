# Title

A centralized tag management for multi-cloud deployment

# Status 

Proposed

# Context

In the context of multi-cloud, the system should be able to manage tags for different cloud providers as an add-on 
to cluster management. User must be able to maintain cloud provider specific configurations for tagging cloud resources,
authenticating and enable cloud provider specific interfaces.

# Decision

Cloud provider configuration must be loosely coupled with controller configuration that enables system to have 
ad-hoc cloud provider addition. Both configurations are represented as different API objects.

# Consequences

1. Additional complexity for having multiple control loops in the system to list/watch different API objects.
