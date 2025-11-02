# Kustomize Inspect

Extract metadata from kustomize overlays without making any changes.

## Features

- 🔍 **Read-only inspection** - No mutations, just analysis
- 📊 **Structured output** - JSON outputs for easy consumption
- 🎯 **Workload detection** - Find all deployments, statefulsets, etc.
- 📁 **Namespace detection** - Extract target namespace
- 🔨 **Build validation** - Ensures kustomization builds successfully

## Usage

```yaml
- name: Inspect kustomization
  uses: skyhook-io/kustomize-inspect@v1
  id: inspect
  with:
    overlay_dir: deploy/overlays/production
    
- name: Use metadata
  run: |
    echo "Namespace: ${{ steps.inspect.outputs.namespace }}"
    echo "Primary deployment: ${{ steps.inspect.outputs.primary_deployment }}"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `overlay_dir` | Path to kustomize overlay | ✅ | - |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `namespace` | Resolved namespace | `production` |
| `workloads_json` | JSON array of workloads | `[{"kind":"Deployment","name":"api","namespace":"production"}]` |
| `primary_deployment` | First deployment name | `api-server` |

## Examples

### Basic inspection
```yaml
- name: Inspect overlay
  uses: skyhook-io/kustomize-inspect@v1
  id: inspect
  with:
    overlay_dir: deploy/overlays/staging
```

### Parse workloads
```yaml
- name: Inspect overlay
  uses: skyhook-io/kustomize-inspect@v1
  id: inspect
  with:
    overlay_dir: deploy/overlays/production

- name: Process workloads
  run: |
    echo "${{ steps.inspect.outputs.workloads_json }}" | jq '.[] | "\(.kind)/\(.name)"'
```

### Use with other actions
```yaml
- name: Edit kustomization
  uses: skyhook-io/kustomize-edit@v1
  id: edit
  with:
    overlay_dir: deploy/overlays/production
    image: backend
    tag: v1.2.3

- name: Inspect changes
  uses: skyhook-io/kustomize-inspect@v1
  id: inspect
  with:
    overlay_dir: ${{ steps.edit.outputs.overlay_dir }}

- name: Apply to cluster
  uses: skyhook-io/kustomize-apply@v1
  with:
    overlay_dir: ${{ steps.edit.outputs.overlay_dir }}
    namespace: ${{ steps.inspect.outputs.namespace }}
```

### Extract for commit message
```yaml
- name: Inspect for metadata
  uses: skyhook-io/kustomize-inspect@v1
  id: inspect
  with:
    overlay_dir: deploy/overlays/production

- name: Commit with context
  run: |
    git add .
    git commit -m "Deploy ${{ steps.inspect.outputs.primary_deployment }} to ${{ steps.inspect.outputs.namespace }}"
```

## Workload Types Detected

The action detects the following Kubernetes workload types:
- Deployments
- StatefulSets
- DaemonSets
- Jobs
- CronJobs

## How It Works

1. **Builds** the kustomization using `kustomize build`
2. **Parses** the output using `yq` (preferred) or `jq`
3. **Extracts** metadata without making any changes
4. **Outputs** structured JSON for easy consumption

## Prerequisites

- `kustomize` must be installed
- Either `yq` (preferred) or `jq` for parsing

## Notes

- This is a read-only action - no files are modified
- Useful for extracting metadata before deployment
- Can be used to make deployment decisions based on content
- Works with any valid kustomization