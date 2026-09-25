# INSTRUCTION.md
 
## Task: RBAC configuration for Django ToDo List app
 
This task extends the Django ToDo List Kubernetes deployment by adding RBAC (Role-Based Access Control) configuration, allowing the application's pod to authenticate against the Kubernetes API and list secrets in its namespace.
 
## What was done
 
1. Added `cluster.yml` — the `kind` cluster configuration used to spin up the local Kubernetes cluster.
2. Created a `security` directory containing an `rbac.yml` manifest with:
   - `ServiceAccount` (`pods-listener`) — identity used by the Deployment's pod
   - `Role` (`role`) — grants permission to `list` `secrets`
   - `RoleBinding` — binds the `Role` to the `ServiceAccount`
3. Updated the `Deployment` manifest to use `serviceAccountName: pods-listener`.
4. Verified access by executing a `curl` request from inside the running pod, using the mounted `ServiceAccount` token to call the Kubernetes API and list secrets.
## How to validate
 
### 1. Create the cluster
 
Run the following command from the repository root:
 
    kind create cluster --name todoapp-cluster --config cluster.yml
 
### 2. Set kubectl context (if not set automatically)
 
    kubectl config use-context kind-todoapp-cluster
 
### 3. Create the namespace (if not already created by a manifest)
 
    kubectl create namespace todoapp
 
### 4. Apply the RBAC manifest
 
    kubectl apply -f security/rbac.yml -n todoapp
 
### 5. Apply the Deployment manifest (and any other required manifests: ConfigMap, Secret, PVC, Service, etc.)
 
    kubectl apply -f <path-to-deployment-manifest> -n todoapp
 
### 6. Confirm the RBAC objects were created correctly
 
    kubectl get serviceaccounts -n todoapp
    kubectl get role -n todoapp
    kubectl get rolebindings -n todoapp
    kubectl describe rolebinding <rolebinding-name> -n todoapp
 
Confirm that `roleRef.name` in the RoleBinding output matches the actual `Role` name (`role`).
 
### 7. Confirm the pod is running and uses the correct ServiceAccount
 
    kubectl get pods -n todoapp
    kubectl describe pod <pod-name> -n todoapp
 
Check the `Service Account` field in the output — it should be `pods-listener`.
 
### 8. Get the pod name
 
    kubectl get pods -n todoapp -o jsonpath='{.items[0].metadata.name}'
 
### 9. Exec into the pod and list secrets via the Kubernetes API
 
    kubectl exec -it <pod-name> -n todoapp -- sh
 
Inside the pod shell, run:
 
    curl -s --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
    -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
    https://kubernetes.default.svc/api/v1/namespaces/todoapp/secrets
 
Alternatively, run it as a one-liner without entering an interactive shell:
 
    kubectl exec <pod-name> -n todoapp -- sh -c 'curl -s --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default.svc/api/v1/namespaces/todoapp/secrets'
 
### 10. Expected result
 
A JSON response containing the list of `Secret` objects in the `todoapp` namespace (a `SecretList` with `items`). This confirms that the `ServiceAccount → RoleBinding → Role` chain is correctly configured and the pod has the required RBAC permissions.
 
Attach a screenshot of this output to the PR as required by the task.
 
### 11. Troubleshooting
 
If the response is `Forbidden` instead of a secrets list, check:
 
- The `Role` includes `secrets` in `resources` and `list` in `verbs`.
- The `RoleBinding`'s `roleRef.name` matches the actual `Role`'s `metadata.name`.
- The `RoleBinding`'s `subjects` correctly references the `ServiceAccount` name and namespace.
- The pod's `Deployment` manifest actually sets `serviceAccountName: pods-listener`.
### 12. Cleanup
 
    kind delete cluster --name todoapp-cluster
 