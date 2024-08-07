# dt-collector-demo
Hands on demo. Ingest logs, metrics and traces using the Dynatrace OpenTelemetry Collector.

## Prerequisites

To follow this hands-on guide, you need access to a Dynatrace environment.

Go to `https://dynatrace.com/trial` if you do not already have access.

## Start Demo

Click this link to start the demo:

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/dynatrace-perfclinics/dt-collector-demo)

Wait for the system to start. A new browser window will be created and a Kubernetes cluster will be installed for you.
Towards the bottom of the screen, you should see an empty terminal prompt like this:

![terminal window](.devcontainer/images/terminal-window.png)

Soon after the codespace starts, you should see this in the terminal window. Wait until it says "Done".
While you are waiting, you can proceed to the next step to create an access token.

![post creation workflow](.devcontainer/images/post-create.png)

## Create an Access Token

Do the following in Dynatrace:

- Press `Ctrl + k` to bring up the search box.
- Type `access tokens` and select the first option
- Create an access token with these permissions: `openTelemetryTrace.ingest`, `metrics.ingest` and `logs.ingest`
- Click `Generate token` and save the token

## Create a Kubernetes secret

Use `kubectl` to create a `Secret` to store your Dynatrace connection details.

Substitute your values into the following command and execute it.

Change `abc12345` to your environment ID and change `dt0c01.sample.secret` to the value of your API token (generated above).

Note: Do not change the name. Leave as `dynatrace-otelcol-dt-api-credentials`

```
kubectl create secret generic dynatrace-otelcol-dt-api-credentials \
--from-literal=DT_ENDPOINT=https://abc12345.live.dynatrace.com/api/v2/otlp \
--from-literal=DT_API_TOKEN=dt0c01.sample.secret
```

You should see this: `secret/dynatrace-otelcol-dt-api-credentials created`

## Add OpenTelemetry Helm charts and Update

Copy and paste the following to add the OpenTelemetry Helm chart and update it to the latest versions.

```
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update
```

## Configure and Install Dynatrace OpenTelemetry Collector

The OpenTelemetry collector requires a configuration file. This is already available in the environment. See [collector-values.yaml](collector-values.yaml)

You do not need to modify this file.

Install the collector by copy and pasting this content:

```
helm upgrade -i dynatrace-collector open-telemetry/opentelemetry-collector -f collector-values.yaml
```

After Helm indicates it has installed, run the following command and you should see a pod either `Pending` or `Running`.

```
kubectl get pods
```

Wait and periodically re-run `kubectl get pods` until the Pod is `Running`.

## Install OpenTelemetry Demo

Use Helm to install the OpenTelemetry demo system, passing the configuration file (already created for you - see [otel-demo-values.yaml](otel-demo-values.yaml)).

This is a demo website we will use to generate OpenTelemetry data (logs, metrics and traces).

```
helm upgrade -i my-otel-demo open-telemetry/opentelemetry-demo -f otel-demo-values.yaml
```

The Pods may take 2-3 minutes to start, but running `kubectl get pods` should eventually show the pods running.

## Access The Demo User Interface

Note: This step is optional because there is a load generator already running. Observability data will be flowing into Dynatrace.

Expose the user interface on port 8080 by port-forwarding:

```
kubectl -n default port-forward svc/my-otel-demo-frontendproxy 8080:8080
```

Go to the `Ports` tab, right click the port `8080` and choose "Open in Browser".

You should see the OpenTelemetry demo application.

![opentelemetry demo application](.devcontainer/images/otel-demo-app.png)

## Query Observability Data

In Dynatrace, press `Ctrl + k` again and search for `Notebooks`.

Create a new notebook and add a new section exploring the logs.

Add a new section and choose `Query Grail`. This section type allows you to write your own Dynatrace Query Language (DQL) to have full control over the data you retrieve.

Type: `fetch spans` in the DQL box and execute the query.

## Cleanup: Delete Codespace

Go to `https://github.com/codespaces` and delete the codespace.

