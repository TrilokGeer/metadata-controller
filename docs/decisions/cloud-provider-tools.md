# Title

Cloud provider tools

# Status 

Proposed

# Context

In case of AWS, alternative tools like AWS tag editor or AWS resource group API, AWS organization and AWS config service 
can be used for tagging. During installation of OpenShift clusters, cloud resources are created with tags.
To perform continuous monitoring and auditing of tags and reconcile the changes, alternative tools listed would require additional 
solution based on EventBridge events. 

# Decision

A controller which can be referred to as single source of tag information that helps to monitor and reconcile any updates for 
cloud resources. The controller should list and monitor all cloud resources which have an initial tag set.

# Consequences

## Good
1. Controller is built with configuration kubernetes custom resource which is easy to manage by a kubernetes-aware user.
2. Reconciles any tag updates outside the scope of the controller.
3. Updates to custom resource can be controlled to select cluster users.

## Bad
1. Cloud resources must have an initial tag (classifier) set that can be used by controller for listing. The initial
tag is not reconciled to allow user to have selective cloud resources being monitored by the controller.
2. Controller will need blanket tagging permissions based on the scope of usage.