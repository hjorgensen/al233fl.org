# al233fl.org

Static website for American Legion Post 233, Palm Valley, Florida.

## Repository layout

- `site/`: deployable website, including all HTML pages.
- `site/index_html_files/`: Xara-exported styles, scripts, images, fonts, and PDFs.
- `.github/workflows/`: GitHub Actions production deployment workflow.

Keep HTML pages and `index_html_files/` together when updating or exporting from
Xara. Their relative paths preserve existing page links and asset references.
No build step is required.

## Deployment

Upload the contents of `site/` to the S3 bucket root, not the repository root or
a `site/` prefix. Repository documentation and configuration are not public assets.

The `Deploy Production Site` GitHub Actions workflow deploys after a pull request
is merged into `prod`. Closing a pull request without merging it, or pushing
directly to `prod`, does not trigger deployment. The workflow checks out the latest
`prod` branch so queued deployments do not publish an older merge. Deployments
are serialized to prevent overlapping S3 uploads. No build step is needed.

The workflow runs `aws s3 sync ./site/ s3://BUCKET/ --delete`. This makes the bucket
root mirror `site/`, including removing objects absent from the repository. Use a
dedicated website bucket; do not store unrelated files in it. Uploads are not
atomic, so visitors may briefly see a mix of versions during deployment.

### Repository secrets

In GitHub, open **Settings > Secrets and variables > Actions > Repository secrets**
and create:

| Secret | Value |
| --- | --- |
| `AWS_ACCESS_KEY_ID` | Access key ID for a dedicated AWS IAM deployment user. |
| `AWS_SECRET_ACCESS_KEY` | The corresponding secret access key. |
| `AWS_REGION` | The bucket's AWS region, for example `us-east-1`. |
| `AWS_S3_BUCKET` | The bucket name only, without `s3://` or a path. |

Do not commit credentials. Rotate the access key regularly. Protect `prod` with
pull request reviews, because merged changes can modify the deployment workflow
and access these secrets. AWS OIDC is a future alternative that avoids storing
long-lived access keys; this workflow uses repository secrets as configured above.

### AWS permissions

Create a dedicated IAM user with an access key and attach the following policy,
replacing both occurrences of `YOUR_BUCKET_NAME` with your bucket name. Do not use
root credentials or grant administrator access.

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": ["s3:ListBucket", "s3:GetBucketLocation"],
			"Resource": "arn:aws:s3:::YOUR_BUCKET_NAME"
		},
		{
			"Effect": "Allow",
			"Action": ["s3:PutObject", "s3:DeleteObject"],
			"Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
		}
	]
}
```

This assumes standard S3 encryption. A bucket using a customer-managed KMS key
also requires appropriate KMS permissions and a key policy allowing the user.
The workflow does not change bucket policies, public access settings, or ACLs;
keep the existing website and Cloudflare origin configuration in place.

### Cloudflare caching

No Cloudflare secrets are required for this S3-only deployment. The workflow does
not purge Cloudflare's cache, so cached pages or assets may remain visible until
their cache TTL expires. Purge the cache in Cloudflare after deployment when an
immediate refresh is needed, or configure suitable cache rules.
