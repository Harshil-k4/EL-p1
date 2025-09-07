# Static Website Hosting on AWS S3 with CI/CD & Cloudflare

This project demonstrates how to host a scalable, secure static website using **Amazon S3**, with automated deployment via **GitHub Actions** and optional custom domain management and HTTPS using **Cloudflare**.  
It is designed for 100% free-tier eligibility where possible.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [1. Prerequisites](#1-prerequisites)
- [2. Prepare the Static Web App](#2-prepare-the-static-web-app)
- [3. Create and Configure S3 Bucket](#3-create-and-configure-s3-bucket)
- [4. Website Hosting & Public Permissions](#4-website-hosting--public-permissions)
- [5. Automate Deployment with GitHub Actions](#5-automate-deployment-with-github-actions)
- [6. (Optional) Custom Domain Setup with Cloudflare](#6-optional-custom-domain-setup-with-cloudflare)
- [7. (Optional) Enforce HTTPS with Cloudflare](#7-optional-enforce-https-with-cloudflare)
- [8. Free Tier Notes](#8-free-tier-notes)
- [References](#references)

---

## Overview

This guide walks through the process of:
- Building a static website (e.g. HTML/CSS/JS, such as a ToDo app)
- Hosting it on AWS S3 with public access and website endpoint
- Automating deployments using GitHub Actions
- (Optional) Setting up a free custom domain and global CDN/HTTPS with Cloudflare

---

## Architecture

+--------------+ +-------------------+ +----------------+
| GitHub Repo | ---> | GitHub Actions | ----> | AWS S3 |
| (Static App)| PUSH | (CI/CD Pipeline) | SYNC | (Web Hosting) |
+--------------+ +-------------------+ +----------------+
|
v
[Static Website URL]
|
v
+---------------------+
| Cloudflare CDN | (Optional)
| + Free SSL & DNS |
+---------------------+

text

---

## 1. Prerequisites

- **AWS Account** (Free Tier eligible)
- **GitHub Account**
- **(Optional) Domain**: Free via [Freenom](https://www.freenom.com/) or any registrar
- **Cloudflare Account** (optional, free tier)

---

## 2. Prepare the Static Web App

1. Create a project folder (e.g., `my-static-site`) with at least:
    - `index.html`
    - style/scripts assets (optional)

2. Initialize a GitHub repository and push your code:
    ```
    git init
    git remote add origin <your-repo-url>
    git add .
    git commit -m "Initial commit"
    git push -u origin main
    ```

---

## 3. Create and Configure S3 Bucket

1. **Login to AWS Console > S3 > Create Bucket**
    - Bucket name: must be globally unique (e.g., `my-static-site-hosting`)
    - Region: pick one eligible for Free Tier (e.g., `ap-south-1`)
    - **Uncheck "Block all public access"** (site must be publicly readable)
    - Create bucket

2. **Enable Static Website Hosting**
    - Go to your bucket > Properties > Static website hosting
    - Choose "Enable"
        - Index document: `index.html`
    - Save changes

---

## 4. Website Hosting & Public Permissions

### Public Read Policy
- Go to your bucket > Permissions > Bucket Policy. Add:
    ```
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "PublicReadGET",
          "Effect": "Allow",
          "Principal": "*",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::your-bucket-name/*"
        }
      ]
    }
    ```
    Replace `your-bucket-name` with your actual bucket name.

---

## 5. Automate Deployment with GitHub Actions

### 1. Create AWS User for CI/CD

- IAM > Users > Add User (e.g., `github-actions-user`)
    - Programmatic access
    - Attach `AmazonS3FullAccess` policy
    - Save Access Key ID & Secret

### 2. Add AWS Secrets in GitHub

- In your repo: Settings > Secrets and variables > Actions
- Add:
    - `AWS_ACCESS_KEY_ID`
    - `AWS_SECRET_ACCESS_KEY`

### 3. Add Workflow File

Create `.github/workflows/deploy.yml`:

name: Deploy to S3

on:
push:
branches:
- main

jobs:
deploy:
runs-on: ubuntu-latest

text
steps:
  - uses: actions/checkout@v3

  - name: Configure AWS Credentials
    uses: aws-actions/configure-aws-credentials@v2
    with:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      aws-region: ap-south-1

  - name: Sync S3 bucket
    run: aws s3 sync . s3://your-bucket-name --exclude ".git/*" --delete
text
Replace `your-bucket-name` with your S3 bucket.

### 4. Deploy

- Push changes to `main` branch
- Check Actions tab for progress
- Site will be available at the S3 Website Endpoint (found in bucket > Properties > Static website hosting)

---

## 6.Custom Domain Setup with Cloudflare

- Register a free domain at [Freenom](https://www.freenom.com/) (if available)
- Sign up at [Cloudflare](https://cloudflare.com/) and add your domain
- Update nameservers at Freenom to point to Cloudflare (per instructions shown during setup)
- In Cloudflare DNS:
    - Add a CNAME record:
        - Name: `www`
        - Target: `your-bucket-name.s3-website.<region>.amazonaws.com`
        - Proxy status: Proxied (orange cloud ON)
- Wait for DNS propagation

---

## 7. (Optional) Enforce HTTPS with Cloudflare

- Go to SSL/TLS tab in Cloudflare for your domain
- Set to **Flexible SSL**
- Enable **Always Use HTTPS** in "Edge Certificates"
- After DNS propagates, your site is accessible via `https://yourcustomdomain.com` with HTTPS and CDN

---

## 8. Free Tier Notes

- **AWS S3 Free Tier:** 5GB storage, 20,000 GET requests/month for 12 months
- **GitHub Actions Free Minutes:** 2,000 minutes/month (private repo)
- **Cloudflare Free Plan:** DNS, CDN, SSL, DDoS protection
- **Freenom Free Domain:** `.tk`, `.ml`, `.ga`, `.cf`, `.gq` (if available)

---

## 9. Public link of the website hosted

http://el-p1.s3-website.ap-south-1.amazonaws.com/