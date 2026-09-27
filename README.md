# octop-docker

Automated **multi-arch** Docker Hub images for
[TencentCloud/Octop](https://github.com/TencentCloud/Octop).

Upstream's official workflow only builds `linux/amd64`. This repo additionally
builds `linux/arm64` and publishes to Docker Hub, so ARM devices (Raspberry Pi,
ARM servers, Apple Silicon, ...) can run it directly.

> Unofficial build: the image content is identical to upstream's
> `docker/Dockerfile`; only the arm64 architecture is added.

## Pull

```bash
docker pull <your-dockerhub-username>/octop:latest
```

A specific version:

```bash
docker pull <your-dockerhub-username>/octop:1.0.2
```

## Run

```bash
docker run -d \
  --name octop \
  -p 8088:8088 \
  -v octop-data:/data/.octop \
  -e HOME=/data \
  <your-dockerhub-username>/octop:latest
```

Open `http://localhost:8088`. On first boot the container runs `octop init`
automatically; if `OCTOP_DEFAULT_PASSWORD` is unset, a random password is
generated and written to `/data/.octop/credential.txt`. See the
[upstream docker docs](https://github.com/TencentCloud/Octop/tree/main/docker)
for all environment variables.

## Tags

| Tag     | Description                          |
| ------- | ------------------------------------ |
| `latest`| Build of the latest upstream release |
| `x.y.z` | Build of upstream tag `vx.y.z`       |

## How it works

1. Runs on schedule every Monday 09:00 MDT (15:00 UTC); can also be triggered manually.
2. Queries the latest upstream GitHub Release and compares it with the `LAST_BUILD`
   file in this repo.
3. Only builds when a new version is found: checks out upstream at that tag and
   builds `linux/amd64,linux/arm64` with upstream's own `docker/Dockerfile`,
   then pushes to Docker Hub.
4. On success, writes the new version back to `LAST_BUILD` so the next run skips it.

## Setup

1. Create an empty **public** repo `top/octop-docker` on GitHub and push these files.
   (Public repos get unlimited free Actions minutes; a private repo's free quota
   can be eaten up by the long multi-arch builds.)
2. Docker Hub > Account Settings > Security > Personal access tokens > Generate
   new token, with **Read & Write** permissions.
3. GitHub repo > Settings > Secrets and variables > Actions > New repository secret:
   - `DOCKERHUB_USERNAME` — your Docker Hub username (also the image namespace)
   - `DOCKERHUB_TOKEN` — the access token from step 2
4. Go to the Actions tab and trigger a manual run once to verify everything works.

## Manual build

1. Open the repo page > **Actions** tab.
2. Select **Build multi-arch images** in the left sidebar.
3. Click **Run workflow** (top right). Optional inputs:
   - `tag` — build a specific upstream tag, e.g. `v1.0.2` (empty = latest release)
   - `force` — rebuild even if the version was already built
4. Click the green **Run workflow** button.

Note: the arm64 leg builds under QEMU emulation, so a full run can take tens of
minutes. That is expected for a weekly job.
