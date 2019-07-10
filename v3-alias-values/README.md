### v3-alias-values example

In Helm 3 (`version.BuildInfo{Version:"v3.0.0-alpha.1", GitCommit:"b9a54967f838723fe241172a6b94d18caf8bcdca", GitTreeState:"clean"}`), `helm install` and `helm upgrade` import values for aliased sub-charts differently.


`parent` chart has dependency on `child` with alias
```
dependencies:
- name: child
  version: 0.1.0
  repository: file://../child
  alias: childzzz
```

`child` deploys a `ConfigMap` as follows. The default value for `bar` set in `child/values.yaml` is `baz`
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ template "child.fullname" . }}-config
  labels:
{{ include "child.labels" . | indent 4 }}
data:
  foo: {{ .Values.bar | quote }}
```

The values file used for deployment `values.override.yaml` has the content
```
child:
  bar: something

childzzz:
  bar: something_else
```

Running `helm install test charts/parent -f values.override.yaml` produce the ConfigMap as follows:
```
$ helm get test --revision 1
MANIFEST:
---
# Source: parent/charts/child/templates/config.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: test-child-config
  labels:
    app.kubernetes.io/name: child
    helm.sh/chart: child-0.1.0
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
data:
  foo: "something"
```
Helm uses `child` (`name` of the sub-chart in `Chart.yaml`) as `.Chart.Name` (seen in `{{ template "child.fullname" . }}-config`).

Also, it renders `{{ .Values.bar }}` using the map under the `child` key.

Now run `helm upgrade test charts/parent -f values.override.yaml`, and check the ConfigMap manifest:
```
$ helm get test --revision=2
MANIFEST:
---
# Source: parent/charts/childzzz/templates/config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-childzzz-config
  labels:
    app.kubernetes.io/name: childzzz
    helm.sh/chart: childzzz-0.1.0
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
data:
  foo: "something_else"
```

Helm now uses `childzzz` (`alias` of the sub-chart in `Chart.yaml`) as `.Chart.Name`.

When rendering templates under the sub-chart, it now uses the map under the key `childzzz`, therefore `{{ .Values.bar }}` is rendered as `something_else`

Conclusion: `helm install` renders sub-chart templates using values under the sub-chart's `name` field in `Chart.yaml`, while `helm upgrade` uses values under the sub-chart's `alias` field.
