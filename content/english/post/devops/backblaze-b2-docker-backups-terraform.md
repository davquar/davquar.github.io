---
author: Davide Quaranta
title: Automatic Docker volume backups on Backblaze B2 with Terraform
date: 2023-12-05T00:00:00+02:00
categories: [DevOps]
tags: [docker,terraform,backblaze,infrastructure,iac]
description: "Creating an automatic backup strategy for Docker volumes, on Backblaze B2 buckets managed with Terraform, with state files securely saved on B2 as well."
---

On different machines I've been running a number of Docker containers, some of which have volumes that I'm interested in saving (and restoring, in case bad things happen).

I've been using [`offen/docker-volume-backup`](https://github.com/offen/docker-volume-backup) for quite some time, which is a great project that allows to automatically backup Docker volumes to multiple places, including local filesystems, WebDAV, SSH remotes, S3, Dropbox.

The problem is that I didn't really have a backup strategy, meaning that the only cool thing in my setup was this `offen/docker-volume-backup` thingy, which I configured to just backup locally; then periodically I used to manually copy the backups somewhere else for long-term storage. Backups were also not encrypted.

The problems with that approach are:

* It feels like a chore.
* Lack of standard.
* Probability of human error.
* Lack of encryption.

## The solution

From the problems directly follows the solution I adopted, which goal is to:

* Automate the long-term storage of backups.
* Use a secure storage.
* Use Infrastructure-as-Code (IaC) to define the storage properties.

In other words, I ended up making this:

```text
schema...
```

## The setup

I'm using:

* offen/docker-volume-backups to:
  * Backup Docker volumes.
  * Encrypt them with PGP.
  * Push them to a Backblaze B2 bucket.
  * Manage its own retention policy.
* Two Backblaze B2 buckets to:
  * Store backups.
  * Store the Terraform statefiles.
* Two Terraform modules to describe the B2 buckets.

Let's see all.

## Backblaze application keys

We want to have a proper access management, meaning that we need two different sets of privileges:

* A broad set for Terraform operations.
* A restricted set for docker-volume-backups operations.

To achieve this, we can create two application keys. Let's respectively call them $K_t$ and $T_o$. We need to annotate the their `keyID` and `keyName`.

## Terraform

We'll have two Terraform modules:

* `backblaze-tfstates`: to manage Terraform state files storage.
* `backups-bucket`: to manage the backups storage.

The structure is:

```
.
├── backblaze-tfstates
│   ├── main.tf
│   ├── outputs.tf
│   ├── requirements.tf
│   └── variables.tf
└── backups-bucket
    ├── backend.tf
    ├── main.tf
    ├── outputs.tf
    ├── requirements.tf
    └── variables.tf
```

### `backblaze-tfstates`

Since state files can contain secrets, we want to store them in a secure location; we'll use this bucket for it. Let's define it.

#### `main.tf`
```hcl
provider "b2" {
  application_key    = var.application_key
  application_key_id = var.application_key_id
}

resource "b2_bucket" "tfstates" {
  bucket_name = "tfstates-${random_string.suffix.id}"
  bucket_type = "allPrivate"
  default_server_side_encryption {
    algorithm = "AES256"
    mode      = "SSE-B2"
  }
}

resource "random_string" "suffix" {
  lower   = true
  upper   = false
  special = false
  length  = 10
}

```

Observations:

* Application keys are given, not hardcoded :)
* Since B2 bucket names have a global scope, we are adding a random suffix.

#### `variables.tf`

```hcl
variable "application_key" {
  type      = string
  sensitive = true
}

variable "application_key_id" {
  type      = string
  sensitive = true
}
```

#### `outputs.tf`

```hcl
output "bucket_name" {
  value = b2_bucket.tfstates.bucket_name
}
```

#### `requirements.tf`

```hcl
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    b2 = {
      source = "Backblaze/b2"
    }
  }
}
```