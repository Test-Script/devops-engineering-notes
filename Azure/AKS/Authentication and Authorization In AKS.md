==========================================================================================================
                    Authentication & Authorization modes in AKS during cluster creation
==========================================================================================================

1. Local accounts with Kubernetes RBAC

2. Microsoft Entra ID authentication with Azure RBAC

3. Microsoft Entra ID authentication with Kubernetes RBAC

==========================================================================================================

1. Local accounts with Kubernetes RBAC:

    Authentication:

    - Uses local Kubernetes identities : We will use local identities which are created by cluster administrator.

    - No Microsoft Entra ID integration

    Authorization:

    Uses Kubernetes native RBAC

    - Role
    - ClusterRole
    - RoleBinding
    - ClusterRoleBinding

Example : 
    kind: ClusterRoleBinding
    subjects:
    - kind: User
      name: tejas   # Local Identities Created for the User "Tejas"

Access is managed purely inside Kubernetes.

❌ Problems

- No centralized identity management
- Manual user lifecycle management
- Not enterprise-grade
- Not Zero Trust aligned

==========================================================================================================

2. Microsoft Entra ID authentication with Azure RBAC

    Authentication:
    
    - Same as above → Microsoft Entra ID

    Authorization:

    - Uses Azure RBAC instead of Kubernetes RBAC

    Instead of:

        ClusterRoleBinding

    We will assign:

az role assignment create \
  --role "Azure Kubernetes Service RBAC Viewer" \
  --assignee tejas@company.com

Azure checks permissions before allowing API access.

## Available Azure Roles

- Azure Kubernetes Service RBAC Reader
- Writer
- Admin
- Cluster Admin

## Important Limitation ⚠

Azure RBAC applies only to:

- Kubernetes API data actions
- It does NOT control:
  - Azure resource-level permissions on the AKS resource itself

==========================================================================================================

3. Microsoft Entra ID authentication with Kubernetes RBAC

    Authentication

    - User authenticates via Microsoft Entra ID
    - Token issued via: 
        kubelogin
        az aks get-credentials
    - OIDC-based authentication to AKS API server

    Authorization
    
    - Still uses Kubernetes native RBAC
    - But the identity comes from Entra ID

    Flow
    
    - User logs in via Azure AD
    - Receives OIDC token
    - API server validates token
    - Kubernetes RBAC checks permissions

Example

    subjects:
    - kind: User
      name: tejas@company.com 

## Above named Identity appeared from the Microsoft Entra ID

✔ Benefits

- Centralized identity management
- Conditional Access supported
- MFA supported
- Clean separation:
  - Azure manages identity
  - Kubernetes manages cluster permissions

✔ Best For

- Enterprise production AKS
- DevOps teams
- Fine-grained Kubernetes access control

## This is what most Azure Architects implement.


==========================================================================================================
                        Microsoft Entra ID authentication with Kubernetes RBAC in AKS
==========================================================================================================

With this option, following things will be locked for us.

## Authentication  → Microsoft Entra ID
## Authorization   → Native Kubernetes RBAC

### High-Level Workflow:

1. User runs:

    az aks get-credentials
    kubelogin convert-kubeconfig -l azurecli

2. User logs in via Entra ID (MFA/Conditional Access applies)

3. Entra ID issues an OIDC JWT token

4. Token sent to AKS API Server

5. API Server validates:

    Signature
    Audience
    Expiry
    Issuer

6. Kubernetes RBAC checks:

- RoleBindings
- ClusterRoleBindings

7. Access decision is made.

## Internall Workflow Of AKS

When you enable Entra ID integration:

- AKS registers an AAD server application
- AKS registers a client application
- OIDC issuer is configured on API server
- API server is configured to trust Entra ID
- The API server uses:

    --oidc-issuer-url
    --oidc-client-id
    --oidc-username-claim
    --oidc-groups-claim

This is fully managed by AKS.


==========================================================================================================
                                Enterprise Example (Production Scenario)
==========================================================================================================

| Team          | Requirement                       |
| ------------- | --------------------------------- |
| Dev Team      | Deploy apps only in dev namespace |
| QA Team       | Read-only access in QA namespace  |
| Platform Team | Full cluster admin                |
| Security Team | View everything                   |

👥 Step 1 — Create Entra ID Groups

In Entra ID:

