# Title

Managing pre-existing tags on cloud resources 

# Status 

Proposed

# Context

The system should be capable to manage pre-existing tags on cloud resources.

User tend to use existing tools to add tags or attach cloud resources with pre-existing tags. This creates a conflicting 
entries due to different sources of information.

# Decision

User decides whether a cloud resource needs to be managed. The system takes an unique pre-existing tag to identify cloud resources.
The system reconciles as per the opcodes in the tag request. Any conflicting entries updated on cloud resource will be reset 
as per request by the system. Maximum limit of tags is considered during merge of the tag lists. Pre-existing tag list (only keys) is 
given precedence for merge usecase.

# Consequences

## Good
1. The system becomes single source of truth for tags.
2. No caching mechanism required for pre-existing tags.
3. Based on opcode, the tags are merged and replaced.

## Bad
1. User would need to update existing tag policies at cloud provider.
2. User must perform validation of request against pre-existing keys when external tools are in use.