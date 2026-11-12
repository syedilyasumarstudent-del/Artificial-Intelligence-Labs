## Exercise 1: Degree Honors Rule

### Objective
Add a new graduation honors rule to the `UNIVERSITY_KB` Knowledge Base.

**Conditions:**
- CGPA must be greater than or equal to 3.70.
- Disciplinary warnings must equal 0.

If both conditions are satisfied, the student receives the status **"Graduated with Magna Cum Laude Honors"**.

### Step 1: Adding the Rule to the Knowledge Base

```python
# EXERCISE 1: DEGREE HONORS RULE
# Adds a graduation honors rule to the Knowledge Base.

honors_rule = {
    "id": "RULE_07",
    "conditions": {
        "cgpa_min": 3.70,
        "disciplinary_warnings": 0
    },
    "condition_type": "degree_honors",
    "status": "Graduated with Magna Cum Laude Honors",
    "action": "Student qualifies for Magna Cum Laude graduation honors."
}

# Add the new rule to the Knowledge Base
UNIVERSITY_KB["rules"].append(honors_rule)

print("New rule added successfully!")
print(honors_rule)
```

### Step 2: Testing the Degree Honors Rule

```python
# TESTING EXERCISE 1: DEGREE HONORS

student = {
    "student_name": "Ali Khan",
    "cgpa": 3.85,
    "disciplinary_warnings": 0
}

# Check graduation honors eligibility
if (student["cgpa"] >= 3.70 and
        student["disciplinary_warnings"] == 0):

    print("Status: Graduated with Magna Cum Laude Honors")
    print("Congratulations! You qualify for graduation honors.")

else:
    print("Status: Not Eligible for Magna Cum Laude Honors")
```

### Expected Output

```text
New rule added successfully!
{'id': 'RULE_07', 'conditions': {'cgpa_min': 3.7, 'disciplinary_warnings': 0}, 'condition_type': 'degree_honors', 'status': 'Graduated with Magna Cum Laude Honors', 'action': 'Student qualifies for Magna Cum Laude graduation honors.'}

Status: Graduated with Magna Cum Laude Honors
Congratulations! You qualify for graduation honors.
```

### Explanation

1. `honors_rule` defines a new graduation honors rule.
2. `UNIVERSITY_KB["rules"].append(honors_rule)` adds the rule to the existing Knowledge Base.
3. `cgpa >= 3.70` checks whether the student's CGPA meets the minimum requirement.
4. `disciplinary_warnings == 0` checks whether the student has any disciplinary warnings.
5. The `and` operator ensures that both conditions must be true.

**Result:** The student with a CGPA of 3.85 and zero disciplinary warnings successfully qualifies for Magna Cum Laude graduation honors.
## Exercise 2: Tracing Forward Chaining

### Objective
Modify the `evaluate_student()` function to print each evaluated rule condition as the Inference Engine processes the student's record.

This demonstrates the step-by-step logical deduction process of a Knowledge-Based System.

### Step 1: Modified Inference Engine

```python

# EXERCISE 2: TRACING FORWARD CHAINING
# Prints each rule as it is evaluated.


def evaluate_student(student_record, kb):

    print(f"\n--- EVALUATING STUDENT: {student_record['student_name']} ---")

    # Check attendance rule first
    print("\nChecking RULE_01: Attendance Requirement")

    if student_record["attendance"] < 75:
        print("Condition matched: Attendance is below 75%.")

        attendance_rule = next(
            r for r in kb["rules"]
            if r["condition_type"] == "attendance_below"
        )

        print("RULE_01 FIRED!")
        return {
            "rule_id": attendance_rule["id"],
            "grade": attendance_rule["grade"],
            "status": attendance_rule["status"],
            "action": attendance_rule["action"]
        }

    print("Condition not matched. Attendance is sufficient.")

    # Check marks range rules
    for rule in kb["rules"]:

        if rule["condition_type"] == "marks_range":

            print(f"\nChecking {rule['id']}:")

            marks = student_record["total_marks"]
            minimum = rule["conditions"]["marks_min"]
            maximum = rule["conditions"]["marks_max"]

            print(f"Checking if {minimum} <= {marks} <= {maximum}")

            if minimum <= marks <= maximum:

                print(f"{rule['id']} FIRED!")
                print("Condition matched successfully.")

                return {
                    "rule_id": rule["id"],
                    "grade": rule["grade"],
                    "status": rule["status"],
                    "action": rule["action"]
                }

            else:
                print("Condition not matched.")

    print("\nNo matching rule found.")

    return {
        "rule_id": "UNKNOWN",
        "grade": "N/A",
        "status": "Invalid Score Entry",
        "action": "Verify submitted marks with the exam department."
    }
```

### Step 2: Testing the Modified Inference Engine

```python
# Test Case 1: High scorer with full attendance

student_1 = {
    "student_id": "BSCS-2024-012",
    "student_name": "Ali Khan",
    "total_marks": 88,
    "attendance": 92
}

# Test Case 2: Good marks but insufficient attendance

student_2 = {
    "student_id": "BSCS-2024-045",
    "student_name": "Sara Ahmed",
    "total_marks": 81,
    "attendance": 68
}

# Run the Inference Engine

result_1 = evaluate_student(student_1, UNIVERSITY_KB)

print("\nFinal Result:")
print(f"Rule: {result_1['rule_id']}")
print(f"Grade: {result_1['grade']}")
print(f"Status: {result_1['status']}")
print(f"Action: {result_1['action']}")

result_2 = evaluate_student(student_2, UNIVERSITY_KB)

print("\nFinal Result:")
print(f"Rule: {result_2['rule_id']}")
print(f"Grade: {result_2['grade']}")
print(f"Status: {result_2['status']}")
print(f"Action: {result_2['action']}")
```

### Step 3: Expected Output

**Student 1: Ali Khan**

```text
--- EVALUATING STUDENT: Ali Khan ---

Checking RULE_01: Attendance Requirement
Condition not matched. Attendance is sufficient.

Checking RULE_02:
Checking if 85 <= 88 <= 100
RULE_02 FIRED!
Condition matched successfully.

Final Result:
Rule: RULE_02
Grade: A
Status: Passed with Distinction
Action: Recommended for Merit Scholarship.
```

**Student 2: Sara Ahmed**

```text
--- EVALUATING STUDENT: Sara Ahmed ---

Checking RULE_01: Attendance Requirement
Condition matched: Attendance is below 75%.
RULE_01 FIRED!

Final Result:
Rule: RULE_01
Grade: F
Status: Debarred
Action: Ineligible to sit for final exam due to short attendance.
```

### Explanation

1. The Inference Engine first checks the attendance requirement.
2. If attendance is below 75%, RULE_01 fires immediately.
3. If attendance is sufficient, the system proceeds to check the marks range rules.
4. Each rule prints its conditions and whether they match.
5. When a matching rule is found, it fires and returns the result.
6. The system stops evaluating further rules after finding a match.

**Result:** The Inference Engine now displays its rule evaluation process, making the logical deduction steps visible.