- aks-dev-team
- aks-qa-team
- aks-platform-admin
- aks-security-readonly

Users are assigned to groups centrally.

This gives you:

- Centralized identity lifecycle
- Automatic access revocation when user leaves

🔐 Step 2 — Enable Entra ID Integration on AKS

When creating cluster:

az aks create \
  --name prod-aks \
  --resource-group rg-prod \
  --enable-aad \
  --enable-azure-rbac false

Important:
We disable Azure RBAC because we want Kubernetes RBAC.

🏗 Step 3 — Create Kubernetes RBAC Policies

Now Kubernetes controls permissions.

Example 1: Dev Team — Namespace Restricted Access

Create namespace:

    kubectl create namespace dev

Create Role:

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: dev-deployer
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "deployments", "services"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]

Bind to Entra ID Group:

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-team-binding
  namespace: dev
subjects:
- kind: Group
  name: aks-dev-team   # Must match Entra group object ID or display name
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-deployer
  apiGroup: rbac.authorization.k8s.io

Now:

## Dev team can deploy only in dev
## They cannot touch prod
## They cannot modify cluster-wide resources

## Example 2: QA Team — Read Only

kind: Role
metadata:
  namespace: qa
  name: qa-readonly
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "deployments"]
  verbs: ["get", "list", "watch"]

Bound to:

aks-qa-team

## Example 3: Platform Admin — Cluster Wide

Here I would like to update the practical scenario.

ClusterAdminRole has to be give to ***aks-platform-admin**.

This Group user will only be able to control the Role creation, and RoleBinding Activity for other groups i.e. 

- aks-dev-team
- aks-qa-team
- aks-platform-admin
- aks-security-readonly

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: platform-admin-binding
subjects:
- kind: Group
  name: aks-platform-admin
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io

Full control.

========================
Hands-One Output
========================

## aks-cluster-admin-binding-aad : This this binding object in Cluster.

dev-user [ ~ ]$ kubectl describe clusterrolebinding aks-cluster-admin-binding-aad
Name:         aks-cluster-admin-binding-aad
Labels:       addonmanager.kubernetes.io/mode=Reconcile
              kubernetes.io/cluster-service=true
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind   Name                                  Namespace
  ----   ----                                  ---------
  Group  bfb55ed7-fc0c-44dc-84b9-e263cbe30c26  
  Group  74166081-9146-4f9e-ba77-8e82aa3906dc 

========================

aks-dev-team : bfb55ed7-fc0c-44dc-84b9-e263cbe30c26
aks-platform-admin : 74166081-9146-4f9e-ba77-8e82aa3906dc

## Example 4: Security Team — View Only Cluster Wide

kind: ClusterRoleBinding
metadata:
  name: security-view-binding
subjects:
- kind: Group
  name: aks-security-readonly
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io

==========================================================================================================
                                    Architecturally Powerful
==========================================================================================================

| Feature             | Benefit                        |
| ------------------- | ------------------------------ |
| Entra ID            | Central identity governance    |
| MFA                 | Secure access                  |
| Conditional Access  | IP/location control            |
| Kubernetes RBAC     | Fine-grained namespace control |
| Group-based binding | Scalable                       |

==========================================================================================================
                            Why This Is Better Than Azure RBAC (In Advanced Setups)
==========================================================================================================

Azure RBAC:

- Simpler
- Less granular

Kubernetes RBAC:

- Can restrict:
  - Specific API groups
  - Specific resources
  - Specific verbs
  - Specific namespaces

==========================================================================================================
                                What Happens When Dev User Runs
==========================================================================================================

kubectl get pods -n prod

Kubernetes RBAC engine evaluates:

- Does user belong to aks-dev-team?
- Is there RoleBinding in prod namespace?
- If not → DENIED

Error:
Error from server (Forbidden)

==========================================================================================================
                                    Audit & Logging
==========================================================================================================

## All access attempts are logged in:

- Azure AD Sign-in logs
- Kubernetes audit logs
- Azure Monitor

## You can trace:

- Who accessed
- From where
- What API was called
- Was it denied or allowed

==========================================================================================================
                                    Final Mental Model
==========================================================================================================

- Entra ID → Identity source
- Kubernetes RBAC → Policy engine
- AKS API Server → Enforcement point

==========================================================================================================