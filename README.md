<p align="center">
  <img width="100" src="https://avatars.githubusercontent.com/u/57727799?s=200&v=4" style="vertical-align: middle; margin: 0 1.5rem" />
  <img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="60" style="vertical-align: middle;" />
</p>


# Revolt Helm Chart

This chart provides a means of deploying Revolt to kubernetes.

---

# Minimal Setup

To use the minimal setup, you will require

- A working kubernetes cluster
- Persistent storage for MongoDB, Redis, MinIO, and RabbitMQ
- A valid hostname and the ability to access it via HTTPS (such as [cert-manager](https://cert-manager.io/docs/))

1. Generate required config keys.  We provide a script to run in docker to generate it in this repo.
   ```shell
   docker run --entrypoint /bin/ash -v ./:/data alpine/openssl /data/generate_config.sh
   ```
2. Fill out required config
    ```yaml
    global:
      namespace: 'revolt'
      domain: 'revolt.example.com'
      ingress:
        enabled: true
        className: nginx
        annotations:
          # Annotations may differ for other ingress controllers.  Consult your documentation.
          nginx.ingress.kubernetes.io/rewrite-target: /$2
          nginx.ingress.kubernetes.io/force-ssl-redirect: 'true'
          nginx.ingress.kubernetes.io/proxy-body-size: '0'
      secret:
        vapid_key: ''
        vapid_public_key: ''
        encryption_key: ''
    ```
3. Run `helm install ./ revolt -f my_values.yaml`
4. Once it's done setting itself, up, access it at your external URL.  It may take a few minutes to spin up from scratch.
    
Congrats, you have a minimal working setup. This is NOT production ready however.
- There is no persistence is enabled by default, so everything is lost on restart.
- The external connections such as MongoDB and Redis have no authentication.

## Persistence

Persistence is handled by the subcharts.  Consult the subcharts for more information.

- [MongoDB](https://github.com/bitnami/charts/tree/main/bitnami/mongodb#persistence)
- [Redis](https://github.com/bitnami/charts/tree/main/bitnami/redis#persistence)
- [MinIO](https://github.com/bitnami/charts/tree/main/bitnami/minio#persistence)
- [RabbitMQ](https://github.com/bitnami/charts/tree/main/bitnami/rabbitmq#persistence)

## External Subcharts

The subcharts all support external connections, with the option to disable the built-in chart.
```yaml
global:
  subcharts:
    # All but rabbitmq are full connection+auth strings.
    mongo:  # mongodb
      enabled: true  # Set to false to use external MongoDB
      connection_url: ''
    redis:
      enabled: true  # Set to false to use external Redis
      connection_url: ''
    minio:
      enabled: true  # Set to false to use external MinIO
      connection_url: ''
    rabbitmq:
      enabled: true  # Set to false to use external RabbitMQ
      host: ''
      port: 5672
      username: 'rabbituser'
      password: 'rabbitpass'
```


---


# Full Configuration

## Global Settings

| config                            | description                                            | default                |
|-----------------------------------|--------------------------------------------------------|------------------------|
| global.namespace                  | Namespace for Revolt                                  | `'Revolt'`            |
| global.domain                     | Domain name used for access (e.g. Revolt.example.com) | `''`                   |
| global.https                      | Enables HTTPS for external connections                 | `false`                |
| global.auth.backend.audience      | Backend OAuth2 audience                                | `''`                   |
| global.auth.backend.extra_scopes  | Backend extra OAuth2 scopes                            | `'offline_access api'` |
| global.dashboard.port             | Dashboard HTTP port                                    | `80`                   |

## Component Specific Settings

| config                                 | description                      | default                  |
|----------------------------------------|----------------------------------|--------------------------|
| dashboard.image.repository             | Dashboard image repository       | `'Revoltio/dashboard'`  |
| dashboard.image.tag                    | Dashboard image tag              | `'v2.14.0'`              |
| dashboard.image.pullPolicy             | Image pull policy                | `'IfNotPresent'`         |
| dashboard.annotations                  | Pod annotations                  | `{}`                     |
| dashboard.labels                       | Pod labels                       | `{}`                     |
| dashboard.nodeSelector                 | Node selector                    | `{}`                     |
| dashboard.tolerations                  | Tolerations array                | `[]`                     |
