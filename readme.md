# 🗄️ PostgreSQL → S3-Compatible Storage Backup via GitHub Actions

This repository includes an automated **GitHub Actions workflow** that performs daily backups of your PostgreSQL database and uploads them to **any S3-compatible storage** (Supabase, AWS S3, MinIO, Backblaze B2, etc.) in `.dump` format.

---

## ✨ Features

- 🔄 Automatic **daily PostgreSQL backups**
- 💾 Uses **PostgreSQL custom `.dump` format** (smaller, faster restore)
- 🧰 Includes **schema selection** and **migration table exclusion**
- ☁️ Uploads to **any S3-compatible storage** using standard S3 API
- 🧹 Automatically deletes backups **older than 7 days**
- 🔐 Runs securely in **GitHub Actions**, no external servers needed
- 🚀 Uses AWS CLI for reliable, standard S3 operations

---

## 📋 Prerequisites

### For Supabase Storage

1. Go to your Supabase Dashboard
2. Navigate to **Settings** → **API**
3. Scroll down to **S3 Access Keys** section
4. Click **Generate new key**
5. Copy both the **Access Key ID** and **Secret Access Key**

**S3 Endpoint for Supabase:**

```
https://[your-project-ref].supabase.co/storage/v1/s3
```

Replace `[your-project-ref]` with your actual project reference (found in your project URL).

### For AWS S3

Use your AWS IAM credentials and the standard AWS S3 endpoint:

```
https://s3.amazonaws.com
```

### For Other S3-Compatible Storage

- **MinIO**: `https://your-minio-server:9000`
- **Backblaze B2**: `https://s3.us-west-004.backblazeb2.com`
- **DigitalOcean Spaces**: `https://[region].digitaloceanspaces.com`

---

## 🔧 Setup

### Add GitHub Secrets

In your repo, go to **Settings → Secrets → Actions** and add the following:

#### PostgreSQL Connection Secrets

| Secret              | Example          | Description                |
| ------------------- | ---------------- | -------------------------- |
| `PGHOST`            | `db.supabase.co` | PostgreSQL host            |
| `PGPORT`            | `5432`           | PostgreSQL port            |
| `PGDATABASE`        | `postgres`       | Database name              |
| `PGUSER`            | `postgres`       | Database user              |
| `PGPASSWORD`        | `supersecret`    | Database password          |
| `PGSCHEMA`          | `public`         | Schema to include          |
| `PGMIGRATION_TABLE` | `_migrations`    | Migration table to exclude |

#### S3 Storage Secrets

| Secret          | Example                                    | Description                              |
| --------------- | ------------------------------------------ | ---------------------------------------- |
| `S3_ENDPOINT`   | `https://abc123.supabase.co/storage/v1/s3` | S3-compatible endpoint URL               |
| `S3_REGION`     | `us-east-1`                                | S3 region (use `us-east-1` for Supabase) |
| `S3_BUCKET`     | `db_backups`                               | Bucket name                              |
| `S3_ACCESS_KEY` | `abc123xyz...`                             | S3 Access Key ID                         |
| `S3_SECRET_KEY` | `secret123...`                             | S3 Secret Access Key                     |
