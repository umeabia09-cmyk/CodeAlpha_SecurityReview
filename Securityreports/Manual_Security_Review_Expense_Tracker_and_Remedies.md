# Manual Security Review – Expense Tracker


## 1. Review Objective

The purpose of this manual security review is to inspect the Expense Tracker source code for security weaknesses, unsafe programming practices, input-validation problems, error-handling issues, and data-integrity risks.

Bandit was also used as an automated static security analyzer. Bandit did not report any security issues in the reviewed code. Therefore, the findings below are based on manual inspection and are mainly related to robustness, input validation, data integrity, and secure coding practices.

## 2. Summary of Findings

| ID | Finding | Severity | Type | Status |
|---|---|---|---|---|
| F-01 | Limited text input validation | Low | Input Validation | Improvement recommended |
| F-02 | File operations have limited error handling | Low | Error Handling / Data Integrity | Improvement recommended |
| F-03 | Expense IDs are generated using list length | Low | Data Integrity | Improvement recommended |
| F-04 | Description can be empty | Low | Input Validation | Improvement recommended |
| F-05 | JSON file is stored without protection | Low | Data Protection | Improvement recommended |
| F-06 | No backup/recovery mechanism | Low | Availability / Data Integrity | Improvement recommended |

## 3. Detailed Findings

### F-01 – Limited Text Input Validation

The program validates that the category is not empty, but it does not enforce a maximum length or additional validation for category and description fields.

Example:

```python
catagory = input("Enter Catagory(Food,Entertainment etc): ").strip()
discription = input("Enter Discription: ").strip()
```

### Risk

Very long or unexpected input can make the stored JSON data unnecessarily large and can make the application less predictable.

### Recommendation

Apply reasonable length limits before storing user input.

Example:

```python
if len(catagory) > 50:
    print("Category is too long.")
```

and:

```python
if len(discription) > 200:
    print("Description is too long.")
```

### Status

Recommended improvement.

---

### F-02 – Limited File Error Handling

The `load_expences()` method handles invalid JSON, but the `save_expenses()` method does not handle possible file-system errors.

Current code:

```python
with open(self.filename,"w") as file:
    json.dump(self.expences,file,indent=4)
```

### Risk

If the file cannot be written because of a permission problem, unavailable storage, or another operating-system error, the application can terminate unexpectedly.

### Recommendation

Handle file-writing errors using `try/except`.

Example:

```python
def save_expenses(self):
    try:
        with open(self.filename, "w") as file:
            json.dump(self.expences, file, indent=4)
    except OSError as e:
        print(f"Error saving expenses: {e}")
```

### Status

Recommended improvement.

---

### F-03 – Expense ID Generation

The application creates an ID using:

```python
"ID": len(self.expences["expence"]) + 1
```

### Risk

If an expense is deleted, the list length decreases. A future expense can therefore receive an ID that was already used.

For example:

```text
ID 1
ID 2
ID 3
```

After deleting ID 2:

```text
ID 1
ID 3
```

The next generated ID can become 3, creating a duplicate ID.

### Recommendation

Generate the next ID from the highest existing ID.

Example:

```python
new_id = max(
    (expense["ID"] for expense in self.expences["expence"]),
    default=0
) + 1
```

### Status

Recommended improvement for data integrity.

---

### F-04 – Description Validation

The category is checked for empty input, but the description is accepted without checking.

Current code:

```python
discription = input("Enter Discription: ").strip()
```

### Risk

An empty description can be stored. This is not a major security vulnerability, but stronger validation improves data quality.

### Recommendation

Require a non-empty description or explicitly allow an empty description.

Example:

```python
if not discription:
    print("Description cannot be empty.")
```

A maximum length can also be applied.

### Status

Recommended improvement.

---

### F-05 – Local JSON Data Protection

The application stores expense information in a local JSON file:

```python
with open(self.filename, "w") as file:
    json.dump(self.expences, file, indent=4)
```

### Risk

Anyone who has access to the user's computer and the JSON file may be able to read the stored expense information.

This is a limitation of the current project design rather than a vulnerability detected by Bandit.

### Recommendation

For a larger or multi-user application, consider access controls, protected application storage, or an appropriate database with authentication and authorization.

For this student project, documenting the limitation is sufficient.

### Status

Documented limitation.

---

### F-06 – No Backup or Recovery Mechanism

The application uses a single JSON file for storing expenses.

### Risk

If the file becomes corrupted or is accidentally deleted, the stored expense data may be lost.

The application already handles invalid JSON by creating an empty expense structure, but this does not recover the previous data.

### Recommendation

Create periodic backups before overwriting the main data file.

For example, a production application could maintain timestamped backups or use a database with backup/recovery support.

### Status

Recommended improvement.

---
## 4. Remediation Plan

The following improvements are recommended:

1. Add maximum length validation for category and description.
2. Improve file-write exception handling.
3. Generate unique expense IDs reliably.
4. Validate description input.
5. Consider appropriate protection for stored financial information.
6. Implement a backup/recovery mechanism.

## 5. Conclusion

The Expense Tracker was reviewed using both automated static analysis and manual inspection.

Bandit reported no security issues in the reviewed source code. Manual inspection identified several low-risk improvements related to input validation, file error handling, data integrity, data protection, and backup/recovery.

The review demonstrates that secure coding is not limited to automated vulnerability scanners. Manual inspection is also important for identifying application-specific risks and improving the reliability and security of software.

**Review Type:** Educational / Internship Project  
**Application:** Daily Expense Tracker  
**Language:** Python  
**Automated Tool:** Bandit
