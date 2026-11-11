<p align="right">
  <img align="left" src="https://hits.sh/github.com/dmironenko/gitsail-example.svg?style=flat&amp;label=views&amp;color=007ec6" alt="views" />
</p>
<br clear="left" />

# GitSail example

A runnable GitOps repository for [GitSail](https://github.com/dmironenko/gitsail). It deploys an API built from source, a web app, and nginx with Certbot. Separate nginx instances route production and stage traffic; a shared ingress selects an environment by cookie.

## Choose a task

- [Run the complete example locally](#run-the-example)
- [Test a change in stage and promote it](#make-and-promote-a-change)
- [Add a personal environment](#add-an-environment)
- [Inspect the NGINX and HTTPS settings](apps/ingress/nginx/vars/base.yaml)
- [Find a recipe or a field list](#where-to-look)

## Run the example

Requires Git and Docker Engine or Docker Desktop. Install GitSail v1.0.0:

```sh
curl -fsSL https://raw.githubusercontent.com/dmironenko/gitsail/v1.0.0/install.sh | sh -s -- v1.0.0
```

Then clone and start the example:

```sh
git clone --branch master https://github.com/dmironenko/gitsail-example.git
cd gitsail-example
git branch dev master
cp host.yaml.example ../gitsail.local.yaml
install -m 600 demo.agekey ../gitsail-example.agekey
gitsail apply --config ../gitsail.local.yaml --wait --timeout 15m
```

The first build downloads and compiles the API. The ingress listens on `127.0.0.1:18080` and `127.0.0.1:18443`; ensure the ports are free. Try production and stage:

```sh
curl -ki https://localhost:18443/api/
curl -ki --cookie 'gitsail_stage=gitsail-example:stage' https://localhost:18443/api/
gitsail run --config ../gitsail.local.yaml
```

Responses include `X-GitSail-Environment: prod` and `stage`. `-k` accepts the local development certificate. Ctrl-C stops GitSail; containers continue running. Host paths are relative to `gitsail.local.yaml`. The sample polls local Git every two seconds; use `source.poll_interval: 1m` with a remote repository.

> `demo.agekey` and the `gitsail-example` stage token are public demo values. Replace both before deploying a real service.

## Make and promote a change

Edit a file on `dev`, then commit it:

```sh
git switch dev
# Edit apps/demo/api/vars/base.yaml.
git add apps/demo/api/vars/base.yaml
git commit -m 'feat: update API settings'
```

Check stage, then promote to production:

```sh
git switch master
git merge dev
```

Local mode reads commits without pushing. Remote mode requires a push. Config-only changes are deployed too. Revert the commit to roll back. Change `build.tag` to build another source revision; matching build inputs reuse an image.

## Add an environment

Create a branch and map it in the external host config:

```sh
git branch users/alice dev
```

```yaml
source:
  branches:
    prod: master
    stage: dev
    alice: users/alice
```

Select it with the cookie `gitsail_stage=<stage-token>:alice`, using the token from `app.stage_token`. It reads `base.yaml` and `stage.yaml`, and uses separate projects and volumes; production data is not copied. The service `nginx` is named `nginx-alice` in that environment. Missing `stage.yaml` disables an app outside production. Removing the mapping removes containers and retains volumes/images.

## Where to look

- [GitSail installation guide](https://github.com/dmironenko/gitsail/blob/master/docs/en/recipes/install.md)
- [Create an app](https://github.com/dmironenko/gitsail/blob/master/docs/en/recipes/create-an-app.md)
- [Let apps send requests to each other](https://github.com/dmironenko/gitsail/blob/master/docs/en/recipes/connect-apps.md)
- [Set up stage or a personal environment](https://github.com/dmironenko/gitsail/blob/master/docs/en/recipes/stage.md)
- [Pass environment variables](https://github.com/dmironenko/gitsail/blob/master/docs/en/recipes/environment-variables.md)
- [Add app settings and `x-gitsail` options](https://github.com/dmironenko/gitsail/blob/master/docs/en/recipes/application-config.md)
- [Disable an app or environment](https://github.com/dmironenko/gitsail/blob/master/docs/en/recipes/disable-app.md)
- [Encrypt and use secrets](https://github.com/dmironenko/gitsail/blob/master/docs/en/recipes/secrets.md)
- [Build an app from source](https://github.com/dmironenko/gitsail/blob/master/docs/en/recipes/build-from-source.md)
- [Find all app fields](https://github.com/dmironenko/gitsail/blob/master/docs/en/reference/app-configuration.md)
- [Find all host configuration fields](https://github.com/dmironenko/gitsail/blob/master/docs/en/reference/host-configuration.md)
- [Find CLI commands](https://github.com/dmironenko/gitsail/blob/master/docs/en/reference/cli.md)
- [Find fields available in templates](https://github.com/dmironenko/gitsail/blob/master/docs/en/reference/templates.md)
- [Run GitSail on a server](https://github.com/dmironenko/gitsail/blob/master/docs/en/recipes/server.md)

The `apps/` tree contains production and stage app configs. `host.yaml.example` is a template; copy it outside Git. `demo.agekey` is intentionally public and must never protect real secrets. Only `apps/ingress/nginx` publishes host ports; other apps communicate over the installation network.

## Contribute

Keep the example runnable and its documentation in English and Russian. Keep host configuration and rendered state out of Git. Use two-space YAML indentation and the top-level block order `build`, `app`, `compose`; put app settings and secrets in `app`, Compose settings in `compose`, and GitSail metadata under `compose.x-gitsail`. Validate changes with GitSail using a disposable copy, separate state directory, and free ports.
