# Title

Opcode based operations for managing tags

# Status 

Proposed

# Context

The system should be usable with explicit commands on expected behaviour for tags on cloud resources.

When designed as kubernetes API object, inferring operations based on editing the spec fields for tags will require to maintain
historical tags list cache. Also, this method would require additional implementation of behavioral policies that control 
append or replace strategies while merging the input lists.

# Decision

A set of opcodes depicting the operations add, update and delete must be used along with tags request.

# Consequences

## Good
1. Explicit operation usage and purpose. No additional tag policies required with separation of purpose by add and update operations.

## Bad
1. Reconciliation during opcode failures pose conflicting scenarios where any updates using external tools will not be reset.