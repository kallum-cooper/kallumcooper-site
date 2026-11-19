---
title: "How this site works"
description: "A Hugo frontend, reusable Terraform modules, and a small AWS deployment pipeline — with a serverless visitor counter alongside the static site."
layout: portfolio
eyebrow: "Behind the portfolio"
---

## Three repositories, one website

The website is also an infrastructure project. Its source, production configuration, and reusable building blocks live in separate repositories.

| Repository | Responsibility |
| --- | --- |
| [kallumcooper-site](https://github.com/kallum-cooper/kallumcooper-site) | Hugo configuration, content, theme overrides, images, and the website deployment workflow. |
| [kallumcooper-infra](https://github.com/kallum-cooper/kallumcooper-infra) | The production Terraform stack: DNS, certificates, hosting, bucket policy, and visitor counter integration. |
| [terraform-modules](https://github.com/kallum-cooper/terraform-modules) | Reusable modules for the private S3 origin, CloudFront distribution, and visitor counter. |

The production stack consumes the S3 and CloudFront modules at `v0.1.1`. The counter currently follows the module repository's `main` branch. Editing a local module checkout therefore does not automatically change production.

## Serving a page

![Hosting architecture: Route 53 resolves the domain to CloudFront. The visitor connects over HTTPS to CloudFront, which reads site files from private S3 through Origin Access Control. ACM supplies the certificate.](/images/architecture/site-hosting.svg)

Route 53 provides the DNS records for the apex domain and `www`. Both point to CloudFront. An ACM certificate in `us-east-1` supplies TLS for the distribution, and CloudFront redirects HTTP requests to HTTPS.

The website files sit in a private S3 bucket in `eu-west-2`. CloudFront signs origin requests using Origin Access Control. The bucket policy grants read access to the specific distribution, while S3 public access is blocked. S3 uses AES256 server-side encryption.

Hugo generates the pages ahead of time, so serving the portfolio does not require an application server or database query. CloudFront can cache the generated files at the edge.

## From a change to a deployment

![Deployment architecture: a push to main or master starts GitHub Actions, which builds Hugo, synchronises the generated files to S3, then invalidates CloudFront.](/images/architecture/site-deployment.svg)

The website workflow runs on pushes to `main` or `master`. It checks out the theme submodules, installs Hugo Extended `0.152.2`, removes the old build output, and builds the site from `hugo.yaml`.

GitHub repository secrets supply AWS credentials, the destination bucket, and the distribution ID. The workflow syncs `public/` to S3 with `--delete`, then requests a CloudFront invalidation for `/*`.

Website deployment and infrastructure provisioning are separate. The site workflow publishes static files; it does not run Terraform. The production infrastructure is configured to use the `kallumcooper` AWS profile and currently has no remote backend configured.

## The visitor counter

![Visitor counter architecture: browser JavaScript calls GET /count on an API Gateway HTTP API, which invokes a Python Lambda. Lambda atomically increments DynamoDB and returns a count for the browser to display.](/images/architecture/site-counter.svg)

Once a page loads, a small JavaScript enhancement calls `GET /count` on an API Gateway HTTP API. A Python 3.12 Lambda atomically increments a DynamoDB item using `UpdateItem` and returns the new count as JSON.

DynamoDB uses on-demand billing. The Lambda role is limited to updating its table and writing to its CloudWatch log group, which has a configurable retention period. Browser access is configured for the HTTPS apex origin.

The number represents **counter requests, rather than unique visitors**. If the request fails, the counter hides itself and the page remains usable. It is independent of serving the static site.

## Keeping styling in step with content

The custom stylesheet uses a content hash in its filename. When the CSS changes, Hugo emits a new asset URL and updates the page reference. This prevents a browser's older copy of `site.css` from masking new typography or layout changes after a deployment.

CloudFront invalidation refreshes the CDN's cached files. A versioned asset URL also gives the browser a new resource to request; these address different layers of caching.

## Why this structure

The module library makes the infrastructure pieces reusable. The environment repository supplies the production-specific configuration. The Hugo repository lets content and presentation change through a short build-and-publish workflow.

That separation keeps the site small to operate while giving me a practical place to work on Terraform, AWS, CI/CD, and debugging the path from source to browser.

[Read the Homelab IPAM case study](/case-studies/homelab-ipam/)
