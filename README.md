# web-host-tf
We host website through S3 bucket using Terraform

create main.tf file and used this code

# main.tf

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# Create the S3 bucket
resource "aws_s3_bucket" "website_bucket" {
  bucket = var.bucket_name
}

# Enable ACLs for the bucket
resource "aws_s3_bucket_public_access_block" "block_public" {
  bucket = aws_s3_bucket.website_bucket.bucket

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

# Ensure the bucket owner has full control
resource "aws_s3_bucket_ownership_controls" "ownership_controls" {
  bucket = aws_s3_bucket.website_bucket.bucket

  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

# Set the bucket ACL to public-read
resource "aws_s3_bucket_acl" "public_read_acl" {
  bucket = aws_s3_bucket.website_bucket.bucket
  acl    = "public-read"

  depends_on = [
    aws_s3_bucket_public_access_block.block_public,
    aws_s3_bucket_ownership_controls.ownership_controls,
  ]
}

# Add a bucket policy to allow public read access
resource "aws_s3_bucket_policy" "bucket_policy" {
  bucket = aws_s3_bucket.website_bucket.bucket

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "PublicReadGetObject"
        Effect    = "Allow"
        Principal = "*"
        Action    = "s3:GetObject"
        Resource  = "${aws_s3_bucket.website_bucket.arn}/*"
      }
    ]
  })

  depends_on = [aws_s3_bucket_public_access_block.block_public]
}

# Configure the bucket as a static website
resource "aws_s3_bucket_website_configuration" "website_config" {
  bucket = aws_s3_bucket.website_bucket.bucket

  index_document {
    suffix = "index.html"
  }

  # Optional: error_document
  # error_document {
  #   key = "error.html"
  # }
}

# Upload index.html
resource "aws_s3_object" "index" {
  bucket       = aws_s3_bucket.website_bucket.bucket
  key          = "index.html"
  source       = "my-website/index.html"
  acl          = "public-read" # Make the object publicly readable
  content_type = "text/html"

  depends_on = [aws_s3_bucket_acl.public_read_acl]
}

# Upload about.html
resource "aws_s3_object" "about" {
  bucket       = aws_s3_bucket.website_bucket.bucket
  key          = "about.html"
  source       = "my-website/about.html"
  acl          = "public-read" # Make the object publicly readable
  content_type = "text/html"

  depends_on = [aws_s3_bucket_acl.public_read_acl]
}

# Upload contact.html
resource "aws_s3_object" "contact" {
  bucket       = aws_s3_bucket.website_bucket.bucket
  key          = "contact.html"
  source       = "my-website/contact.html"
  acl          = "public-read" # Make the object publicly readable
  content_type = "text/html"

  depends_on = [aws_s3_bucket_acl.public_read_acl]
}

# Upload styles.css
resource "aws_s3_object" "style" {
  bucket       = aws_s3_bucket.website_bucket.bucket
  key          = "css/styles.css"
  source       = "my-website/css/styles.css"
  acl          = "public-read" # Make the object publicly readable
  content_type = "text/css"

  depends_on = [aws_s3_bucket_acl.public_read_acl]
}


------------------------------------------------------------------------
then create variable.tf and used this

# variables.tf

variable "aws_region" {
  description = "AWS region where the bucket will be created"
  default     = "us-east-1"
}

variable "bucket_name" {
  description = "The unique name for the S3 bucket"
  default     = "my-multi-page-website-bucket" # Make sure the bucket name is unique

  
}

