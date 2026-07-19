# Pack Family apt repository

Shared apt repository for the lossless recompressor family:
[packJPG](https://github.com/YadeWira/packJPG),
[packMP3](https://github.com/YadeWira/packMP3),
packMP2, packPNG.

## Usage

```bash
curl -fsSL https://YadeWira.github.io/pack-apt/pack-apt.gpg.asc | \
  sudo gpg --dearmor -o /usr/share/keyrings/pack-apt.gpg

echo "deb [signed-by=/usr/share/keyrings/pack-apt.gpg] https://YadeWira.github.io/pack-apt stable main" | \
  sudo tee /etc/apt/sources.list.d/pack-apt.list

sudo apt update
sudo apt install packjpg   # or packmp3, packmp2, packpng, once published
```

## How it's fed

Each member project's own release workflow fires a `repository_dispatch`
to this repo when it publishes a stable (non-prerelease) GitHub release
with a `.deb` asset:

```bash
gh api repos/YadeWira/pack-apt/dispatches \
  -f event_type=deb-release \
  -f client_payload[repo]=YadeWira/<project> \
  -f client_payload[tag]=<release-tag>
```

This repo's own workflow (`.github/workflows/publish.yml`, on the `main`
branch) then downloads that `.deb`, adds it to `pool/`, regenerates the
`Packages`/`Release`/`InRelease` index, signs with the dedicated GPG key
(kept only in this repo's secrets — not duplicated per project), and
publishes to `gh-pages`.

Triggering repos need a `PACK_APT_TOKEN` secret (a PAT with `repo` scope,
just for dispatching here — `GITHUB_TOKEN` can't trigger workflows in a
different repository).
