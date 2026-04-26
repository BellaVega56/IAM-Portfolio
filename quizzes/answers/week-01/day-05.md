# Week 1 Day 5 Quiz Answers

**Score: 4/5**

---

## Questions and Answers

**Q1.** How many Super Admin accounts should a typical organization have?
- **My Answer:** B — 2-3 (break-glass only) ✓
- **Correct:** B

**Q2.** A custom admin role defines what actions an admin can take. What defines WHICH resources they can act on?
- **My Answer:** B — Resource set ✓
- **Correct:** B

**Q3.** In a System Log event, which field shows WHO performed the action?
- **My Answer:** C — actor ✓
- **Correct:** C

**Q4.** An external auditor needs read-only access to review your Okta configuration. Which role?
- **My Answer:** C — Report Admin ✗
- **Correct:** D — Read-Only Admin
- **Lesson learned:** Report Admin only sees reports/logs. Read-Only Admin sees all configuration (policies, apps, groups, admin roles). Auditors reviewing "configuration" need Read-Only Admin.

**Q5.** The event type `user.account.privilege.grant` indicates:
- **My Answer:** B — An admin role was assigned ✓
- **Correct:** B
