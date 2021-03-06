{% panel style="warning", title="Warning: Alpha Recommendations" %}
This chapter contains recommendations that are still being actively evaluated, and may
be changed in the future.
{% endpanel %}

{% panel style="info", title="TL;DR" %}
- Use **directory hierarchy to structure Resource Config**
  - Separate directories for separate Environment and Cluster [Config Variants](../app_customization/bases_and_variants.md)
{% endpanel %}

# Directory Structure Based Composition

The are several techniques for users to structure their Resource Config files.

| Type                                   | Summary               | Benefits                                           |
|----------------------------------------|-----------------------|----------------------------------------------------|
| **[Directories](structure_directories.md)**   | **Simplest approach**   | **Easy to get started and understand**               |
| [Branches](structure_branches.md)   | *More flexible*       | Loose coupling between release specific and operation changes |
| [Repositories](structure_repositories.md) | *Fine grain control*  | Isolated permissions model                         |

## Motivation

This chapter describes conventions for using **Directories** alone.

**Advantages:**

- Simplicity
  - Simple to learn
  - Simple to navigate using conventional tools
  - Simple to audit

**Drawbacks:**

- Limited permissions and ownership models
  - Read/Write access typically given at a per-repo model
- Less granular event triggers
  - Webhooks triggered for repo+branch can't be focused on a specific environment + cluster
- Harder to decouple release related changes from operational related changes
  - Changes to scale or cpu should be rolled out immediately
  - Changes to image or flags should be rolled out with the release

{% panel style="info", title="Config Repo or Mono Repo?" %}
The techniques and conventions in this Chapter work regardless of whether or not the Resource Config
exists in the same Repository as the source code that is being deployed.
{% endpanel %}

## Directory Structure

{% method %}

### Resource Config

The convention shown here should be changed and adapted as needed.

Structure:

- Put reusable bases under `*/bases/`
  - `<project-name>/bases/`
  - `<project-name>/<environment>/bases/`
- Put deployable targets under `<project-name>/<environment>/<cluster>/`

Techniques:
 
- Each Layer adds a [namePrefix](../app_management/namespaces_and_names.md#setting-a-name-prefix-or-suffix-for-all-resources) and [commonLabels](../app_management/labels_and_annotations.md#setting-labels-for-all-resources).
- Each deployable target sets a [namespace](../app_management/namespaces_and_names.md#setting-the-namespace-for-all-resources).
- Override [Pod Environment Variables and Arguments](../app_customization/customizing_pod_templates.md) using `configMapGenerator`s with `behavior: merge`.
- Perform Last-mile customizations with [patches / overlays](../app_customization/customizing_arbitrary_fields.md)

{% sample lang="yaml" %}

```bash
$ tree
.
├── bases # Used as a Base only
│   ├── kustomization.yaml
│   ├── backend
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── frontend
│   │   ├── deployment.yaml
│   │   ├── ingress.yaml
│   │   └── service.yaml
│   └── storage
│       ├── service.yaml
│       └── statefulset.yaml
├── prod # Production
│   ├── bases 
│   │   ├── kustomization.yaml # Uses bases: ["../../bases"]
│   │   ├── backend
│   │   │   └── deployment-patch.yaml # Production Env specific backend overrides
│   │   ├── frontend
│   │   │   └── deployment-patch.yaml # Production Env specific frontend overrides
│   │   └── storage
│   │       └── statefulset-patch.yaml # Production Env specific storage overrides
│   ├── us-central
│   │   ├── kustomization.yaml # Uses bases: ["../bases"]
│   │   └── backend
│   │       └── deployment-patch.yaml # us-central cluster specific backend overrides
│   ├── us-east 
│   │   └── kustomization.yaml # Uses bases: ["../bases"]
│   └── us-west 
│       └── kustomization.yaml # Uses bases: ["../bases"]
├── staging # Staging
│   ├── bases 
│   │   ├── kustomization.yaml # Uses bases: ["../../bases"]
│   └── us-west 
│       └── kustomization.yaml # Uses bases: ["../bases"]
└── test # Test
    ├── bases 
    │   ├── kustomization.yaml # Uses bases: ["../../bases"]
    └── us-west 
        └── kustomization.yaml # Uses bases: ["../bases"]
```

{% endmethod %}

{% panel style="warning", title="Applying Environment + Cluster" %}
Though the directory structure contains the cluster in the path, this won't be used by
Apply to determine the cluster context.  To Apply a specific cluster, add that cluster to the 
kubectl config`, and specify the corresponding context when running Apply.

For more information see [Multi-Cluster](accessing_multiple_clusters.md).
{% endpanel %}
