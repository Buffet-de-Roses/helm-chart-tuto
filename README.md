# TUTO HELM CHART

## Lexique

#### Manifests
Manifests are the files that describe what will be deployed in a Kubernetes cluster. Your actual deployments are created by applying your manifest files to your cluster, typically using Kubectl or a package manager like Helm.

## Cluster Kubernetes

To start a kubernetes cluster using kind you can use the cmd 

```sh
kind create cluster --config kind/kind-config.yaml
```


## HELM Charts

to install a chart on the cluster kubernetes previously created.

use the cmd 

```sh
helm install full-coral .
```

Install and check how the template will render

```sh
helm install --debug --dry-run goodly-guppy .
```

**full-coral** will be the name of the chart

to see what's inside the manifest

```sh
helm get manifest full-coral
```

#### Built-in Objects

Release object.

- Name: the release name
- Namespace: the namespace to be released into
- IsUpgrade: true if the current operation is an upgrade or rollback
- IsInstall: true if the current operation is an install
- Revision: The revision number for this release. On install, this is 1, and is incremented with each upgrade and rollback
- Service: The service that is rendering the present template. On Helm, this is always Helm.


Values object.
Content from the *values.yaml* file

Chart object.
Content from the *Chart.yaml* file

Subchart object.
This provide access to the scope of subcharts to the parent. For example *.Subcharts.mySubChart.myValue* 

Files object.
This provides access to all non-special files in a chart. While you cannot use it to access templates, you can use it to access other file in the chart. 
- Get: function for getting a file by name (.Files.Get configmap.yaml)
- GetBytes: function for getting the contents of a file as an array of bytes instead of as a string. (Useful for things like images)
- Glob: function that returns a list of files whose names match the given shell glob pattern
- Lines: function that reads a file line-by-line. (Cool for iteration)
- AsSecrets: function that returns the file bodies as Base 64 encoded strings
- AsConfig: function that returns file bodies as a YAML map

Capabilites object.
This provides information about what capabilities the Kubernetes cluster supports.
- APIVersions: is a set of versions.
- APIVersions.Has $version: indicates whether a version or resource is available on the cluster.
- KubeVersion && KuberVersion.Version: is the kubernetes cluster version
- KubeVersion.Major: Kubernetes major version
- KubeVersion.Minor: Kubernetes minor version
- HelmVersion: is the object containing the Helm Version details, it is the same output of *helm version .*
- HelmVersion.Version: is the current Helm version in semver format.
- HelmVersion.GitCommit: is the Helm git sha1.

Templates object.
This contains information about the current template that iis being executed.
- Name: a namespaced file path to the current template (mychart/templates/mytemplate.yaml)
- BasePath: the namespaced path to the templates directory of the current chart. (mychart/templates).

#### Templates Functions and Pipelines

Pipelines are an efficient way of getting several things done in sequence.

Functions:

(functions lists to refer)[https://helm.sh/docs/chart_template_guide/function_list/]

- lookup:
    - apiVersion: string
    - kind: string
    - namespace: string
    - name: string

Few examples:

|Behavior|lookup function|
|--------|---------------|
|kubectl get pod mypod -n mynamespace|lookup "v1" "Pod" "mynamespace" "mypod"|
|kubectl get pods -n mynamespace|lookup "v1" "Pod" "mynamespace" ""|
|kubectl get pods --all-namespaces|lookup "v1" "Pod" "" ""|
|kubectl get namespace mynamespace|lookup "v1" "Namespace" "" "mynamespace"|
|kubectl get namespaces|lookup "v1" "Namespace" "" ""|

The following example will return the annotations present for the mynamespace object:

(lookup "v1" "Namespace" "" "mynamespace").metadata.annotations

```yaml
{{ range $index, $service := (lookup "v1" "Service" "mynamespace" "").items }}
    {{/* do something with each service */}}
{{ end }}
```

:warning: To test lookup against a running cluster, **helm template|install|upgrade|delete|rollback --dry-run=server** should be used instead to allow cluster connection :warning:

:exclamation:**Operators are function**, for templates, the operators (eq, ne, lt, gt, and, or, etc ...) are all implemented as functions. In pipelines, operations can be grouped with parentheses ( , and ).:exclamation:

#### Flow control

- if/else: for creating conditional blocks
- with: to speficy a scope
- range: provides a "for each"-style loop

- define: declares a new named template inside of your template
- template: imports a named template
- block: declares a special kind of fillable template area

##### IF/Else

```yml
{{ if PIPELINE }}
    # Do something
{{ else if OTHER PIPELINE }}
    # Do something else
{{ else }}
    # Default case
{{ end }}
```

:warning:
A pipeline is evaluated as false if the value is :
- a boolean false
- a numeric zero
- an empty string
- a nil (empty or null)
- an empty collection (map, slice, tuple, dict, array)
:warning:

##### With

This controls variable scoping. Recall that **.** is a reference to the current scope.

```yml
{{ with PIPELINE}}
    # restricted scope
{{ end }}
```

##### Range

It's the way to iterate through a collection.




# LA SUITE DU TUTO, pour mardi 21/05/2024
https://helm.sh/docs/chart_template_guide/named_templates/
