# INSTRUCTION.md

## Task: RBAC configuration for Django ToDo List app

This task extends the Django ToDo List Kubernetes deployment by adding RBAC (Role-Based Access Control) configuration, allowing the application's pod to authenticate against the Kubernetes API and list secrets in its namespace.

## What was done

1. Created a `security` directory containing an `rbac.yml` manifest with:
   - `ServiceAccount` — identity used by the Deployment's pod
   - `Role` — grants permission to `list` `secrets`
   - `RoleBinding` — binds the `Role` to the `ServiceAccount`
2. Updated the `Deployment` manifest to use the newly created `ServiceAccount`.
3. Verified access by executing a `curl` request from inside the running pod, using the mounted `ServiceAccount` token to call the Kubernetes API and list secrets.

## How to validate

1. Create the cluster using `kind`: