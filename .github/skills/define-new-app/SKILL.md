---
name: define-new-app
description: A skill to define a new app in the Umbrel community store by creating the necessary directory and files.
---

# Skill: Define New App

## Description
This skill guides the process of defining a new app in the Umbrel community store repository. It involves creating the app directory, umbrel-app.yml metadata file, and docker-compose.yml configuration file.

## Parameters

### Core Parameters (Always Required)
These are the metadata fields that belong in `umbrel-app.yml`.
- `id`: Unique identifier for the app (lowercase, hyphens allowed, e.g., "luis815-umbrel-ollama-amd")
- `name`: Display name shown in UI (e.g., "Ollama AMD")
- `tagline`: Short one-line description (max ~80 chars, e.g., "Self-host open source AI models")
- `category`: App category (e.g., "ai", "productivity", "utilities")
- `version`: Semantic version of the app (e.g., "0.17.4")
- `port`: External port users access the app on (e.g., 11434)
- `description`: Detailed markdown description with usage instructions, warnings, setup steps
- `developer`: Organization/developer name (e.g., "Ollama")
- `website`: Official website URL (e.g., "https://ollama.com/")
- `repo`: GitHub repository URL

### Always Required (YAML Boilerplate)
- `manifestVersion`: Always set to `1`
- `icon`: HTTPS URL to app icon (JPG/PNG)
- `submitter`: Your username or handle
- `submission`: Link to your app store repository
- `support`: Issue tracker or support URL
- `dependencies`: Array (can be empty: `[]`)
- `path`: Empty string: `""`
- `defaultUsername`: Empty string unless app requires default credentials
- `defaultPassword`: Empty string unless app requires default credentials

### Conditionally Required
- `gallery`: Array of image URLs for app showcase (optional but recommended)
- `releaseNotes`: Release notes text (optional but recommended)
- `permissions`: Array of required permissions (e.g., `["GPU"]` for GPU-dependent apps)

### Docker Configuration Parameters
These are parameters used when creating `docker-compose.yml`.
- `docker_image`: The Docker image to use
- `internal_port`: The port the app listens on inside the container (may differ from external `port`)
- `environment_vars`: Environment variables needed by the app
- `volumes`: Data persistence mappings (use `${APP_DATA_DIR}` prefix)
- `devices`: Device access (e.g., `["/dev/kfd", "/dev/dri"]` for GPU)
- `group_add`: Linux groups for permissions (e.g., `["44", "991"]` for GPU access)
- `restart_policy`: Docker restart policy (e.g., `on-failure`)

## Examples

### Example 1: Creating an Ollama AMD app (GPU-enabled)
**umbrel-app.yml parameters:**
```yaml
id: luis815-umbrel-ollama-amd
name: Ollama AMD
tagline: Self-host open source AI models like DeepSeek-R1, Llama, and more
category: ai
version: "0.17.4"
port: 11434
description: >-
  Ollama allows you to download and run advanced AI models directly on your own hardware...
developer: Ollama
website: https://ollama.com/
repo: https://github.com/ollama/ollama
icon: https://raw.githubusercontent.com/luis815/luis815-umbrel-personal-app-store/master/luis815-umbrel-ollama-amd/icon.jpg
submitter: luis815
submission: https://github.com/luis815/luis815-umbrel-personal-app-store
support: https://github.com/ollama/ollama/issues
dependencies: []
path: ""
defaultUsername: ""
defaultPassword: ""
permissions:
  - GPU
```

**docker-compose.yml service configuration:**
```yaml
version: "3.7"
services:
  app_proxy:
    environment:
      APP_HOST: luis815-umbrel-ollama-amd_luis815-umbrel-ollama-amd_1
      APP_PORT: 11434
      PROXY_AUTH_ADD: 'false'
  luis815-umbrel-ollama-amd:
    image: ollama/ollama:0.17.4
    environment:
      OLLAMA_ORIGINS: '*'
      OLLAMA_CONTEXT_LENGTH: 8192
      OLLAMA_VULKAN: 1
      HIP_VISIBLE_DEVICES: -1
    group_add:
      - "44"
      - "991"
    volumes:
      - ${APP_DATA_DIR}/data:/root/.ollama
    devices:
      - /dev/kfd
      - /dev/dri
    restart: on-failure
```

### Example 2: Creating an Open WebUI AMD app (web interface with different internal/external port)
**Key difference:** Internal port (8080) differs from external port (2876)

**docker-compose.yml service configuration:**
```yaml
version: "3.7"
services:
  app_proxy:
    environment:
      APP_HOST: luis815-umbrel-open-webui-amd_luis815-umbrel-open-webui-amd_1
      APP_PORT: 8080
      PROXY_AUTH_ADD: 'false'
  luis815-umbrel-open-webui-amd:
    image: ghcr.io/open-webui/open-webui:v0.8.5
    volumes:
      - ${APP_DATA_DIR}/data/open-webui:/app/backend/data
    environment:
      CUSTOM_VAR: value
    restart: on-failure

> Note: Some apps may also include cross-app integration variables in `docker-compose.yml`, such as `OLLAMA_BASE_URL` for Open WebUI when connecting to an Ollama service running in the same Umbrel instance.
```

## Implementation

1. **Determine store ID**: Read `umbrel-app-store.yml` to get the store ID (e.g., "luis815-umbrel").
2. **Create app directory**: The directory should be named using the full `id` value from umbrel-app.yml (e.g., "luis815-umbrel-ollama-amd").
3. **Create umbrel-app.yml**: 
   - Include all required fields from "Core Parameters" and "Always Required" sections
   - Use proper YAML formatting with multi-line strings using `>-` for descriptions
   - Set `manifestVersion: 1` always
4. **Create docker-compose.yml**: 
   - Set version to `"3.7"`
   - Create `app_proxy` service with correct `APP_HOST` naming convention
   - Create main app service with image, environment, volumes, devices as needed
   - Include `restart: on-failure` policy
5. **Validate**: Check for any errors in the generated files and ensure they follow the repository conventions.

## Common Configurations

### GPU-Enabled Apps (AMD/Vulkan)
Use for apps requiring GPU acceleration on AMD devices:
```yaml
environment:
  OLLAMA_VULKAN: 1
  HIP_VISIBLE_DEVICES: -1
  OLLAMA_FLASH_ATTENTION: 0
  OLLAMA_VRAM_OVERRIDE: 8589934592
group_add:
  - "44"
  - "991"
devices:
  - /dev/kfd
  - /dev/dri
```

### Data Persistence
Always use `${APP_DATA_DIR}` prefix for persistent storage:
```yaml
volumes:
  - ${APP_DATA_DIR}/data:/app/data
```

### Restart Policy
Recommended for production apps:
```yaml
restart: on-failure
```

### Proxy Service Format
The `APP_HOST` must follow the pattern: `{app-directory-name}_{service-name}_1`

All three components are interconnected through the `APP_HOST` format:
  - **Directory name**: Matches the `id` value in `umbrel-app.yml` (e.g., `luis815-umbrel-ollama-amd`)
  - **Service name**: Arbitrary, but it is common to use the directory name for consistency
  - **APP_HOST pattern**: `{directory-name}_{service-name}_1`
    - The `_1` suffix is automatically appended by Docker Compose to the service name

Example breakdown:
- `umbrel-app.yml` has `id: luis815-umbrel-ollama-amd`
- Directory created as: `luis815-umbrel-ollama-amd`
- Service name in docker-compose.yml: `luis815-umbrel-ollama-amd` (could be different)
- Resulting `APP_HOST`: `luis815-umbrel-ollama-amd_luis815-umbrel-ollama-amd_1`

The `APP_PORT` is the internal container port (may differ from external `port`):
```yaml
app_proxy:
  environment:
    APP_HOST: luis815-umbrel-ollama-amd_luis815-umbrel-ollama-amd_1
    APP_PORT: 11434
    PROXY_AUTH_ADD: 'false'
```

## Validation Checklist

### Directory & Files
- [ ] Directory name matches the `id` value from `umbrel-app.yml` (all lowercase)
- [ ] Contains `umbrel-app.yml` file
- [ ] Contains `docker-compose.yml` file
- [ ] Optional: `icon.jpg` image file in directory
- [ ] Optional: Gallery images in directory

### umbrel-app.yml Validation
- [ ] `manifestVersion: 1` present
- [ ] `id`, `name`, `tagline`, `category`, `version`, `port`, `description` all present
- [ ] `developer`, `website`, `repo` URLs valid
- [ ] `icon`, `submitter`, `submission`, `support` all present
- [ ] All YAML keys use correct casing (camelCase)
- [ ] `description` uses `>-` for multi-line text (literal block, no trailing newline)
- [ ] Arrays use proper YAML syntax: `[]` for empty, `- item` for items
- [ ] No trailing whitespace
- [ ] Valid YAML (can be validated with `yq eval` or YAML validator)

### docker-compose.yml Validation
- [ ] `version: "3.7"` specified
- [ ] `app_proxy` service present with `APP_HOST`, `APP_PORT`, `PROXY_AUTH_ADD`
- [ ] `APP_HOST` follows pattern: `{app-directory-name}_{service-name}_1` (where `_1` is automatically appended by Docker Compose)
- [ ] Main app service name matches the app-directory-name (which matches the `id` field in umbrel-app.yml)
- [ ] `id` from `umbrel-app.yml` matches the directory name and service name
- [ ] `image` field has valid Docker image name with tag
- [ ] Service includes `restart: on-failure`
- [ ] Volumes use `${APP_DATA_DIR}` prefix for persistence
- [ ] GPU-enabled apps have proper `group_add` and `devices` mappings
- [ ] All environment variables properly quoted
- [ ] Valid YAML formatting

### Quality Checks
- [ ] Description includes setup instructions or important warnings
- [ ] Description uses markdown formatting appropriately
- [ ] Port numbers are reasonable and don't conflict with common services
- [ ] Docker image version is pinned (not `latest`)
- [ ] All external URLs are HTTPS
- [ ] Consistent naming with other apps in the store