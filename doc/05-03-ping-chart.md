# Create your pingalicious chart ...

In this 🥼 Lab we will 🏗️ Build our Helm Chart:

For our simple service a simple `helm create pingalicious` would skafold a default helm chart with the name `pingalicious` ;).


Chart requirements / Instllation Use Cases:

- 1️⃣ redis + api      - no-auth `standard components`

Next Labs / Versions:

- 2️⃣ redis auth + api - with auth - `configmap + secret`
- 3️⃣ disable/enable pinger + poller components (enable only when developing) `Adding additional apps`
- 4️⃣ pinger & poller should not start until they can reach the $API_URL (e.g curl http://api:8080/probe/readiness) - `implement initContainers`


## 5️⃣.3️⃣.1️⃣ redis + api - no-auth - Initial Chart Version

```sh
cd ./deployment/helm/charts/
helm create `pingalicious`
```

When you create a `Helm chart` for the first time, this is the typical structure you will find: 

```yaml
pingalicious/
├── Chart.yaml                  - depdency lock file (more on that in a bit)
├── charts                      - A folder for dependencies (Other charts / library charts)
├── templates                   - All the processesd templates
│   ├── NOTES.txt               - helm installation info notes (which are templated too)
│   ├── _helpers.tpl            - helm templating functions
│   ├── hpa.yaml                - a template for `hpa - horisontal pod autoscaler`
│   ├── deployment.yaml         - a template for `deployment`
│   ├── ingress.yaml            - a template for `ingress`
│   ├── service.yaml            - a template for kubernetes `service` 
│   ├── serviceaccount.yaml     - a template for kubernetes `service account` 
│   └── tests                   - helm tests 
└── values.yaml

3 directories, 9 files
```

Let's add binami/redis as adependency like so:
```yaml
# ">>" appends ">" overrides !
# cat<<EOF>>./deployment/helm/charts/pingalicious/Chart.yaml

dependencies:
  - name: redis
    repository: https://charts.bitnami.com/bitnami
    version: 15.5.5
    condition: redis.enabled
EOF
```

Our full `Chart.yaml` file now has 3 important things to note:

1. `version: 0.1.0` 

    ```sh
    # This is the chart version. This version number should be incremented each time you make changes
    # to the chart and its templates, including the app version.
    # Versions are expected to follow Semantic Versioning (https://semver.org/)
    version: 0.1.0
    ```

2. `appVersion: "1.16.0"` 

    ```sh
    # This is the version number of the application being deployed. This version number should be
    # incremented each time you make changes to the application. Versions are not expected to
    # follow Semantic Versioning. They should reflect the version the application is using.
    # It is recommended to use it with quotes.
    appVersion: "1.16.0"
    ```

2. dependencies - condition
   well, version & condition which means we can turn it on and off ...
    ```yaml
    dependencies:
    - name: redis
      repository: https://charts.bitnami.com/bitnami
      version: 15.5.5
      condition: redis.enabled
    ```

🔨 - Get chart depdendecies by running `helm depdendency build` 

- Should yield:

```sh
helm dependency build ./deployment/helm/charts/pingalicious/
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository 
...
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading redis from repo https://charts.bitnami.com/bitnami
Deleting outdated charts
```

> ❗ **if** you get an error like the following:  
>  `Error: no repository definition for https://charts.bitnami.com/bitnami. Please add the missing repos via 'helm repo add'` 

> Make sure you `helm repo add bitnami https://charts.bitnami.com/bitnami` & run the helmdependency build again ❗


## 5️⃣.3️⃣.2️⃣ redis - noauth

- 5️⃣.3️⃣.2️⃣.1 redis configration - disable auth
- 5️⃣.3️⃣.2️⃣.2 api image + service name
- 5️⃣.3️⃣.2️⃣.3 containerPort 
- 5️⃣.3️⃣.2️⃣.4 set NODE_ENV environment variable


### 5️⃣.3️⃣.2️⃣.1 redis configration - disable auth

As weve seen in the previous 🥼 Lab let's set the redis values for helm in the `pingalicious` helm-chart like the following:

```yaml
redis:
  cluster:
    enabled: false
  image:
    tag: "5.0"
  usePassword: false
```

> Add the `redis:` map to the `values.yaml` file we generated with the helm create command.
> The file should be located at `./deployment/helm/charts/pingalicious/values.yaml`

### 5️⃣.3️⃣.2️⃣.2 api image + service name

To use our correct image + set our fullnameOverride to be api, lets just edit our `values.yaml`

```yaml
image:
  repository: nginx
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""

# set the current chart's name to be api-*
fullnameOverride: api
```

🔨 - Update the `values.yaml`
🔨 - Test it with `helm template <release-name> <path-to-chart>`
    
> please note this `fullnameOverride` is just a good excuse to understand what this value does to the chart ...

### 5️⃣.3️⃣.2️⃣.3 💥 Problem: hardcoded port 

In our `templates/deployment.yaml` we will find that we have the `containerPort` configured to `80` which may be suitable for nginx, we would like it to be `8080`.

```yaml
# deployment.yaml
ports:
  - name: http
    containerPort: 80
    protocol: TCP
```

  🔨 - Add a containerPort: 8080 to your `values.yaml` files
  
  🔨 - Edit the `deployment.yaml` so the containerPort is set the the `containerPort: 8080`


### 5️⃣.3️⃣.2️⃣.4 NODE_ENV environment variable

Add to your `./templates/deployment.yaml` add something like the following: (make it wotk like with the `containerPort: 8080` above but for our `NODE_ENV` to be any String value)

```yaml
# see ./deployment/kustomize/configs/api/deployment.yaml
  containers:
  - image: registry.gitlab.com/tikal-external/academy-public/images/nodejs-ping:latest
    name: nodejs-ping
    env:
      - name: NODE_ENV
        # the name of our configfile ... - json
        value: <your-value>
```

  🔨 - Add a `nodeEnv: kubernetes` to your `values.yaml` files
  🔨 - Edit the `deployment.yaml` so the containerPort is set the the `containerPort: 8080`
  🔨 - Enable `ingress` so we can access our api via localhost


Complete 5️⃣.3️⃣.2️⃣ in order ot test the chart.

### 5️⃣.3️⃣.2️⃣.5 better envVars - more flexible

Environment variables:

```yaml

# deployment.yaml

  env:
    {{- range $name, $value := .Values.envVars }}
    - name: {{ $name }}
      value: {{ quote $value }}
    {{- end }}
```

```yaml
# values.yaml

envVars:
  NODE_ENV: kubernetes
  REDIS_SERVICE_HOST: nso-ping-redis-master

```

#### 🙌 Init Containers - Use Case

💡 `` - `wait for` database before starting backend application ...

e.g.

  until `dig api.$APP_NAMESPACE.svc.cluster.local` don't continue (keep sleeping for 2 sec then try again ...)

  ```yaml
  initContainers:
      - name: backend-init
        image: registry.gitlab.com/tikal-external/academy-public/images/bind-tools
        command: ['sh', '-c', "until dig +short A api.\${APP_NAMESPACE}.svc.cluster.local; do echo waiting for api; sleep 2; done"]
  ```

  we can also map por attributes as environment variables like so:

  ```yaml
    env:
    - name: APP_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
  ```

> ❗ This is how we use `${APP_NAMESPACE}` with the dig command do synamically get the namespace ...

## 5️⃣.3️⃣.3️⃣ - Test initial version

helm install <release-name> ./path/to/chart

should result in curl localhost

---