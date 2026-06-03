# kallumcooper-site

Personal portfolio website built with **Hugo** using the **hugo-profile** theme, and deployed to **AWS (S3 + CloudFront)**.

This repo contains the site source (content + config). Infrastructure is provisioned separately via Terraform.

---

## What this is

- **Static site generator:** Hugo
- **Theme:** hugo-profile
- **Hosting:** Private S3 bucket behind CloudFront
- **DNS + TLS:** Route 53 + ACM (CloudFront cert in us-east-1)
- **Deployment:** GitHub Actions builds Hugo, syncs the `public/` output to S3, then invalidates CloudFront

---

## Related repositories (this site depends on these)

- **Terraform module library:** [terraform-modules](https://github.com/kallum-cooper/terraform-modules/tree/main)  
  Reusable Terraform modules (e.g. S3 private origin bucket, CloudFront static site, etc.)

- **Infrastructure environment repo:** [kallumcooper-infra](https://github.com/kallum-cooper/kallumcooper-infra)  
  Composes the module library to provision the production AWS environment (Route 53, ACM, CloudFront, S3).

---

## Local development

From the repo root:

```bash
hugo server -D --config ./hugo.yaml
```

---

## Deployment workflow

The site is deployed by the GitHub Actions workflow in `.github/workflows/deploy.yml`.

The workflow runs on pushes to `main` or `master`:

```yaml
on:
  push:
    branches: ["master", "main"]
```

This means the normal update flow is:

1. Create a local branch for the site change.
2. Edit the Hugo source files, such as `hugo.yaml`, `content/`, `layouts/`, or `static/`.
3. Test locally with `hugo server -D --config ./hugo.yaml`.
4. Push the branch to GitHub.
5. Open a pull request into `main`.
6. Merge the pull request.

When the pull request is merged, GitHub creates a push event on `main`. That push triggers the workflow, which:

1. Checks out the repository, including submodules.
2. Installs Hugo.
3. Cleans and rebuilds the static site into `public/`.
4. Configures AWS credentials from GitHub repository secrets.
5. Syncs the generated `public/` files to the S3 bucket.
6. Invalidates the CloudFront distribution so the live site serves the new version.

The generated `public/` directory should not be edited directly. Make content and configuration changes in the Hugo source files, then let the workflow rebuild and publish the static output.
