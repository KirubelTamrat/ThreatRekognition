# AI Threat Detection System

A cloud-based image threat detection system built in Python using AWS Rekognition and Amazon S3. Upload images to a bucket, run the script, and it tells you whether anything dangerous is in them — weapons, fire, smoke, explosions, or other unsafe content — along with a confidence score for each finding.

This project was built to explore how cloud computer vision can be applied to real-world safety problems, and to get hands-on experience connecting Python applications to AWS services.

---

## Why This Exists

Manually reviewing images for unsafe content doesn't scale. Whether you're moderating user uploads, monitoring security feeds, or just learning how AWS works, automating that detection with a machine learning API is a practical and powerful approach.

This project demonstrates how to do that with minimal infrastructure — no servers, no model training, just Python and AWS.

---

## The Tools and Why We Use Them

**AWS Rekognition**
Rekognition is Amazon's computer vision service. It can identify objects, scenes, people, text, and unsafe content in images without you needing to build or train any model yourself. We use it here because it's accurate, fast, and already integrated into the AWS ecosystem. You send it an image, it sends back labels with confidence scores.

**Amazon S3**
S3 is Amazon's object storage service. Instead of reading images off your local machine, we store them in S3 and point Rekognition directly at them. This is how production systems work — keeping compute and storage separate, and making it easy to scale up to thousands of images.

**boto3**
boto3 is the official AWS SDK for Python. It's how our script talks to both S3 and Rekognition. Every API call — detecting labels, handling errors, connecting to your account — goes through boto3.

**IAM (Identity and Access Management)**
IAM is how AWS controls who can do what. Before the script can call Rekognition or read from S3, your AWS user needs to be granted those specific permissions. We configure this through an IAM policy so the script only has access to exactly what it needs.

---

## How It Works

At a high level, the script takes a list of image file names, finds them in your S3 bucket, and runs two types of analysis on each one using Rekognition:

**1. Moderation Label Detection — `detect_moderation_labels()`**

This API is specifically designed to catch unsafe content. It looks for things like violence, graphic imagery, and explicit material. AWS has trained this model on a large dataset of flagged content, so it knows what "unsafe" looks like across a wide range of scenarios.

**2. Object and Scene Detection — `detect_labels()`**

This API identifies everything visible in an image — people, objects, environments, activities. We take those results and check them against a custom danger list:

```python
{"Gun", "Weapon", "Knife", "Fire", "Explosion", "Smoke", "Blood", "Rifle", "Pistol"}
```

If Rekognition detects any of those objects with at least 50% confidence, the image gets flagged. You can customize this list to fit your use case.

Both checks run on every image. If either one finds something, the image is marked as a threat and the details are printed out.

---

## Before You Start

You will need:

- Python 3.9 or higher
- An AWS account
- An S3 bucket with images uploaded to it
- AWS credentials configured on your machine (explained below)

---

## Setup

**1. Clone the repo**

```bash
git clone https://github.com/YOUR-USERNAME/threat-detector-image-analyzer.git
cd threat-detector-image-analyzer
```

**2. Create a virtual environment**

Mac/Linux:
```bash
python3 -m venv venv
source venv/bin/activate
```

Windows:
```bash
python -m venv venv
venv\Scripts\activate
```

**3. Install dependencies**

```bash
pip install -r requirements.txt
```

---

## AWS Credentials

The script needs to authenticate with your AWS account to call Rekognition and access S3. There are two ways to do this.

**Option 1 — AWS CLI (Recommended)**

```bash
aws configure
```

It will prompt you for your Access Key ID, Secret Access Key, region (use `us-east-1` unless you know otherwise), and output format (just hit Enter). boto3 picks these up automatically — no changes to the code needed.

**Option 2 — Hardcode in app.py**

```python
AWS_ACCESS_KEY_ID = "your_key_here"
AWS_SECRET_ACCESS_KEY = "your_secret_here"
```

> **Never push real credentials to GitHub.** Use placeholders when sharing code publicly. If you accidentally expose a key, go to the AWS IAM console and deactivate it immediately.

---

## AWS Permissions

Your IAM user needs explicit permission to call Rekognition and read objects from S3. Without this, every API call will return an "Access Denied" error.

Go to the [AWS IAM console](https://console.aws.amazon.com/iam/), find your user, and attach this policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "rekognition:DetectLabels",
        "rekognition:DetectModerationLabels"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

> Replace `YOUR-BUCKET-NAME` with your actual bucket name.

---

## Running the Project

Open `app.py` and update these two things:

```python
bucket = "your-bucket-name"

photos = [
    "your-image-1.jpg",
    "your-image-2.jpg"
]
```

Make sure the file names match **exactly** what's in your S3 bucket — spelling, capitalization, and file extension all have to be right.

Then run:

```bash
python app.py
```

---

## Example Output

```
Checking: image1.jpg

Threats detected:
  - Object: Gun (98.4%)

----------------------------------------

Checking: image2.jpg

No threats found.

----------------------------------------

Checking: image3.jpg

Threats detected:
  - Moderation: Violence (91.2%)
  - Object: Fire (87.6%)

----------------------------------------
```

---

## Troubleshooting

**"The security token included in the request is invalid"**
Your credentials are wrong or expired. Re-run `aws configure` with valid keys.

**"Access Denied"**
Your IAM user doesn't have the right permissions. Add the policy shown in the AWS Permissions section above.

**Image not found / NoSuchKey**
The file name in your code doesn't match what's in S3. Double-check spelling and capitalization.

**"AWS credentials not found"**
You haven't configured credentials yet. Run `aws configure`.

---
