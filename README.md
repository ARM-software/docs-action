# Arm Docs GitHub Action

This repository contains GitHub Actions for validating and publishing Arm documentation sites with `docs-action`.

Application developers need clear, high quality documentation that helps them succeed on Arm hardware and with Arm tools more quickly. Product documentation should help users complete the task in front of them, without distracting them with unnecessary surrounding context.

This documentation system exists to give product teams a common way to publish documentation that is focused, consistent, and discoverable, while still allowing each product to keep its own identity, structure, and release cadence.

Each product team maintains its own documentation in its own repository and publishes its own independent documentation site. The shared part of the system is not a single combined site, but a common documentation generator and publishing model used across products.

This GitHub action is the entry point for teams to publish their docs to a shared documentation portal under developer.arm.com in a way that is decentralised and easy to use.

Please [read the documentation to get started](https://docs.devplatform.arm.com/docs-action/getting-started/).

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
- the Terraform `paths` value should match `site-id`

### Add the Terraform access

Your GitHub workflow needs an AWS role that is allowed to upload only your docs route.

Add a module like this in the Terraform repository that manages the docs platform:

```hcl
module "product_guides_upload_role" {
  source = "../../../modules/reusable/docs-s3-upload-role"

  github_repository = "your-org/your-docs-repo"
  bucket_name       = module.ai_docs.bucket_name
  paths             = ["product-guides"]

  additional_policy_arns = [
    module.ai_docs.cloudfront_cache_invalidation_role_arn,
  ]
}
```

What each field does:

- `github_repository`: the only repository allowed to assume the role
- `bucket_name`: the S3 bucket that stores published sites
- `paths`: the docs routes this repository may publish to; this should usually contain exactly one value, and it should match `site-id`. These are the only paths this role will have access to publish
- `additional_policy_arns`: optional extra permissions, such as CloudFront invalidation

If you publish to both staging and production, add the equivalent module in both environments and apply Terraform in both places.

The `role-to-assume` value in your publish workflow should be the AWS role ARN created by this Terraform.

Terraform is managed in `https://github.com/Arm-Debug/edge-ai-infra-terraform`.

Relevant PR: `https://github.com/Arm-Debug/edge-ai-infra-terraform/pull/176`

For the focused Terraform walkthrough, see `https://docs.devplatform.arm.com/docs-action/deployment/terraform-access/`.

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

- The installed Debian package version is controlled by maintainer-edited variables in `publish/action.yml`, `validate/action.yml`, and `.github/workflows/ci.yml`.
- CI checks that all of those pinned version values are set before the package install job runs.
