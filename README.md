# MegaLinter Custom Flavor: ee-pod-iac-md-py

This custom MegaLinter aims to have an optimized Docker image size.

It is built from official MegaLinter images, but is maintained on [mckraken/megalinter-custom-flavor-iac-md-py](https://github.com/mckraken/megalinter-custom-flavor-iac-md-py) by McKraken

## Embedded linters

- [BICEP_BICEP_LINTER](https://megalinter.io/latest/descriptors/bicep_bicep_linter/)
- [CLOUDFORMATION_CFN_LINT](https://megalinter.io/latest/descriptors/cloudformation_cfn_lint/)
- [JSON_JSONLINT](https://megalinter.io/latest/descriptors/json_jsonlint/)
- [JSON_PRETTIER](https://megalinter.io/latest/descriptors/json_prettier/)
- [MARKDOWN_MARKDOWNLINT](https://megalinter.io/latest/descriptors/markdown_markdownlint/)
- [PYTHON_FLAKE8](https://megalinter.io/latest/descriptors/python_flake8/)
- [PYTHON_PYLINT](https://megalinter.io/latest/descriptors/python_pylint/)
- [PYTHON_RUFF](https://megalinter.io/latest/descriptors/python_ruff/)
- [PYTHON_RUFF_FORMAT](https://megalinter.io/latest/descriptors/python_ruff_format/)
- [REPOSITORY_CHECKOV](https://megalinter.io/latest/descriptors/repository_checkov/)
- [TERRAFORM_TERRAFORM_FMT](https://megalinter.io/latest/descriptors/terraform_terraform_fmt/)
- [TERRAFORM_TFLINT](https://megalinter.io/latest/descriptors/terraform_tflint/)
- [TERRAFORM_TOFU_FMT](https://megalinter.io/latest/descriptors/terraform_tofu_fmt/)
- [TERRAFORM_TOFU_VALIDATE](https://megalinter.io/latest/descriptors/terraform_tofu_validate/)
- [YAML_PRETTIER](https://megalinter.io/latest/descriptors/yaml_prettier/)
- [YAML_YAMLLINT](https://megalinter.io/latest/descriptors/yaml_yamllint/)

## How to use the custom flavor

Follow [MegaLinter installation guide](https://megalinter.io/latest/install-assisted/), and replace related elements in the workflow.

- **GitHub Action**: On MegaLinter step in `.github/workflows/mega-linter.yml`, define `uses: mckraken/megalinter-custom-flavor-iac-md-py@main`
- **Docker image**: Replace official MegaLinter image with `ghcr.io/mckraken/megalinter-custom-flavor-iac-md-py/megalinter-custom-flavor:latest`

## How the flavor is generated and updated

This custom flavor is automatically kept up to date with MegaLinter releases:

1. **Automatic version sync**: The `check-new-megalinter-version` workflow runs daily, checks for new MegaLinter releases, and automatically creates matching releases in this repository.

2. **Automated builds**: Each release triggers the `megalinter-custom-flavor-builder` workflow, which:
   - Builds a Docker image with only the selected linters
   - Publishes to GitHub Container Registry (ghcr.io)
   - Optionally publishes to Docker Hub (if credentials are configured)

3. **Available image tags**:
   - Release tags (e.g., `v9.0.0`): Built from MegaLinter releases
   - `beta` tag: Built from non-main branch pushes for testing
   - `latest` tag: Points to the most recent release

## Configuration requirements

### Optional: Personal Access Token (use with care)

> **Security warning**: Using a Personal Access Token (PAT) is **not recommended**. Open-source projects have been heavily targeted by supply-chain attacks in recent months, and a leaked or compromised PAT can give attackers broad write access to your repository — better safe than sorry!
> If you do not need fully automatic daily version sync, you can skip the PAT entirely and trigger the `check-new-megalinter-version` workflow manually whenever you want to upgrade.

If you decide automatic daily releases are worth the trade-off, configure a `PAT_TOKEN` secret as a **repository-scoped fine-grained token** with:

- **Repository access**: Only select repositories (select this repository)
- **Repository permissions**:
  - Contents: Read and write
  - Actions: Read and write

Rotate the token regularly.

See the [Custom Flavors documentation](https://megalinter.io/beta/custom-flavors/) for detailed setup instructions.

### Optional: Docker Hub publishing

To publish to Docker Hub in addition to ghcr.io, configure:

- `DOCKERHUB_REPO` variable (e.g., your Docker Hub username)
- `DOCKERHUB_USERNAME` secret
- `DOCKERHUB_PASSWORD` secret

## How to generate the flavor manually

If you need to manually trigger a build:

1. **Create a GitHub release**: Creates a versioned build matching the tag name (e.g., `v9.0.0`)
2. **Push to any branch** (except main): Builds a `beta` tagged image for testing
3. **Manually run the workflow**: Go to Actions > Build & Push MegaLinter Custom Flavor > Run workflow

See [full Custom Flavors documentation](https://megalinter.io/beta/custom-flavors/).

## Interim release on a pre-release base (LOCAL TWEAK)

> **This is a local customization, not part of the generated scaffolding.** The
> `megalinter-custom-flavor-builder` workflow has been hand-edited to support it.
> **Re-apply these edits after running `npx mega-linter-runner --custom-flavor-setup`**,
> which regenerates the workflow and overwrites them. The edited regions are
> marked with `LOCAL TWEAK` comments.

Sometimes a linter has merged upstream but is not yet in a stable `vX.Y.0`
MegaLinter image (only the `beta` builder base has it). This lets you publish an
interim flavor release built on that pre-release base, while still tagging it
with a normal semver version so [Dependabot](https://docs.github.com/en/code-security/dependabot)
can track and later bump it in consumer repositories.

Run the builder workflow manually (Actions > Build & Push MegaLinter Custom
Flavor > Run workflow), or via the CLI:

```bash
gh workflow run megalinter-custom-flavor-builder.yml --ref main \
  --field release-version=v10.0.1 \
  --field builder-base=beta
```

- `release-version` — semver tag to publish the image under **and** create as a
  git tag (for Dependabot). Leave empty for a normal build with no tag.
- `builder-base` — builder base image to build against (e.g. `beta`). Leave
  empty to build against the resolved version tag (normal behaviour).

The workflow publishes the image under `release-version` but builds it from the
`builder-base` image, then pushes a **git tag** (not a GitHub Release) for
`release-version`. A bare tag push does not emit a `release` event, so the build
does not re-trigger itself.

Once the upstream stable release that includes the linter ships, the daily
`check-new-megalinter-version` sync builds it on the real base and Dependabot
bumps consumers from the interim tag to the stable one — at which point the
interim tag can be retired.

## License

A MegaLinter custom flavor is still MegaLinter: this repository and the Docker image it publishes are covered by the [AGPL-3.0 license](https://github.com/oxsecurity/megalinter/blob/main/LICENSE).

The published image is built from the official MegaLinter image and bundles MegaLinter, whose source is available at [oxsecurity/megalinter](https://github.com/oxsecurity/megalinter).

[![MegaLinter is provided by OX Security](https://raw.githubusercontent.com/oxsecurity/megalinter/main/docs/assets/images/ox-banner.png)](https://www.ox.security/?ref=megalinter)
