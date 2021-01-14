---
title: How to safely store secrets in Git
date: 2020-09-04
description: Storing and securing secrets has always been a headache of software engineering. Secrets are private information that you don't want the world to know like SSH keys, API tokens, passwords,...
---

![soap](/img/sops.jpeg)

Storing and securing secrets has always been a headache of software engineering. Secrets are private information that you don't want the world to know like SSH keys, API tokens, passwords,... If your application is small, a few environment variables on your local machine would suffice. At enterprisey level, you probably want a highly-available [Vault](https://www.vaultproject.io/) cluster backed by a highly-available [Consul](https://www.consul.io/) cluster. However, what if your scope is somewhere in between? You have too many secrets to manage by hand but not that many to justify a Vault installation. Maybe you should consider storing them in the same place where your code uses them (hint: G\*\*).

This way, your secrets could be versioned, diff-ed, reviewed and easily consumed by application code. However, even if the repository is private, your secrets would still need to be encrypted to reduce the attack surface. There are many open-source encrypting solutions like [blackbox](https://github.com/StackExchange/blackbox), [git-crypt](https://github.com/AGWA/git-crypt), [git-secret](https://github.com/sobolevn/git-secret), etc. but I prefer [SOPS](https://github.com/mozilla/sops) because it offers several advantages:

- Integrate with cloud key management system: AWS KMS, GCP KMS or Azure Key Vault
- Encrypt only some specific keys in JSON or YAML file instead of encrypting the whole file
- Split key using [Shamir's Secret Sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing) algorithm
- Backed by Mozilla, a large open-source not-for-profit organization

You can learn SOPS with a locally generated PGP key by reading this [perfectly written blog post](https://medium.com/@minuddinahmedrana/secrets-management-with-sops-3ed3b2ceadaa). However, in production, you probably would want to use a cloud key management solution like AWS KMS, GCP KMS or Azure Key Vault. Though inexpensive, a cloud solution will help with backup, key rotation, access management and encryption down to the [hardware level](https://en.wikipedia.org/wiki/Hardware_security_module). Because each encrypt or decrypt operation is an API call, permissions are being managed in the cloud so you never have to worry about your encrypting keys getting leaked. It's also easier for machines (CI/CD, automation scripts,...) to consume as well.

Now we have just got familiar with the newest buzzword of 2020: *DevSecGitOps.* (j/k)
