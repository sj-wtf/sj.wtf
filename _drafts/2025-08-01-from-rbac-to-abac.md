---
layout: post
title:  "From RBAC to ABAC: an adventure in IAM"
date:   2025-08-01 00:00:00 -0500
categories: iam cloud coding
---
For as long as I can remember, RBAC (or Role Based Access Controls) have been the primary way of managing access to and within systems, incuding cloud systems, which will be my focus today. A user is usually assigned a specially crafted role granting them the access they should need to do their job. Crafting those job-specific IAM roles often includes both actions they can do as well as resources they should have access to. Managing a role per job family and tweaking each role separately from other roles can be a tedious task, and can result in either role sprawl (so many roles that they become unmanageable) or overly permissive and broadly applied roles. Both can be a detriment, role sprawl usually means you're not paying enough attention to each of your roles, and overly permissive roles mean you're not keeping to the principle of least privilege, and people can possibly get into trouble with the access they have.

ABAC, or Attribute Based Access Control, avoids the problem of role sprawl and can allow you to grant a great number of users roles tailored specifically to their needs. This is achieved by ensuring that users have appropriate attributes in your identity system, and that your cloud resources have corresponding labels or tags to either match or help make decisions about what users should and should not have access to.
