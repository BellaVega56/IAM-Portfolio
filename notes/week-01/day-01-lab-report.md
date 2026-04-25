# Week 1 Day 1: Identity Foundations

**Date:** April 25, 2026  
**Platform:** Okta (Preview Sandbox)  
**Focus:** Universal Directory, User Lifecycle States, Mastering

---

## Objective

Understand how Okta's Universal Directory works as the foundation for identity management. Learn how users are represented, how data flows between sources, and how users move through lifecycle states.

---

## Core Concepts Learned

### 1. The Directory as a Hub

A directory isn't just a list of users — it's a **hub** that:
- **Ingests** identity data from multiple sources (HR systems, Active Directory, manual entry)
- **Normalizes** that data into a consistent format
- **Distributes** it to downstream applications (Salesforce, Slack, GitHub, etc.)

```
     Sources                    Hub                    Destinations
     ───────                    ───                    ────────────
     HR System    ─────┐                      ┌────▶  Salesforce
                       │                      │
     Active       ─────┼────▶  OKTA  ─────────┼────▶  Slack
     Directory         │     DIRECTORY        │
                       │                      └────▶  GitHub
     Manual Entry ─────┘
```

This "hub and spoke" model is why Okta calls it the **Universal Directory** — it's meant to be the single source of truth that unifies all identity sources.

### 2. Three Profile Types

Okta maintains three distinct profile types, which is a common source of confusion:

| Profile Type | What It Represents | Example |
|--------------|-------------------|---------|
| **Okta User Profile** | The master identity record within Okta | Sarah Chen as Okta knows her |
| **App User Profile** | How a specific application sees the user | Sarah's Salesforce username might differ from her Okta login |
| **IdP User Profile** | How an external identity provider describes the user | What Google says about Sarah if she uses Google SSO |

**Key insight:** When troubleshooting "why does the app have the wrong value?", the answer is usually in the **mappings** between the Okta User Profile and the App User Profile.

### 3. Mastering

**Mastering** determines which system is the authoritative source for each attribute.

| If This System Masters... | Then Changes Elsewhere... |
|---------------------------|---------------------------|
| HR masters `department` | Any change to `department` in Okta gets overwritten on next HR sync |
| Okta masters `phone` | HR's phone value is ignored; Okta's value wins |

**Critical understanding:** The master wins on **every sync**, not just when the master's value changes. If HR masters an attribute, Okta copies from HR every time — regardless of whether HR made a change.

### 4. User Lifecycle States

Users move through states that control what they can do:

| State | Can Login? | How to Enter | How to Exit |
|-------|------------|--------------|-------------|
| **Staged** | No | CSV import, API creation | Admin activates |
| **Pending User Action** | No | Activation email sent | User completes setup |
| **Active** | Yes | User sets password OR admin sets password | Admin suspends/deactivates |
| **Password Expired** | Yes (forced change) | Password policy expires it | User changes password |
| **Locked Out** | No | Too many failed attempts | Admin unlocks OR timeout |
| **Suspended** | No | Admin suspends | Admin unsuspends |
| **Deactivated** | No | Admin terminates | Can be reactivated |

### 5. Login vs Email

| Attribute | Purpose |
|-----------|---------|
| `login` | What the user types to authenticate |
| `email` | Contact address for notifications |

These are often identical but **don't have to be**. Assuming they're the same can break app integrations when they diverge (e.g., after a name change).

---

## What I Configured

### Test Data Import

Imported 50 fictional employees representing a realistic company structure.

**Source file:** `data/employees.csv`
- 50 employees across 8 departments
- Mix of employment types: full-time, contractor, intern
- Includes management hierarchy (manager_email field)
- Security clearance levels: none, standard, elevated, privileged
- Future hire dates for pre-staging scenarios

