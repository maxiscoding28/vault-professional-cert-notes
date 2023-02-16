## 8b. Configure templating
### Consul tempalte
- Can render data from Consul or Vault onto the target file system
- Consul template retrieves secrets from vault
  - Manages acquisition and renewal lifecycle
  - Requires a valid Vault token to operate
- Example
```
{{ with secret "kv/data/pass" }}
{{ .Data.data.max }}
{{ end }}
```
- Workflow: 1. Create a templated file, execute consul template to retrieve data, application reads from file.
- Agent workflow. 1. Auth with Vault. 2 retrieve token. 3 Store token in sink. 4 Retrieve secrets with consul template. 5 application reads secret from file.
- Example config file for agent (full)
```
auto_auth {
  method "approle" {
    config = {
      role_id_file_path   = "/vault/agent/role-id"
      secret_id_file_path = "/vault/agent/secret-id"
      remove_secret_id_file_after_reading = false
    }
  }

  sink "file" {
    config = {
      path = "/vault/token"
    }
  }
}

template_config {
  static_secret_render_interval = "5s"
}

template {
  source      = "/vault/agent/template.ctmpl"
  destination = "/vault/render.txt"
  error_on_missing_key = true
}
```