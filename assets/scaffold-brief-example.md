# Example ScaffoldBrief

```json
{
  "productName": "Acme Distro",
  "targetDir": "backend",
  "framework": "nestjs",
  "database": "pgsql",
  "storage": "s3",
  "stages": ["S1", "S2", "S3", "S4", "S5", "S6"],
  "packageManager": "npm",
  "auth": "jwt",
  "language": "ts",
  "notes": [
    "S3 via MinIO locally",
    "Skip payments (S3 monetization) in v1 skeleton"
  ]
}
```

`S3` in `stages` means monetization stage id — not Amazon S3. Storage choice is the separate `storage` field.
