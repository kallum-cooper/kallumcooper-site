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
