# Metadata Controller

The metadata controller adds, replaces and reconciles metadata required for cloud resources.
The scope of the present proposal is to enable continuous management of user-defined tags for cloud resources.

## Summary

Tagging cloud resources enables users to perform administrative operations like, organize the resources, 
apply security policies, optimize operations, etc. The cloud resources can be tagged using cloud service provider tools, 
kubernetes controllers that create resources via cloud service provider api, other open source tools. A reconciliation 
using a generic metadata controller helps to keep the tags synchronized for cloud resources.

## Motivation

Users should be able to maintain lifecycle of tags for cloud resources created during installation and by other day-2 kubernetes
controllers and operators using kubernetes API.

### User stories

As a cluster admin, I want to add tags to the cloud resources created during cluster installation,  which help
for setting security policies.

As a cluster admin, I want tags to be added to the cloud resources created post cluster installation.

As a cluster admin, I want all tags to be added via metadata CR, which helps to reconcile the tag key and values.

As a cluster admin, I want to delete tags, which are not required on the cloud resources.

As a cluster admin, I want to add tags on selected resources, which are identified using specified labels/identifiers.

As a cluster admin, I want to be able to ignore tags on the cloud resource from all operations, which helps to avoid overwriting values 
set by cloud service provider's policies.

As a cluster admin, I want the tag list to be lexicographically sorted, to have a deterministic set of tags applied 
when tag list size exceeds the maximum allowed limit on cloud resource.

As a cluster admin, I want the tag list to be lexicographically sorted, to have a deterministic set of tags appended to 
existing tags for cloud resources when the tag list exceeds the maximum allowed limit on cloud resource.

As a cluster admin, I want to use a CLI, that helps to securely login to cloud service provider and perform management operations for tags.

### Goals

1. Enable continuous management operations (create, append , update and delete) applicable for tagging 
 cloud resources created by kubernetes services.

2. Provide a new custom resource of kubernetes api for specification and status of tags.

3. Enable continuous monitoring of tags applied and reconcile when there are changes made external to 
the controller's API.

4. Provide CLI for operations, cloud service provide authentication and ability to edit custom resource with ease. 

5. Provide metrics collection and audit logging features for controller and user operations.

### Non-goals

1. Managing metadata which are not related to cloud resource tagging.
2. Data-related metadata for stored objects and records.
3. Sub-set of input tags to be applied on selected cloud resource.
4. Managing conflict with cloud service provider tagging policies

## Proposal 

A new kubernetes controller to generates actions based on tag specification defined by a custom resource. The controller
reads existing data, applies tags based on overwrite policy, cloud service provider request api rate limit and maximum limit policy.
The controller would require an initial tag or label, that can be used to identify cloud resources to apply and manage tags.
Cloud resources can be ignored from being managed by the controller by using ignore tag on the cloud resource and the same 
is configured on controller's custom resource. Controller reconciles the tag list on cloud resources based on configurable 
parameter for sync period. 

### Configuration Policies

#### Overwrite policy

Overwrite policy is used when an existing entry is found for the given input tag key. The existing tag key entry may be created 
either on cloud resource or by previous edit of controller's customer resource. The policy can have following valid values.

1. POLICY_KALL - Replaces any existing entry found for tag key from the input list with new value.
2. POLICY_KSPEC - Replace any existing entry found and created using custom resource specification.
3. POLICY_LAPPEND - Appends input tag list to existing list of tags on cloud resource and entries found in custom resource specification.
4. POLICY_LREPLACE_ALL - Replaces existing tag list found on cloud resources with new input tag list. 
5. POLICY_LREPLACE_SPEC - Replaces existing tag list created using controller custom resource specification.

#### Maximum limit policy

Maximum limit policy indicates whether the input tag list can be applied partially when the length of the list exceeds the 
allowed limit on the cloud resource. The result might differ when applied along overwrite policy. Following are the list of valid values.

1. POLICY_APPLY_PARTIAL - Allows input tag list applied to be partial on cloud resource. 
2. POLICY_APPLY_STRICT - Applies input tag list strictly in complete.

#### Label policy

Label policy defines the tag that define cloud resource inclusivity for management of tags. 

1. "apps.metadata.controller/op : managed/unmanaged>" - 

### High-level workflow sequence

#### Bootstrap
1. User starts the controller with cloud service provider credentials and sync period set.
2. Controller starts to list/watch custom resource.
3. Controller queries and lists all resources based on identifier tag or apps.metadata.controller/op tag.
4. Creates list of existing tags on cloud resource and adds to custom resource.

#### Add/Update tags 

1. User adds input tag key/value pair list to custom resource specification.
2. User specifies operation to be performed on the tag list by specific opcode.
3. Controller checks overwrite policy from custom resource and populates a new list of tags.
4. Controller performs lexicographic sorting.
5. Controller trims the tag list as per maximum limit policy specific to cloud resource.
6. Controller replaces the tag list on cloud resource.

#### Delete tags 

1. User adds tag key list (optionally, value for validation) to custom resource specification.
2. User specifies operation to be performed on the tag list by specific opcode.
3. Controller checks overwrite policy from custom resource and populates a new list of tags.
4. Controller replaces the tag list on cloud resource.

#### Toggle between managed and unmanaged

TODO

## API

### Example

```yaml
kind: CloudMetadata 
metadata: 
  name : metadata
  namespace: metadata-controller
  labels:
    "app.metadata-controller": "use"
spec:
  cloudprovider:
    awsref:
      name: awstags
      namespace: metadata-controller
  classifiers:
    "kubernetes.io/cluster/test-7lpkm-crhfx" : "owned"
  controllerpolicy:
    overwrite: "POLICY_LREPLACE_ALL"
    limit: "POLICY_APPLY_PARTIAL"
```

```yaml
kind: AWSMetadata
metadata:
  name: awstags
  namespace: metadata-controller
  labels:
    app.metadata-controller: "use"
spec:
  resourcetags:
    "env": "test"
    "centre": "eng"
```

```go
type CloudMetadata struct {
    metav1.TypeMeta `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty"`
	
	Spec MetadataSpec `json:"spec"`
	
	Status MetadataStatus `json:status`
}

type MetadataSpec struct {
	CloudProviderSpec CloudProviderSpec `json:"cloudprovider"`
	GlobalClassifiers *ClassifierSpec `json:"classifier, omitempty"`
    GlobalControllerPolicy *ControllerPolicyConfig `json:"controllerpolicy, omitempty"`
}

type CloudProviderSpec struct {
	// TODO: variable list of cloud type required
	AWS *AWSMetadataRef `json:"awsmetadataref", omitempty`
	//... and other cloud providers
}

type AWSMetadataRef struct {
	Name string `json:"name"`
	Namespace string `json:"namespace"`
}

type AWSMetadata struct {
    metav1.TypeMeta `json:",inline"`
    metav1.ObjectMeta `json:"metadata,omitempty"`

    Spec AWSMetadataSpec `json:"spec"`

    Status AWSMetadataStatus `json:status`
}

type AWSMetadataSpec struct {
	ResourceTags map[string]string `json:"resourcetags"`
}

type ClassifierSpec struct {
	Classifiers map[string]string `json:"classifiers"`
}

type ControllerPolicyConfig struct {
	OverwritePolicy OPolicy `json:"overwrite, omitempty"`
	LimitPolicy LPolicy `json:"limit, omitempty"`
}
```