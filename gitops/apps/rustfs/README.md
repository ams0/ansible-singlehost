# RustFS - S3-Compatible Object Storage

RustFS is an S3-compatible object storage server written in Rust. It runs in standalone mode on the cluster with a web console and S3 API.

## Endpoints

| Service | URL |
|---------|-----|
| S3 API | `https://rustfs.vps.kubespaces.cloud` |
| Console | `https://rustfs.vps.kubespaces.cloud/rustfs/console/` |

## Authentication

Credentials are managed outside of Flux via a Kubernetes secret (`rustfs-auth` in the `rustfs` namespace), created by the Ansible `flux` role. The vault variables are:

- `vault_rustfs_access_key` - S3 access key (also used for console login)
- `vault_rustfs_secret_key` - S3 secret key (also used for console login)

To update credentials:

```bash
ansible-vault edit group_vars/oracle_hosts/vault.yml
ansible-playbook site.yml --tags flux
```

## Server-Side Encryption (KMS)

RustFS requires a KMS key for server-side encryption. This is configured via `extraEnv` in the HelmRelease:

```yaml
extraEnv:
  - name: RUSTFS_KMS_ENABLE
    value: "true"
  - name: RUSTFS_KMS_BACKEND
    value: "local"
```

The `local` backend generates and stores encryption keys on disk (`/data`). Without this, uploads fail with:

```
InternalError: No KMS key available for managed server-side encryption (required for SSE-KMS)
```

> **Note:** The `extraEnv` field was added to the Helm chart on `main` and may not be available in older chart versions. If it's not supported, add `RUSTFS_KMS_ENABLE` and `RUSTFS_KMS_BACKEND` to the `rustfs-auth` secret instead.

## S3 API Examples

### Create a bucket

```bash
curl -X PUT https://rustfs.vps.kubespaces.cloud/my-bucket \
  -H "Authorization: AWS4-HMAC-SHA256 ..."
```

Using the AWS CLI (easier):

```bash
aws --endpoint-url https://rustfs.vps.kubespaces.cloud \
  s3 mb s3://my-bucket
```

### Upload a file

```bash
aws --endpoint-url https://rustfs.vps.kubespaces.cloud \
  s3 cp my-file.pdf s3://my-bucket/my-file.pdf
```

### List buckets

```bash
aws --endpoint-url https://rustfs.vps.kubespaces.cloud \
  s3 ls
```

### List objects in a bucket

```bash
aws --endpoint-url https://rustfs.vps.kubespaces.cloud \
  s3 ls s3://my-bucket/
```

### Download a file

```bash
aws --endpoint-url https://rustfs.vps.kubespaces.cloud \
  s3 cp s3://my-bucket/my-file.pdf ./my-file.pdf
```

### Delete a file

```bash
aws --endpoint-url https://rustfs.vps.kubespaces.cloud \
  s3 rm s3://my-bucket/my-file.pdf
```

### Using curl directly (presigned URL)

```bash
# Generate a presigned URL (valid 1 hour)
aws --endpoint-url https://rustfs.vps.kubespaces.cloud \
  s3 presign s3://my-bucket/my-file.pdf --expires-in 3600

# Then use the presigned URL with curl
curl -O "<presigned-url>"
```

### AWS CLI configuration

Configure the AWS CLI to use rustfs credentials:

```bash
aws configure --profile rustfs
# AWS Access Key ID: rustfsadmin
# AWS Secret Access Key: <your secret key>
# Default region name: eu-oracle-1
# Default output format: json
```

Then use `--profile rustfs` with all commands, or set:

```bash
export AWS_PROFILE=rustfs
export AWS_ENDPOINT_URL=https://rustfs.vps.kubespaces.cloud
```
