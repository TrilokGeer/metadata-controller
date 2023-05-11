# Title

A new controller for metadata management.

# Status 

Proposed

# Context

Many cloud resources are created during installation of OpenShift clusters. Some the cloud resources, like, VPC, 
security groups, etc are not managed by controllers post installation. That means, user can edit the configuration for these 
cloud resources using external tools without being reconciled by OpenShift. Thus, post installation, no new 
or update to tags can be performed for these cloud resources. Whilst, there are few cloud resources managed by OpenShift controllers
like image registry operator, ingress operator, machine api, cco, etc which manage corresponding cloud resources. A uniform way of managing tags
is required for the cloud resources that are created and used by OpenShift clusters.

The system should be extensible to manage cloud resources that will be used along OpenShift for applications.

The lack of generic controllers to manage tag specific configuration and controllers will pose restrictions on maintenance and usability.

# Decision

A new generic controller that enables user to manage tags for all resources created by OpenShift installer.
A new operator that manages installation of the controller and enable lifecycle management.

# Consequences

## Good
1. Dedicated controller which avoids conflict with other infrastructure configurations.
2. It is cleaner to support and develop metadata specific add-ons and features.

## Neutral
1. A new generic non-core controller that can be installed on-demand.
2. A new day-2 operator for installation and lifecycle management for the controller pods.

## Bad
1. Additional resources for operation and maintenance on the controller.
