# Deploy Static Website with S3 + CloudFront + Cloudflare

This project demonstrates how to deploy a static website using AWS S3 for storage, CloudFront for CDN, and Cloudflare for DNS and security.

---

## Architecture Overview

<div align="center">
  <img src="Static%20Website%20with%20S3%20%2B%20CloudFront%20%2B%20Cloudflare.drawio.svg" alt="Architecture Diagram" width="800"/>
</div>

- **S3 Bucket:** Stores static website files (HTML, CSS, JS, images, PDFs).
- **CloudFront:** Distributes content globally, provides HTTPS, and caches content for performance.
- **Cloudflare:** Manages DNS, provides additional CDN/security features, and handles your custom domain.

---

## Step-by-Step Deployment

### 1. Configure the S3 Bucket for Static Website Hosting

#### 1.1 Enable Static Website Hosting
- Go to AWS S3 Console, select your bucket.
- Go to **Properties** → **Static website hosting** → **Edit**.
- Enable it, set `index.html` as the index document.
- Note the S3 website endpoint (e.g., `http://your-bucket.s3-website-us-east-1.amazonaws.com/`).

#### 1.2 Configure Bucket Permissions
- Update the bucket policy to allow public read access:
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "PublicReadGetObject",
        "Effect": "Allow",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::your-bucket-name/*"
      }
    ]
  }
  ```
- Go to **Permissions** → **Block public access**: Disable "Block all public access".
- Save changes.

#### 1.3 Test S3 Website
- Visit the S3 website endpoint to ensure your site loads.

---

### 2. Set Up CloudFront Distribution

#### 2.1 Create a Distribution
- Open CloudFront Console → **Create Distribution**.

#### 2.2 Configure Origin Settings
- **Origin Domain:** Use the S3 website endpoint (not the bucket name).
- **Origin Path:** Leave blank unless your files are in a subdirectory.
- **Origin Access:** Public or OAI/OAC (see optional step 4).

#### 2.3 Cache Behavior Settings
- **Viewer Protocol Policy:** Redirect HTTP to HTTPS.
- **Allowed HTTP Methods:** GET, HEAD.
- **Cache Policy:** `CachingOptimized`.
- **Origin Request Policy:** `Managed-CORS-S3Origin` (if using CORS).

#### 2.4 Distribution Settings
- **Alternate Domain Name (CNAME):** e.g., `https:www.test.domain.com`
- **SSL Certificate:** Request a public certificate in ACM (use DNS validation).
- **Default Root Object:** `index.html`
- **Price Class:** Choose as needed.

#### 2.5 Create and Deploy
- Click **Create Distribution** and wait for deployment.
- Note the CloudFront domain name (e.g., `d1234567890abcdef.cloudfront.net`).

---

### 3. Set Up DNS with Cloudflare

#### 3.1 Log In to Cloudflare
- Access your domain from the Cloudflare Dashboard.

#### 3.2 Add a CNAME Record
- Go to **DNS** → **Records** → **Add Record**:
  - **Type:** CNAME
  - **Name:** `www.test`
  - **Target:** CloudFront domain name
  - **Proxy Status:** DNS only (not proxied) while validating SSL

#### 3.3 Validate SSL Certificate
- Wait for ACM to show the certificate as **Issued**.
- Optionally, change Proxy Status to **Proxied** in Cloudflare for extra features.

---

### 4. (Optional) Secure Access with OAI (Origin Access Identity)
- In CloudFront, go to **Origins** → **Edit** → **Set Origin Access to Origin Access Identity (OAI)**.
- Create a new OAI.
- Set the bucket policy automatically using CloudFront.
- Disable public access in the bucket and remove the public-read policy.

---

### 5. Final Testing

- Visit your custom domain (e.g., `https:www.test.domain.com`).
- Confirm:
  - Fast loading via CloudFront
  - HTTPS is working
  - All resources load correctly

---

## Notes

- For production, always use HTTPS and restrict S3 access using OAI/OAC.
- Cloudflare can provide additional security (DDoS protection, WAF, etc.).
- Use versioning and backups for your S3 bucket. 
