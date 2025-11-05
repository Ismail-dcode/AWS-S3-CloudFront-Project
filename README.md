# 🌐 AWS S3 + CloudFront Static Website (Free Tier Friendly)

Host a secure, long-lived **static website** on **Amazon S3** with **CloudFront** for HTTPS and global CDN caching — **without making your S3 bucket public**.  
All steps use the **AWS Console only** (no Terraform), and the setup stays **within AWS Free Tier limits** for as long as possible.

---

## 🚀 Live Demo

🔗 **Live Website:** [View on CloudFront]( d2bhhd6ys9k0o4.cloudfront.net)

*(Hosted using AWS S3 + CloudFront)*


## 📋 Table of Contents
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Free-Tier & Costs](#free-tier--costs)
- [Step-by-Step (Console)](#step-by-step-console)
- [Optional: SPA Routing](#optional-single-page-app-routing)
- [Recommended Cache Headers](#recommended-cache-headers)
- [Update Workflow](#update-workflow)
- [Troubleshooting](#troubleshooting)
- [Clean Up](#clean-up)
- [FAQ](#faq)
- [License](#license)

---

## 🏗 Architecture

![Architecture Diagram: CloudFront + OAC to Private S3](/architecture-dia.jpg?raw=true.png "Architecture: CloudFront and S3 setup")

> Users access CloudFront (HTTPS), which fetches files from a **private S3 bucket** through **Origin Access Control (OAC)**.  
> The S3 bucket is never made public.

---

## ✨ Features
- 🔒 **Private S3 bucket** (no public ACLs or bucket policies)
- 🌍 **Global CDN** with **HTTPS** using CloudFront
- 💸 **Free-tier friendly** — minimal cost and safe defaults
- 🧰 **Console-only setup** — beginner-friendly, no CLI required
- ⚡ Optional **SPA (React/Vue)** routing support
- 🪶 Lightweight and long-lasting

---

## 🧰 Prerequisites
- An **AWS account**
- Basic static website (e.g., `index.html`, CSS, JS, images)


---

## 💸 Free-Tier & Costs

| Service | Free Tier | Duration | Notes |
|----------|------------|-----------|--------|
| **S3** | 5 GB storage, 20k GET, 2k PUT, 15 GB transfer | 12 months | Private bucket only |
| **CloudFront** | 1 TB data out, 10M requests/month | **Always free** | Global CDN |
| **AWS Budgets** | Free | Forever | For cost alerts |

> ⚠️ After 12 months, S3 may cost a few rupees per month depending on usage. CloudFront’s “Always Free” tier remains active.

---

## 🪜 Step-by-Step (Console)

### **1️⃣ Create S3 Bucket**
1. Go to **AWS Console → S3 → Create bucket**  
2. Enter a unique **bucket name** (e.g., `ismail-static-site`)  
3. Choose **Region:** Asia Pacific (Mumbai)  
4. Leave **Block Public Access** **ON** ✅  
5. Disable versioning, logging, encryption (keep free-tier light)  
6. Click **Create bucket**

---

### **2️⃣ Upload Your Website Files**
1. Open the bucket → **Upload** → add your files (`index.html`, CSS, JS, images).  
2. Click **Upload** — no permission changes needed.

📝 Ensure your main file is named **index.html** at the root.

---

### **3️⃣ Create Origin Access Control (OAC)**
1. Open **CloudFront Console** → **Origin Access** → **Create control setting**.  
2. Name it (e.g., `oac-s3-static-site`)  
3. Type = **S3**  
4. Click **Create**

---

### **4️⃣ Create a CloudFront Distribution**
1. Go to **CloudFront → Create Distribution**
2. **Origin domain:** choose your S3 bucket (ending with `.s3.ap-south-1.amazonaws.com`)
3. **Origin access:**  
   - Select **Origin Access Control (recommended)**  
   - Choose your OAC  
   - Allow CloudFront to update the bucket policy automatically (if prompted)
4. **Default root object:** `index.html`
5. **Viewer protocol policy:** `Redirect HTTP to HTTPS`
6. **Cache policy:** `CachingOptimized`
7. **Price class:** `Price Class 100` (low-cost)
8. **SSL certificate:** `Default CloudFront certificate (*.cloudfront.net)`
9. Click **Create distribution** and wait until status is **Deployed**

---

### **5️⃣ Verify or Add S3 Bucket Policy**
If CloudFront didn’t add it automatically, go to:

**S3 → Your bucket → Permissions → Bucket Policy**  
Add this (replace placeholders):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontReadOnlyAccess",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<YOUR-BUCKET-NAME>/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<YOUR-AWS-ACCOUNT-ID>:distribution/<YOUR-DISTRIBUTION-ID>"
        }
      }
    }
  ]
}

