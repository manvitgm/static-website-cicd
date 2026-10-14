# 🚀 Static Website CI/CD Pipeline

## GitHub → AWS CodeBuild → S3

A simple static website with a CI/CD pipeline that automatically builds and deploys to AWS S3 whenever you push code to GitHub.

---

## 📁 Project Structure

```
static-website-cicd/
├── index.html          # Static website
├── styles.css          # Stylesheet
├── buildspec.yml       # AWS CodeBuild configuration
├── .gitignore          # Git ignore rules
└── README.md           # This file
```

---

## 🛠️ Setup Instructions

### Step 1: Create a GitHub Repository

1. Go to [github.com](https://github.com) → Click **"New Repository"**
2. Name it `static-website-cicd`
3. Keep it **Public** (or Private)
4. Click **"Create repository"**
5. Push this project to GitHub:

```bash
cd static-website-cicd
git init
git add .
git commit -m "Initial commit - static website with buildspec"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/static-website-cicd.git
git push -u origin main
```

---

### Step 2: Create an S3 Bucket for Hosting

1. Go to **AWS Console** → **S3**
2. Click **"Create bucket"**
3. Bucket name: `my-static-website-cicd` (must be globally unique)
4. Region: Choose your preferred region
5. **Uncheck** "Block all public access" (since it's a public website)
6. Acknowledge the warning and click **"Create bucket"**
7. Go to bucket → **Properties** → **Static website hosting** → **Enable**
   - Index document: `index.html`
8. Go to **Permissions** → **Bucket Policy** → Add this policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-static-website-cicd/*"
        }
    ]
}
```

> ⚠️ Replace `my-static-website-cicd` with your actual bucket name.

---

### Step 3: Create an IAM Role for CodeBuild

1. Go to **AWS Console** → **IAM** → **Roles** → **Create role**
2. Trusted entity: **AWS service** → **CodeBuild**
3. Attach these policies:
   - `AmazonS3FullAccess` (for deploying to S3)
   - `CloudWatchLogsFullAccess` (for build logs)
4. Role name: `codebuild-static-website-role`
5. Click **"Create role"**

---

### Step 4: Create a CodeBuild Project

1. Go to **AWS Console** → **CodeBuild** → **Create build project**

2. **Project configuration:**
   - Project name: `static-website-build`

3. **Source:**
   - Source provider: **GitHub**
   - Click **"Connect to GitHub"** → Authorize AWS CodeBuild
   - Repository: Choose **"Repository in my GitHub account"**
   - Select your `static-website-cicd` repository

4. **Primary source webhook events (Enable CI/CD):**
   - ✅ Check **"Rebuild every time a code change is pushed to this repository"**
   - Event type: Select **PUSH**
   - Filter group: HEAD_REF = `^refs/heads/main$` (to trigger only on main branch)

5. **Environment:**
   - Environment image: **Managed image**
   - Operating system: **Ubuntu**
   - Runtime: **Standard**
   - Image: **aws/codebuild/standard:7.0**
   - Service role: Select `codebuild-static-website-role`

6. **Environment variables:**
   - Name: `S3_BUCKET_NAME` → Value: `my-static-website-cicd`

7. **Buildspec:**
   - Select **"Use a buildspec file"** (it will auto-detect `buildspec.yml`)

8. Click **"Create build project"**

---

### Step 5: Test the Pipeline

1. Make a small change to `index.html` (e.g., change the title text)
2. Commit and push:

```bash
git add .
git commit -m "Test CI/CD pipeline"
git push origin main
```

3. Go to **CodeBuild** → Your project → Watch the build run automatically! ✅
4. Once the build succeeds, visit your S3 website URL:
   ```
   http://my-static-website-cicd.s3-website-<region>.amazonaws.com
   ```

---

## 📊 Pipeline Flow

```
  Developer          GitHub            CodeBuild           S3 Bucket
     |                  |                  |                  |
     |--- git push ---->|                  |                  |
     |                  |--- webhook ----->|                  |
     |                  |                  |--- build ------->|
     |                  |                  |--- deploy ------>|
     |                  |                  |                  |
     |<-------------- Website is Live! --------------------- |
```

---

## 🔍 Troubleshooting

| Issue | Solution |
|-------|----------|
| Build fails with S3 permission error | Check IAM role has `AmazonS3FullAccess` |
| Webhook not triggering | Verify webhook is enabled in CodeBuild source settings |
| Website not accessible | Ensure S3 bucket policy allows public read access |
| 404 on website | Check that `index.html` is in the root of the S3 bucket |

---

## 📝 Notes

- The `buildspec.yml` defines the build steps that CodeBuild executes
- The webhook automatically triggers a build whenever you push to the `main` branch
- Build logs are available in **CloudWatch Logs** and in the **CodeBuild console**
