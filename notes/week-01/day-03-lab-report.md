# Week 1 Day 3: Custom Attributes and Groups

**Date:** April 25, 2026  
**Platform:** Okta (Preview Sandbox)  
**Focus:** Schema extension, static groups, dynamic group rules, Expression Language

---

## Objective

Extend the Okta user schema with custom attributes, create organizational groups, and implement dynamic group rules that automatically manage membership based on user attributes.

---

## Core Concepts Learned

### 1. Custom Attributes

Built-in attributes (`firstName`, `lastName`, `email`, `department`) cover basics, but every organization has unique data needs. Custom attributes extend the schema.

**Attribute types:**

| Type | Use Case | Example |
|------|----------|---------|
| String | Free text | `costCenter = "ENG-100"` |
| Number | Integers | `employeeNumber = 12345` |
| Boolean | True/False flags | `isManager = true` |
| Enum | Restricted list of values | `employmentType` = one of: full-time, contractor, intern |
| Array | Multiple values | `certifications = ["AWS", "CISSP"]` |

**Why enums matter:** Without enums, you get inconsistent data (`Full-Time` vs `full time` vs `FT`). Group rules fail because they look for exact matches.

### 2. Attribute Permissions

| Permission Level | Who Can Read | Who Can Write |
|------------------|--------------|---------------|
| Hidden | Admins only | Admins only |
| Read-only | Admins + user | Admins only |
| Read-write | Admins + user | Admins + user |

**Examples:** 
- `securityClearance` → Hidden (sensitive)
- `hireDate` → Read-only (user can see, can't edit)
- `phone` → Read-write (user self-service)

### 3. Static vs Dynamic Groups

| Type | How Membership Works | Scales? |
|------|---------------------|---------|
| Static | Admin manually adds/removes users | No — manual effort, easy to forget |
| Dynamic | Rule evaluates user attributes automatically | Yes — always in sync with attribute values |

**When to use each:**
- **Dynamic:** Department groups, contractor groups, anything driven by attributes
- **Static:** Project teams, executive groups, anything without a defining attribute

### 4. Okta Expression Language

The syntax for writing group rule conditions.

**Basic operators:**

| Operator | Meaning | Example |
|----------|---------|---------|
| `==` | Equals | `user.department == "Engineering"` |
| `!=` | Not equals | `user.department != "HR"` |
| `AND` | Both conditions true | `user.department == "Engineering" AND user.title == "Manager"` |
| `OR` | Either condition true | `user.department == "Sales" OR user.department == "Marketing"` |

**String functions:**

| Function | Purpose | Example |
|----------|---------|---------|
| `String.stringContains(source, target)` | Check if source contains target | `String.stringContains(user.title, "Senior")` |
| `String.toLowerCase(value)` | Convert to lowercase | `String.toLowerCase(user.department)` |

**Critical distinction:** 
- `==` is comparison (use in rules)
- `=` is assignment (don't use in rules)

### 5. Null Handling

If an attribute is null (empty), expressions don't error — they just evaluate to false.

```
user.department == "Sales"
```

If `department` is null → condition is false → user not added to group. No error.

For string functions, null can cause issues:
```
String.stringContains(user.title, "Senior")
```

Defensive version:
```
user.title != null AND String.stringContains(user.title, "Senior")
```

### 6. Rule Evaluation Timing

Group rules **don't evaluate instantly**. Okta processes them periodically — usually within a few minutes, but can be longer at scale.

This means:
- Change an attribute → wait 2-5 minutes → check group membership
- Don't assume immediate updates in production automation

---

## What I Configured

### Custom Attributes Added

Extended the Okta User Profile with 5 custom attributes:

| Attribute | Type | Purpose | Permission |
|-----------|------|---------|------------|
| `costCenter` | String | Accounting cost center code | Read-only |
| `employmentType` | String | Full-time, contractor, or intern | Read-only |
| `hireDate` | String | Employee start date (YYYY-MM-DD) | Read-only |
| `managerEmail` | String | Direct manager's email | Read-only |
| `securityClearance` | String | Security clearance level | Hidden |

**Note:** `costCenter` already existed in the sandbox. `employmentType` was created as plain string (enum option was restricted in Preview Sandbox).

### User Attributes Populated

Populated attributes for 10 test users:

| User | Department | Employment Type | Security Clearance |
|------|------------|-----------------|-------------------|
| Sarah Chen | Engineering | full-time | standard |
| Michael Torres | Engineering | full-time | elevated |
| Jessica Patel | Engineering | contractor | standard |
| Lisa Martinez | Sales | full-time | standard |
| Nathan Coleman | Sales | intern | none |
| Patricia Morgan | Finance | full-time | privileged |
| Samantha Green | Finance | full-time | standard |
| Matthew Carter | HR | full-time | elevated |
| Jennifer Walsh | Executive | full-time | privileged |
| Ryan O'Brien | Engineering | full-time | privileged |

### Groups Created

**Static groups (8):**

| Group Name | Purpose |
|------------|---------|
| `dept-engineering` | Engineering department members |
| `dept-sales` | Sales department members |
| `dept-marketing` | Marketing department members |
| `dept-finance` | Finance department members |
| `dept-hr` | Human Resources members |
| `dept-operations` | Operations department members |
| `role-managers` | Users with management responsibilities |
| `role-contractors` | Contract employees |

### Dynamic Group Rules Created

**Rule 1: Auto-assign Engineering**
- Condition: `user.department == "Engineering"`
- Assigns to: `dept-engineering`
- Status: Active

**Rule 2: Senior Engineers**
- Condition: `user.department == "Engineering" AND String.stringContains(user.title, "Senior")`
- Assigns to: `role-senior-engineering`
- Status: Active

**Rule 3: Contractors**
- Condition: `user.employmentType == "contractor"`
- Assigns to: `role-contractors`
- Status: Active

### Rule Behavior Tested

1. Changed a user's `employmentType` from `full-time` to `contractor`
2. Waited for rule evaluation (~2-3 minutes)
3. Verified user appeared in `role-contractors` group
4. Changed attribute back to `full-time`
5. Verified user was removed from `role-contractors`

**Confirmed:** Dynamic group membership automatically syncs with attribute values.

---

## Quiz Results

**Score: 5/5**

| Q | My Answer | Correct | Result |
|---|-----------|---------|--------|
| 1 | C (Enum) | C | ✓ |
| 2 | B (==) | B | ✓ |
| 3 | B (Attributes matching condition) | B | ✓ |
| 4 | C (Not added to group) | C | ✓ |
| 5 | B (Within a few minutes) | B | ✓ |

---

## Directory State After Day 3

| Metric | Count |
|--------|-------|
| Total users | 52 |
| Custom attributes added | 4 (1 pre-existed) |
| Static groups | 8 |
| Dynamic group rules | 3 |
| Users with populated attributes | 10 |

---

## Key Takeaways

1. **Custom attributes extend the schema** for organization-specific data.

2. **Enums enforce consistency** — but aren't always available. Consistent typing is the fallback.

3. **Dynamic groups scale; static groups don't.** Use dynamic whenever an attribute defines the membership.

4. **Expression Language uses `==`** for comparison. Using `=` breaks the rule.

5. **Null values don't error** — they just evaluate to false. Still, defensive null checks are good practice.

6. **Rule evaluation takes minutes**, not seconds. Don't expect instant updates.

---