# 🗄️ PostgreSQL → S3 / S3-Compatible Storage Backup via GitHub Actions

This repository includes an automated **GitHub Actions workflow** that performs daily backups of your PostgreSQL database and uploads them to **AWS S3 or any S3-compatible storage** (Supabase, MinIO, DigitalOcean Spaces, Backblaze B2, etc.) in `.dump` format.

---

## ✨ Features

- 🔄 Automatic **scheduled PostgreSQL backups** (4 times daily by default)
- 💾 Uses **PostgreSQL custom `.dump` format** (smaller, faster restore)
- 🧰 Includes **schema selection** and **migration table exclusion**
- ☁️ Supports **AWS S3** and **any S3-compatible storage** using standard S3 API
- 📁 **Optional nested folder structure** with prefix support
- 🧹 **Configurable retention policy** - automatically deletes old backups or keeps all
- 🔐 Runs securely in **GitHub Actions**, no external servers needed
- 🚀 Uses AWS CLI with path-style addressing for maximum compatibility
- ⚙️ **Fully dynamic** - all configuration via GitHub Secrets

---

## 📋 Prerequisites

### For AWS S3

1. Create an IAM user with S3 permissions
2. Generate access keys
3. **Do not set** `S3_ENDPOINT` secret (leave it empty)
4. Use your AWS region (e.g., `us-east-1`)

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

### For Other S3-Compatible Storage

- **MinIO**: `https://your-minio-server:9000`
- **Backblaze B2**: `https://s3.us-west-004.backblazeb2.com`
- **DigitalOcean Spaces**: `https://[region].digitaloceanspaces.com`
- **Wasabi**: `https://s3.[region].wasabisys.com`

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

#### S3 Storage Secrets (Required)

| Secret          | Example                                    | Description                                                     |
| --------------- | ------------------------------------------ | --------------------------------------------------------------- |
| `S3_BUCKET`     | `db_backups`                               | Bucket name                                                     |
| `S3_REGION`     | `us-east-1`                                | S3 region                                                       |
| `S3_ACCESS_KEY` | `abc123xyz...`                             | S3 Access Key ID                                                |
| `S3_SECRET_KEY` | `secret123...`                             | S3 Secret Access Key                                            |
| `S3_ENDPOINT`   | `https://abc123.supabase.co/storage/v1/s3` | S3 endpoint URL (leave empty for AWS S3, set for S3-compatible) |

#### S3 Storage Secrets (Optional)

| Secret           | Example            | Description                                                                  |
| ---------------- | ------------------ | ---------------------------------------------------------------------------- |
| `S3_PREFIX`      | `backups/postgres` | Folder path in bucket (leave empty to store in root)                         |
| `RETENTION_DAYS` | `7`                | Days to keep backups (leave empty or set to `0` to keep all backups forever) |

---

## 🚀 Usage Examples

### Example 1: AWS S3 with 30-day retention

```
S3_ENDPOINT: (not set or empty)
S3_REGION: us-east-1
S3_BUCKET: my-db-backups
S3_PREFIX: production/postgres
RETENTION_DAYS: 30
```

**Result:** Uploads to AWS S3 bucket `my-db-backups` in folder `production/postgres/`, keeps backups for 30 days.

### Example 2: MinIO with no retention (keep all)

```
S3_ENDPOINT: https://minio.example.com:9000
S3_REGION: us-east-1
S3_BUCKET: database-backups
S3_PREFIX: (not set)
RETENTION_DAYS: 0
```

**Result:** Uploads to MinIO bucket root, keeps all backups forever.

### Example 3: DigitalOcean Spaces with 7-day retention

```
S3_ENDPOINT: https://nyc3.digitaloceanspaces.com
S3_REGION: nyc3
S3_BUCKET: my-backups
S3_PREFIX: db/postgres
RETENTION_DAYS: 7
```

**Result:** Uploads to DigitalOcean Spaces in folder `db/postgres/`, keeps backups for 7 days.

### Example 4: Supabase Storage

```
S3_ENDPOINT: https://abc123xyz.supabase.co/storage/v1/s3
S3_REGION: us-east-1
S3_BUCKET: db_backups
S3_PREFIX: postgres/daily
RETENTION_DAYS: 14
```

**Result:** Uploads to Supabase Storage bucket in folder `postgres/daily/`, keeps backups for 14 days.

---

## 🕐 Backup Schedule

By default, backups run **4 times daily** at:

- 4:00 AM UTC
- 8:00 AM UTC
- 12:00 PM UTC
- 4:00 PM UTC

You can also trigger backups manually from the **Actions** tab.

To modify the schedule, edit the cron expressions in `.github/workflows/exec.yml`:

```yaml
on:
  schedule:
    - cron: "0 4 * * *" # 4 AM UTC
    - cron: "0 8 * * *" # 8 AM UTC
    - cron: "0 12 * * *" # 12 PM UTC
    - cron: "0 16 * * *" # 4 PM UTC
```

---

## 📦 Backup File Format

Backups are created in PostgreSQL custom format (`.dump`) with the naming pattern:

```
backup_[database]_[timestamp].dump
```

Example: `backup_postgres_20251109_143052.dump`

This format:

- ✅ Is smaller than SQL text format
- ✅ Supports parallel restore
- ✅ Can restore specific tables/schemas
- ✅ Includes compression

---

## 🔄 Restoring a Backup

To restore a backup, download the `.dump` file and use `pg_restore`:

```bash
pg_restore -h your-host \
  -p 5432 \
  -U your-user \
  -d your-database \
  -v backup_postgres_20251109_143052.dump
```

Or with connection string:

```bash
pg_restore -d "postgresql://user:pass@host:5432/database" \
  -v backup_postgres_20251109_143052.dump
```

---

## 🛠️ How It Works

1. **Install Dependencies**: PostgreSQL 17 client and AWS CLI
2. **Create Backup**: Uses `pg_dump` to create a custom format backup
3. **Configure AWS CLI**: Sets up credentials and path-style addressing (for S3-compatible storage)
4. **Upload to S3**: Uploads backup to specified bucket/prefix
5. **Cleanup Old Backups**: Deletes backups older than retention period (if configured)

---

## 🔒 Security Notes

- All sensitive credentials are stored as **GitHub Secrets**
- Secrets are never exposed in logs or outputs
- Workflow runs in isolated GitHub Actions environment
- Uses official PostgreSQL and AWS CLI tools
- Supports AWS IAM credentials and S3-compatible storage keys

---

## 📝 License

MIT License - Feel free to use and modify!

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

## ⚠️ Important Notes

- Ensure your S3 bucket exists before running the workflow
- For AWS S3, ensure IAM user has appropriate S3 permissions
- Test the workflow manually first using **workflow_dispatch**
- Monitor the **Actions** tab for backup status
- Backups include the specified schema only (default: `public`)
- Migration tables are excluded by default

---

## 📞 Support

If you encounter any issues:

1. Check the **Actions** tab for detailed logs
2. Verify all secrets are correctly set
3. Ensure your S3 credentials have proper permissions
4. For AWS S3, don't set the `S3_ENDPOINT` secret
5. For S3-compatible storage, ensure the endpoint URL is correct

---

**Made with ❤️ for automated database backups**