**Okta import file:** `data/okta-users-import.csv`
- Transformed to Okta's expected format
- Columns: login, firstName, lastName, email, secondEmail, mobilePhone
- All emails use `@acmelabs.test` (reserved TLD, won't route to real mailboxes)

### Import Process

1. Navigated to **Directory → People → More Actions → Import users from CSV**
2. Uploaded `okta-users-import.csv`
3. Mapped columns to Okta attributes (auto-detected correctly)
4. Left "Automatically activate new users" **unchecked** (to observe Staged state)
5. Left "Do not create password / IdP only" **unchecked** (no external IdP configured)
6. Imported 50 users — all created in **Staged** state

### User Activation

Activated one user (Victoria Reed) to observe state transitions:

| Step | State Before | Action | State After |
|------|--------------|--------|-------------|
| 1 | Staged | Clicked "Activate" | Pending User Action |
| 2 | Pending User Action | Clicked "Set Password & Activate" | Active (Password Expired) |

**Why "Password Expired"?** When an admin sets a password, Okta flags it as one-time/temporary. The user must change it on first login. This is a security practice — admin-set passwords might be shared insecurely.

### System Log Observations

Found the activation audit trail in **Reports → System Log**:

| Timestamp | Event Type | Actor | Target | Outcome |
|-----------|-----------|-------|--------|---------|
| 13:31:20 | Create okta user | User Admin | Victoria Reed | SUCCESS |
| 13:31:20 | Activate factor for user | User Admin | Victoria Reed | SUCCESS |
| 13:35:52 | Activate Okta user | User Admin | Victoria Reed | SUCCESS |
| 13:36:51 | User update password | User Admin | Victoria Reed | SUCCESS |
| 13:36:51 | Password expired | User Admin | Victoria Reed | SUCCESS |

**Key fields in each event:**
- **Actor** — Who performed the action (me, the admin)
- **Target** — Who was affected (Victoria Reed)
- **Client IP** — 74.244.143.185 (my IP address)
- **Outcome** — SUCCESS/FAILURE

---

## Quiz Results

**Score: 4/5**

| Q | My Answer | Correct | Result |
|---|-----------|---------|--------|
| 1 | B (Staged) | B | ✓ |
| 2 | C (login) | C | ✓ |
| 3 | A (change kept) | B (overwritten) | ✗ |
| 4 | C (actor) | C | ✓ |
| 5 | C (complete setup via email) | C | ✓ |

### Question 3: What I Got Wrong and Why

**The question:** If HR is the "master" for the `department` attribute, and you change it directly in Okta, what happens on the next HR sync?

**My answer:** A — Your change is kept

**Correct answer:** B — Your change is overwritten with HR's value

**My thought process:**
I was thinking that the master only "wins" when the master actually changes something. So if HR didn't make a change, maybe Okta's local change would persist.

**Why I was wrong:**
Mastering doesn't work conditionally. The master wins on **every sync**, unconditionally. When the sync runs, Okta doesn't ask "did HR change this value?" — it simply copies from the master source.

Think of it like this: the sync is a **full copy**, not a **differential merge**. Whatever the master has, Okta takes — every time.

**The corrected mental model:**
- Master source always wins on sync
- It doesn't matter if the master's value changed or not
- It doesn't matter if someone manually edited Okta
- The only way to prevent overwriting is to change who masters the attribute

---

## Directory State After Day 1

| Metric | Count |
|--------|-------|
| Total users | 52 (50 imported + 2 existing) |
| Staged | 49 |
| Active | 1 (Victoria Reed) |
| Pending User Action | 0 |
| Groups | 0 (creating these on Day 3) |

---

## Key Takeaways

1. **The directory is a hub, not a database.** It aggregates from sources and distributes to apps.

2. **Mastering is absolute.** The master wins every sync, not just when it changes.

3. **Lifecycle states control capabilities.** A Staged user can't log in or be provisioned to apps.

4. **The System Log is the audit trail.** Every action has an actor, target, timestamp, and outcome.

5. **Login ≠ Email (necessarily).** Don't hardcode assumptions about their relationship.

---

## Next Steps

Day 2: Microsoft Entra ID — Set up tenant, import users, compare directory model to Okta.
