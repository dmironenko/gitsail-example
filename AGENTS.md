# GitSail example

This is a runnable GitOps example for GitSail, not deployment state.

- Check Git status before edits and preserve existing work.
- Keep user documentation in English and Russian. The root README is a language selector; maintain matching `README.en.md` and `README.ru.md` task guides. Keep Go CLI names, YAML keys, and file paths unchanged in both languages. Use the installed `gitsail` command in guides.
- Quote YAML values only when needed to preserve their type or meaning.
- Use two-space YAML indentation and single-line command/healthcheck.test strings.
- Put a blank line before compose blocks. Use sh -c for shell commands.
- Use only build, app, and compose at the top level, in that order; omit unused blocks.
- Keep application settings and secrets under app in the service's native shape;
  render YAML/JSON directly with toYAML/toJSON instead of rebuilding the config in templates.
- Put environment variables with their original names under app.env. Render an env
  file and reference it through compose.services.<service>.env_file.
- Docker settings belong in compose; GitSail options, including enabled, belong
  in compose.x-gitsail. The built-in Compose template does not read app.
- Keep application secrets under app alongside ordinary settings as encrypted ENC[<base64>] values.
- demo.agekey is an intentionally public demo identity; never use it for real secrets.
- Keep real credentials, private keys, and rendered state out of Git.
- Use gitsail secrets encrypt/decrypt with the shared age key.
- Keep host configuration outside this repository; host.yaml.example is a template.
- One app directory is one Compose project; all apps in an installation share a network.
- Prod reads base.yaml; every other environment reads base.yaml plus stage.yaml.
- Missing stage.yaml disables an app outside prod. Keep shared settings in base.yaml.
- Use image tags without digests. Only apps/ingress/nginx publishes ports; scalable apps have no container_name.
- Validate changes with GitSail. Try deployments in a disposable copy using a unique
  installation, separate state directory, and free ports.
- Keep this repository focused on the runnable example; do not add test infrastructure.
- Keep the two README guides aligned with the actual host YAML, app vars, templates,
  ports, cookies, and currently published GitSail release. Link to the matching
  language version of GitSail's recipe/reference pages.
