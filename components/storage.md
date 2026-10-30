# File storage

Storing user uploads and generated files (images, PDFs, exports) outside the database. Databases hold the file's metadata and URL; object storage holds the bytes.

## Decide

### Questions to ask

- Who uploads (users, or only the app)? How large are files?
- Public files (avatars, images on pages) or private (documents, receipts)?
- How much download traffic? (Egress fees can dominate cost.)
- Do you need image resizing/optimization?
- What do you want to learn: the S3 API (the industry standard), or the quickest managed upload?

### Options

| Option | Tier | Best for | Main trade-off |
| :- | :- | :- | :- |
| Vercel Blob | ★ default | Simple uploads in Vercel apps | Vercel-specific; usage pricing |
| Cloudflare R2 | ◆ alternative | S3-compatible storage with no egress fees | Separate account and setup |
| AWS S3 | ◆ alternative | The standard; any scale; most features | IAM complexity; egress fees |
| Supabase Storage | ◆ alternative | Supabase projects; RLS-style policies on files | Tied to Supabase |
| UploadThing | ✦ distinctive | Typed upload routes with prebuilt components | Another service in the middle |

### Option notes

**Vercel Blob** ★
- Pros: a few lines to upload from server or client; integrated env var; public or private blobs.
- Cons: Vercel-only; check pricing for heavy traffic.
- What you'll learn: object storage basics, client uploads with tokens.

**Cloudflare R2** ◆
- Pros: S3-compatible API (same SDK and skills as S3); no egress fees; public buckets via custom domain; Workers binding.
- What you'll learn: the S3 API, presigned URLs, CORS on buckets.

**AWS S3** ◆
- Pros: the reference implementation; lifecycle rules, versioning, events; everything integrates with it.
- Cons: IAM policies and bucket policies take learning; egress costs.
- What you'll learn: IAM, bucket policies, presigned URLs, AWS in general.

**Supabase Storage** ◆
- Pros: buckets with access policies tied to Supabase Auth; image transformations.
- What you'll learn: policy-based file access.

**UploadThing** ✦
- Pros: define typed "file routes" with size/type limits and auth; drop-in React components.
- Cons: middle-man service; built on top of object storage.
- What you'll learn: typed upload pipelines.

### How to choose

1. On Vercel, simple needs → Vercel Blob.
2. Lots of downloads or want S3 skills cheaply → R2.
3. Learning AWS or need advanced features → S3.
4. Supabase project → Supabase Storage.

## Configure

### Common to every option

- [ ] Validate file type and size on the server (don't trust the client's MIME type).
- [ ] Upload large files directly from the browser to storage via presigned URLs or client tokens; don't stream them through serverless functions.
- [ ] Store the object key/URL and owner in the database.
- [ ] Private files: serve via short-lived signed URLs after an auth check.
- [ ] Decide what happens to files when the owning record is deleted.

### Vercel Blob

**Setup**
- [ ] Vercel dashboard → **Storage → Create → Blob**, connect to the project (adds `BLOB_READ_WRITE_TOKEN`).
- [ ] `npm i @vercel/blob`; `put()` on the server, or `upload()` from the client with a token route (`handleUpload`).
- [ ] `vercel env pull` to get the token locally.

**Verify**
- [ ] Upload a file and load it from the returned URL.

### Cloudflare R2

**Setup**
- [ ] Create a bucket; create an R2 API token (Access Key ID + Secret) scoped to it.
- [ ] Use `@aws-sdk/client-s3` with `endpoint: https://<account-id>.r2.cloudflarestorage.com` and `region: "auto"`.
- [ ] Presigned uploads with `@aws-sdk/s3-request-presigner`.
- [ ] Configure bucket CORS for browser uploads.
- [ ] Public files: connect a custom domain to the bucket.

**Env vars**
- `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET`.

**Verify**
- [ ] Presigned browser upload succeeds; public URL serves the file.

### AWS S3

**Setup**
- [ ] Create a bucket (Block Public Access on unless intentionally public).
- [ ] IAM user or role with a least-privilege policy for that bucket only.
- [ ] `@aws-sdk/client-s3`; presigned PUT for uploads, presigned GET for private reads.
- [ ] Bucket CORS for browser uploads.

**Env vars**
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, `S3_BUCKET`.

**Verify**
- [ ] Credentials can't access other buckets; uploads and signed reads work.

### Supabase Storage

**Setup**
- [ ] Create a bucket (public or private); add storage policies.
- [ ] `supabase.storage.from(bucket).upload(...)`; `createSignedUrl` for private files.

**Verify**
- [ ] Another user cannot read a private file.

### UploadThing

**Setup**
- [ ] Create an app; `npm i uploadthing @uploadthing/react`.
- [ ] Define a file router with limits and auth middleware; mount the route handler; use the components.

**Env vars**
- `UPLOADTHING_TOKEN`.

**Verify**
- [ ] Oversized or wrong-type files are rejected; `onUploadComplete` stores the URL.

## Related

- [`database.md`](database.md), [`auth.md`](auth.md), [`costs.md`](costs.md).
