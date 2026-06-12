# Arm Docs GitHub Action

This repository contains internal GitHub Actions used by Arm product teams to validate and publish documentation sites to [developer.arm.com](https://developer.arm.com/).

Application developers need clear, high quality documentation that helps them succeed on Arm hardware and with Arm tools more quickly. Product documentation should help users complete the task in front of them, without distracting them with unnecessary surrounding context.

This documentation system exists to give product teams a common way to publish documentation that is focused, consistent, and discoverable, while still allowing each product to keep its own identity, structure, and release cadence.

Each product team maintains its own documentation in its own repository and publishes its own independent documentation site. The shared part of the system is not a single combined site, but a common documentation generator and publishing model used across products.

This GitHub action is the entry point for teams to publish their docs to a shared documentation portal under [developer.arm.com](https://developer.arm.com/) in a way that is decentralised and easy to use. The actions, workflows, and publishing model in this repository are designed for Arm-owned documentation repositories and Arm's internal publishing process.

## Usage

Validate a docs site:

```yaml
- uses: ARM-Software/docs-action/validate@latest
  with:
    docs-root: docs
    site-id: product-guides
```

Publish a docs site:

```yaml
- name: AWS GitHub OIDC Login
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::<account-id>:role/<role-name>
    aws-region: eu-west-1

- uses: ARM-Software/docs-action/publish@latest
  with:
    docs-root: docs
    site-id: product-guides
    environment: staging
```

## Getting Started

`arm-docs` expects a docs root containing Markdown or MDX content, plus optional `static/` assets and a `docs-config.json` file for site-level navigation and labelling.

When you publish:

- `site-id` becomes the route for your docs
- the published site ends up at `<site-url>/<site-id>/`

### Prepare the repository layout

This is a simple repository layout when the docs root lives in `docs/`:

```text
your-docs-repo/
├── .github/
│   └── workflows/
│       ├── validate-docs.yml
│       └── publish-docs.yml
└── docs/
    ├── index.md
    ├── getting-started/
    │   └── install.md
    ├── reference/
    │   └── cli.md
    ├── static/
    │   └── img/
    │       └── architecture.png
    └── docs-config.json
```

The important rules are:

- point `docs-root` at the docs root itself, not at the whole repository unless the docs live at repository root
- place Markdown or MDX files directly in the docs root or its subdirectories
- use `static/` for assets that should be copied into the built site as-is
- include `docs-config.json` to define the site label and top-level navigation for the docs site

If your repository is docs-only, `docs-root` can be `.` instead of `docs`.

## Validate Inputs

- `docs-root`: Docs root inside the repository.
- `site-id`: Explicit site ID for the docs site.

## Publish Inputs

- `docs-root`: Docs root inside the repository.
- `site-id`: Explicit site ID for the docs site.
- `environment`: Deployment environment. Accepted values are `staging` and `production`.

## Package Version

- The installed Debian package version is controlled by `arm-docs-github-action-package-version.txt`.
- That file is intentionally named after the underlying Debian package so it is clearly distinct from this shim action's own version.
- CI checks that this underlying tool package version file is set before the package install job runs.
