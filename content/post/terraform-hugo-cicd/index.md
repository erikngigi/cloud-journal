+++
author = "Eric Ngigi"
title = "Automating a Full CI/CD Pipeline for a Hugo Website with Terraform. Github Actions, AWS and Cloudflare"
date = "2025-10-13"
description = "Learn how to build, deploy, and deliver static files securely via OIDC to S3, distribute them globally with CloudFront, and connect a custom subdomain through Cloudflare with ACM-managed SSL. Achieve a fully automated, secure, and scalable workflow for continuous delivery."
tags = [
    "Terraform",
    "GitHub Actions",
    "AWS",
    "Cloudflare",
    "Hugo",
    "CI/CD",
]
categories = [
    "DevOps",
    "Infrastructure as Code",
    "Cloud Automation",
    "Continous Integration & Delivery"
]
series = ["DevOps Automation Series"]
image = "terraform-aws-ci-cd-0.png"
toc = false
readingTime = false
+++

## Introduction
### Overview of the project and objectives
### Why automate static site deployment
### Tech stack overview: Terraform, AWS (S3, CloudFront, ACM), Cloudflare, GitHub Actions

## Project Strucuture and Module Overview
### Directory layout of the Terraform project
### Role of the `main.tf` file in orchestrating modules

## Module: Cloudflare – Managing DNS and SSL Integration
### Purpose: binding subdomain to CloudFront URL
### Terraform setup for Cloudflare DNS records
### Using ACM validation options to finalize SSL setup
### Code snippet and walkthrough

## Module: Security - Integrating GitHub Actions with AWS via OIDC
### Why use OIDC for GitHub-to-AWS authentication
### Key variables: GitHub org, repo, and OIDC thumbprints
### IAM roles and policies for secure S3 and CloudFront access
### Code snippet and explanation of each variable

## Module: Storage – Managing the S3 Bucket
### Purpose: hosting Hugo-generated static files
### Terraform configuration and parameters used
### Connecting S3 with CloudFront distribution
### Code snippet and explanation

## Module: Network – Configuring CloudFront and SSL
### Role of CloudFront in content delivery
### How the CloudFront distribution connects to S3
### Setting up ACM certificates for HTTPS
### Output variables: CloudFront domain and ACM validation options
### Code snippet and configuration breakdown

## Bringing It All Together in main.tf
### How the modules connect and depend on one another
### Variable flow between modules
### Running terraform init, plan, and apply
### Visual diagram of module interconnections

## Automating Deployment with GitHub Actions
### Overview of the CI/CD workflow for Hugo
### Building static files and deploying via Terraform
### Using OIDC authentication for secure AWS access

## Testing and Verifying the Deployment
### Checking CloudFront distribution and S3 content
### Verifying domain resolution in Cloudflare
### Confirming HTTPS and SSL validation

## Conclusion and Next Steps
### Benefits of using modular Terraform design
### Security and scalability advantages
### Future improvements (e.g., adding monitoring, versioning, or cache invalidations)

![Image 1](terraform-aws-ci-cd-0.png) ![Image 2](terraform-aws-ci-cd-1.png)
