---
title: "5. Debugging Helm"
weight: 5
sectionnumber: 5
---

Debugging templates can be tricky because the rendered templates are sent to the {{% param distroName %}} API server which may reject the YAML files for reasons other than formatting.

There are a few commands that can help you debug:

* `helm lint` is your go-to tool for verifying that your chart follows best practices.
* `helm install --dry-run --debug --generate-name <chart> --namespace <namespace>` or `helm template --debug <chart>`: We’ve seen this trick already. It’s a great way to render your templates without applying them to the cluster.
* `helm template -s templates/<template-file> . --debug | cat -n -` render a single file to the standard output
* `helm get manifest <release> --namespace <namespace>`: This is a good way to see what templates are installed on the server.
* `helm get values <release> --namespace <namespace>`: This helps you to understand which values are used for a release.

When your YAML is failing to parse but you want to see what is generated, one easy way to retrieve the YAML is to comment out the problem section in the template and then re-run `helm install --dry-run --debug --generate-name <chart>`.

```
apiVersion: v2
# some: problem section
# {{ .Values.foo | quote }}
```

The above will be rendered and returned with the comments intact:

```
apiVersion: v2
# some: problem section
#  "bar"
```

This provides a quick way of viewing the generated content without YAML parse errors blocking.


## Task {{% param sectionnumber %}}.1: Get the chart

Get the `error-chart` chart by either [downloading the repository's ZIP file](https://github.com/acend/error-chart/archive/main.zip) or by cloning it from GitHub:

```bash
git clone https://github.com/acend/error-chart.git
```


## Task {{% param sectionnumber %}}.2: Fix the chart

The `error-chart` chart contains some deliberate errors. Try to find all of these errors and fix them, then install the chart using `myrelease` as release name. You can use one or several ways shown above to do so.
The goal of this task is a successfully running `myrelease-error-chart` pod in your own namespace.

If you try to install the chart with following command, you will get an error:
```bash
helm install my-error-chart . --dry-run
```


### Hints


#### YAML

YAML has some strict formatting rules. Check if all files conform to these rules. There is a bad formatting in the `ingress.yaml` template.
Use the `helm template -s templates/ingress.yaml . --debug | cat -n -` command to render the ingress template. Be aware that the first two lines (`---` YAML directive marker and the `# Source: error-chart/templates/ingress.yaml` YAML comment are not considered).


#### Kubernetes resource definitions

Find out what resource files are not correct resource definitions.


#### Values

Check whether all defined values in `values.yaml` look ok to you. Also check if those values used in the templates reference the correct ones.


### Solution


#### Ingress path

The first error we get when trying to install the chart or when using the `helm lint` command is this:

```
[ERROR] templates/ingress.yaml: unable to parse YAML
  error converting YAML to JSON: yaml: line 15: found character that cannot start any token
```

"found character that cannot start any token" probably doesn't ring a bell so we try to find out what's wrong with line 15 in file `templates/ingress.yaml`. Beware that line 15 corresponds to line 15 of the rendered file!
If you want to know what is on line 15 in the rendered file, you can execute the following command:

```bash
helm template -s templates/ingress.yaml . --debug | cat -n -
```

{{% details title="Hint / Solution" %}}
 Opening the file and going to the appropriate line containing `paths:`, we notice that a tab instead of whitespace characters was used to indent. Replace it with whitespace characters.

```yaml
...
spec:
{{- if .Values.ingress.tls }}
  tls:
  {{- range .Values.ingress.tls }}
    - hosts:
      {{- range .hosts }}
        - {{ . | quote }}
      {{- end }}
      secretName: {{ .secretName }}
  {{- end }}
{{- end }}
  rules:
  {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths: #<----- replace tab with whitespace
          {{- range .paths }}
          - path: {{ . }}
            pathType: ImplementationSpecific
            backend:
              service:
                name: {{ $fullName }}
                port: 
                  number: {{ $svcPort }}
        {{- end }}
  {{- end }}
{{- end }}
```
{{% /details %}}


#### Deployment empty selector

The second error we get when trying to install the chart reads:

```
Error: release myrelease failed: Deployment.apps "myrelease-error-chart" is invalid: spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string(nil), MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: empty selector is invalid for deployment
```

{{% details title="Hint / Solution" %}}
If you look at the deployment template or its rendered form you'll notice that something's wrong with the selector:

```
  selector:
    matchLabels:
    app.kubernetes.io/name: {{ include "error-chart.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
```

Those two key-value pairs `app.kubernetes.io/name` and `app.kubernetes.io/instance` should be indented by two more whitespaces to look like this:

```
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "error-chart.name" . }}
      app.kubernetes.io/instance: {{ .Release.Name }}
```
{{% /details %}}


#### Ingress host

The host value seems to be incorrect (parts of the url were replaced by placeholders in the following message):

```
Error: release myrelease failed: Ingress.extensions "myrelease-error-chart" is invalid: spec.rules[0].host: Invalid value: "helmtechlab-errorchart-<namespace>.{{% param labAppUrl %}}": a DNS-1123 subdomain must consist of lower case alphanumeric characters, '-' or '.', and must start and end with an alphanumeric character (e.g. 'example.com', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*')
```

{{% details title="Hint / Solution" %}}
If you look closely, you'll notice that the placeholder `<namespace>` is still in the host's value. This does not correspond to hostname conventions. The file `templates/ingress.yaml` reveals that the host's value is defined in the `values.yaml` file. Fix the `host` value.
{{% /details %}}


#### Deployment image tag


Even though the chart could successfully be installed the pod has a status of `InvalidImageName`. Looking at the rendered deployment we notice that the image is missing a tag:

```
    spec:
      containers:
      - image: 'nginx:'
        imagePullPolicy: IfNotPresent
```

As with the ingress host, the used value seems to be wrong. The ingress template has the following definition of `image:`:

```
          image: "{{ .Values.image.repository }}:{{ .Values.image.tags }}"
```

So let's have a look at the `values.yaml` file:

```
image:
  repository: nginx
  tag: stable
  pullPolicy: IfNotPresent
```

{{% details title="Hint / Solution" %}}
There's no variable `tags`, instead it's named `tag`. Fix that in the template so that the `image:` line reads:

```
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

You might also have to change the `repository:` value because Docker Hub might not be accessible from the used environment.
{{% /details %}}


#### Compare with solution

You can find the solution in the repository under the `solution` branch.
If you want to check if your solution is correct, you can simply compare your workspace against the solution branch.

```bash
git diff origin/solution
```

{{% onlyWhen openshift %}}


#### Deployment image

We've already encountered this one in [lab 2](../02/). Because of the used image, which wants to have root and privileged port numbers, it cannot be started on OpenShift.

We need to change the default image to an unprivileged one (`nginxinc/nginx-unprivileged:latest`) and also change the containerPort to `8080`.
{{% /onlyWhen %}}
