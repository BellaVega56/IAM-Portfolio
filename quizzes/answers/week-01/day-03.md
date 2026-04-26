# Week 1 Day 3 Quiz Answers

**Score: 5/5**

---

## Questions and Answers

**Q1.** What type of attribute restricts input to a predefined list of values?
- **My Answer:** C — Enum ✓
- **Correct:** C

**Q2.** Which operator is used for comparison in Okta Expression Language?
- **My Answer:** B — `==` ✓
- **Correct:** B

**Q3.** A dynamic group rule evaluates membership based on:
- **My Answer:** B — User attributes matching a condition ✓
- **Correct:** B

**Q4.** You create a rule: `user.department == "Sales"`. A user with `department = null` will:
- **My Answer:** C — Not be added to the group ✓
- **Correct:** C
- **Why:** Null doesn't equal "Sales", so the condition evaluates to false. No error, just no match.

**Q5.** After activating a group rule, how long before membership updates?
- **My Answer:** B — Within a few minutes (rule evaluation is periodic) ✓
- **Correct:** B
