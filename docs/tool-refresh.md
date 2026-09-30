# Updating the lab tools

Every CLI is pinned in `mise.toml` to a major or major.minor version, and
`mise.lock` records the exact build, download URL and checksum for x86_64 and arm64
Linux. On macOS and Windows the tools run in `.devcontainer/`, which installs from
the same lock.
`make tools` installs those locked builds and runs `mise run setup` (completion,
the `~/.bashrc` block, Helm indexes, `.lab-versions.json`). CI installs Node and
ShellCheck from the same lock.

To move to newer releases, run `make refresh`: `mise upgrade` takes the newest
release within each version in `mise.toml` and rewrites `mise.lock`. Use
`mise upgrade --bump` to raise the versions in `mise.toml` too. Commit both files
together. Run it on Linux or in the devcontainer: on macOS, mise adds a macOS
entry to the lock.

`make refresh` also updates Gitea: it pulls `gitea/gitea:latest` when a
Gitea container already exists. It does not create Gitea on a machine that has
never run `make gitea`. When the image changed, the refresh starts the
replacement against the existing `gitea-data` volume and waits for its health
endpoint. If startup fails, it puts the previous container back.

The commands write `.lab-versions.json`. This gitignored file records the installed
CLI versions, the exact Helm chart versions that installs actually
landed, and known image references or digests. Attach it when reporting an
upstream compatibility problem.

The scheduled `Cluster smoke` GitHub Actions workflow checks the current kind
release against the configured Kubernetes node image once a week. It runs the
same `make up` path with `CNPE_SMOKE=1`, then tests node readiness, scheduling,
cluster DNS, Service routing, and API audit logging. Smoke mode omits the local
registry, cloud-provider-kind, Gateway API, metrics-server, and VPA. It never
starts Gitea, GitOps controllers, the second cluster, or the full platform stack.
