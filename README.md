# al233fl.org

Static website for American Legion Post 233, Palm Valley, Florida.

## Repository layout

- `site/`: deployable website, including all HTML pages.
- `site/index_html_files/`: Xara-exported styles, scripts, images, fonts, and PDFs.
- `.github/workflows/`: reserved for the future GitHub Actions deployment workflow.

Keep HTML pages and `index_html_files/` together when updating or exporting from
Xara. Their relative paths preserve existing page links and asset references.
No build step is required.

## Deployment

Upload the contents of `site/` to the S3 bucket root, not the repository root or
a `site/` prefix. Repository documentation and configuration are not public assets.

The future GitHub Actions workflow should use AWS OIDC authentication and deploy
to a dedicated S3 bucket behind CloudFront. No deployment workflow is configured
yet. Git does not track the empty `.github/workflows/` directory; it will be
included once a workflow file is added.
