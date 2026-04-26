# Week 1 Day 5: Admin Roles and System Log

**Date:** April 25, 2026  
**Platform:** Okta (Preview Sandbox)  
**Focus:** Admin privilege models, scoped admin roles, System Log for debugging and audit

---

## Objective

Understand how admin privileges work in Okta, implement the principle of least privilege through scoped admin roles, and master the System Log as the primary tool for debugging and compliance.

---

## Core Concepts Learned

### 1. Principle of Least Privilege

Every admin should have exactly the permissions they need — no more, no less.

| Anti-pattern | Best Practice |
|--------------|---------------|
| "Everyone on IT is Super Admin" | Each admin has specific role for their job function |
| One shared admin account | Individual accounts with auditable actions |
| "We'll figure it out when something breaks" | Documented role matrix with clear boundaries |

### 2. Standard Admin Roles

Okta provides built-in roles for common scenarios:

| Role | Permissions | Use Case |
|------|-------------|----------|
| **Super Admin** | Everything — full control | Break-glass only, 2-3 per org max |
| **Org Admin** | Almost everything except some security settings | Senior admin daily work |
| **App Admin** | Manage applications | Integration team |
| **Group Admin** | Manage groups and membership | Team leads |
| **Help Desk Admin** | Reset passwords, unlock accounts | Tier 1 support |
| **Report Admin** | View reports and System Log only | Security analysts |
| **Read-Only Admin** | View all configuration, change nothing | External auditors |
| **API Access Management Admin** | Manage OAuth authorization servers | API security team |

**Key insight:** Read-Only Admin sees configuration; Report Admin sees only reports/logs. For auditors reviewing how things are set up, Read-Only Admin is appropriate.

### 3. Custom Roles + Resource Sets

Standard roles are broad. Custom roles enable fine-grained control:

| Component | What It Defines |
|-----------|-----------------|
| **Custom Role** | What actions the admin can perform |
| **Resource Set** | Which resources (users, groups, apps) those actions apply to |

**Example:** 
- Custom role: "Password Reset Only" (just reset passwords, unlock accounts)
- Resource set: "Marketing Department Users" (only users in dept-marketing)
- Result: Admin can reset passwords for Marketing only

This is **scoped admin** — limiting both action AND scope.

### 4. The System Log

Every action in Okta generates an event in the System Log. This is the audit trail.

**Event structure:**

| Field | What It Contains |
|-------|------------------|
| `eventType` | The type of action (e.g., `user.lifecycle.activate`) |
| `actor` | Who performed the action |
| `target` | What was acted upon |
| `client` | Device, browser, IP address |
| `outcome` | SUCCESS or FAILURE |
| `published` | Timestamp |
| `debugContext` | Additional diagnostic details |

**Common event types:**

| Event Type | Meaning |
|------------|---------|
| `user.lifecycle.create` | User created |
| `user.lifecycle.activate` | User activated |
| `user.lifecycle.deactivate` | User terminated |
| `user.session.start` | User logged in |
| `user.account.reset_password` | Password reset |
| `user.account.privilege.grant` | Admin role assigned |
| `group.user_membership.add` | User added to group |
| `policy.evaluate_sign_on` | Authentication policy evaluated |

### 5. Using System Log for Debugging

**Example debug flow:**
1. User reports: "I can't log in"
2. Search System Log for user's recent events
3. Find `policy.evaluate_sign_on` event
4. Event shows: policy required WebAuthn, user only has SMS
5. Solution: User enrolls WebAuthn, or adjust policy

### 6. Using System Log for Compliance

Auditors ask questions like:
- "Show me all admin role changes last quarter"
- "Who reset passwords for executives?"
- "When was this user terminated?"

The System Log has the answers. Filter, export to CSV, provide as evidence.

---

## What I Configured

### Reviewed Standard Roles

Examined all built-in admin roles and identified appropriate use cases:

| Role | Appropriate For |
|------|-----------------|
| Super Admin | IAM Team Lead (break-glass only) |
| Org Admin | Senior IAM Engineer |
| App Admin | Integration Specialist |
| Group Admin | Department Manager |
| Help Desk Admin | Tier 1 Support |
| Report Admin | Security Analyst (log access only) |
| Read-Only Admin | External Auditor (full config visibility) |

### Created Custom Admin Role

**Role name:** `Tier1 Help Desk - Password Only`

**Permissions granted:**
- Reset passwords
- Unlock accounts

**Permissions NOT granted:**
- Create/delete users
- Modify groups
- Change policies
- Access applications
- Everything else

### Created Resource Set

**Resource set name:** `Marketing Department Users`

**Scope:** Users in the `dept-marketing` group

### Assigned Scoped Admin

Assigned a test user the custom role scoped to the resource set.

**Result:** This admin can ONLY reset passwords for Marketing users. Attempts to:
- Reset password for Engineering user → blocked
- Create new user → blocked
- Modify policies → blocked

### System Log Exploration

**Events examined:**

| Event Type | What I Found |
|------------|--------------|
| `user.lifecycle.activate` | My Day 1 user activations |
| `user.account.privilege.grant` | The admin role assignment I just made |
| `group.user_membership.add` | Dynamic group rule additions |

**Event structure observed:**
- Actor = my admin account
- Target = affected user
- Client IP = my IP address
- Outcome = SUCCESS
- Timestamp = precise to millisecond

---

## Quiz Results

**Score: 4/5**

| Q | My Answer | Correct | Result |
|---|-----------|---------|--------|
| 1 | B (2-3 Super Admins) | B | ✓ |
| 2 | B (Resource set) | B | ✓ |
| 3 | C (actor) | C | ✓ |
| 4 | C (Report Admin) | D (Read-Only Admin) | ✗ |
| 5 | B (Admin role assigned) | B | ✓ |

### Question 4: What I Got Wrong and Why

**The question:** External auditor needs read-only access to review Okta configuration. Which role?

**My answer:** C — Report Admin

**Correct answer:** D — Read-Only Admin

**Why I was wrong:**

| Role | What They Can See |
|------|-------------------|
| Report Admin | Reports and System Log only |
| Read-Only Admin | Everything — all configuration, policies, apps, groups |

An auditor reviewing "configuration" needs to see how things are set up — policies, apps, group rules, admin assignments. Report Admin only sees logs, not config.

**The distinction:** Report Admin = log access. Read-Only Admin = full visibility.

---

## System Log Event Types Reference

Quick reference for common debugging scenarios:

| Scenario | Event Type to Search |
|----------|---------------------|
| User can't log in | `user.session.start` (look for failures), `policy.evaluate_sign_on` |
| User claims they didn't reset password | `user.account.reset_password` (check actor) |
| When was user terminated? | `user.lifecycle.deactivate` |
| Who granted admin access? | `user.account.privilege.grant` |
| Why is user in wrong group? | `group.user_membership.add`, `group.user_membership.remove` |
| App access issues | `user.authentication.sso`, `application.user_membership.add` |

---

## Week 1 Okta Summary

| Day | Topic | Key Concepts |
|-----|-------|--------------|
| Day 1 | Identity Foundations | Directory as hub, mastering, lifecycle states |
| Day 3 | Custom Attributes & Groups | Schema extension, static vs dynamic groups, Expression Language |
| Day 5 | Admin Roles & System Log | Least privilege, scoped admin, audit trail |

---

## Key Takeaways

1. **Super Admin is break-glass only.** 2-3 accounts max, used for emergencies.

2. **Custom role + resource set = scoped admin.** Define WHAT they can do AND WHO they can do it to.

3. **Read-Only Admin ≠ Report Admin.** Read-Only sees all config; Report Admin sees only logs.

4. **System Log is the debugging home.** Every action recorded with actor, target, outcome.

5. **Event types are vocabulary.** Learn `user.lifecycle.*`, `user.session.*`, `user.account.*`.

6. **Export for compliance.** CSV exports provide audit evidence.

---