## 7d.	(Vault Enterprise) Define control groups and describe their basic workflow
- Control groups add an additional authorization requirement on configured paths
- When a control group is created
  - The client makes a request to a path
  - Vault returns a wrapping token - rather than requested secrets.
  - Client sends accessor to authorizers
  - The authorizers defined in the control group policy must approve the request using accessor
  - Once all authorizations are satisfied, the client can unwrap the secrets
- Can be specific in ACL policies or Sentinel policies
- Currently only supported Control Group factor is an Identity group
  - Authorizer must belong to a specific group
  - The policy will define the group or groups who are approvers for the requested path
- Example ACL policy
```
path "kv/data/max" {
    capabilities = [ "read" ]
    control_group = {
        factor "approvers" {
            identity {
                group_names = ["manager-group" ]
                approvals = 1
            }
        }
    }
}
```
- Example Sentinel Policy
```
import "controlgroup"

control_group = func() {
  numAuthzs = 0
  for controlgroup.authorizations as authz {
    if "manager-group" in authz.groups.by_name {
      numAuthzs = numAuthzs + 1
    }
  }
  if numAuthzs >= 2 {
    return true
  }
  return false
}

main = rule {
  control_group()
}
```