# Implement Client Script & UI Policy (Incident)

## Project Overview

This project demonstrates the implementation of **Client Scripts** and **UI Policies** in ServiceNow Incident Management.

The project improves Incident data quality by controlling field behavior, automatically updating values, enforcing mandatory fields, and preventing invalid changes. It uses **UI Policies**, **UI Policy Actions**, and Client Scripts such as `onChange`, `onSubmit`, and `onCellEdit`.

## Problem Statement

Incident records require consistent and accurate data entry for effective triage, routing, and resolution.

Relying only on manual checks can result in:

- Incomplete Incident records
- Incorrect or inconsistent data
- Missing mandatory information
- Poor reporting accuracy
- Potential SLA compliance issues
- Reduced service quality

To address these challenges, conditional field behavior and validation are implemented at the ServiceNow user-interface level.

## Objective

The objective is to demonstrate how ServiceNow **UI Policies** and **Client Scripts** can enforce data integrity on Incident records.

The implementation:

- Makes fields mandatory based on conditions
- Automatically populates field values
- Controls field behavior
- Makes fields read-only when required
- Validates Incident records before saving
- Prevents unauthorized list edits
- Improves data consistency

## Skills Demonstrated

- Incident Management
- UI Policy
- UI Policy Actions
- Client Scripts
- Form Validation
- `onChange` Client Script
- `onSubmit` Client Script
- `onCellEdit` Client Script

---

# Task 1: Create UI Policy on Incident

## UI Policy: High Impact Control

A UI Policy named **High Impact Control** is created for the Incident table.

| Property | Value |
|---|---|
| Name | High Impact Control |
| Table | Incident |
| Active | True |
| Field | Impact |
| Operator | is |
| Value | 1 - High |
| Reverse if false | True |

When Impact is set to High, the UI Policy applies the configured field controls.

### Steps

1. Log in to the ServiceNow instance.
2. Navigate to **System UI → UI Policies**.
3. Click **New**.
4. Enter the configuration shown above.
5. Configure the condition as **Impact is 1 - High**.
6. Configure the required field behavior.
7. Set **Reverse if false** to **True**.
8. Click **Submit**.

---

# Task 2: Create UI Policy Action – Urgency

A UI Policy Action is created under **High Impact Control**.

| Property | Value |
|---|---|
| Field Name | Urgency |
| Read-only | True |
| Visible | Leave unchanged |

When an Incident has High Impact, the **Urgency** field becomes read-only.

### Verification

1. Open an Incident.
2. Set **Impact** to **High**.
3. Verify that **Urgency** becomes read-only.
4. Change Impact to another value.
5. Verify that the field behavior is reversed.

---

# Task 3: Create onChange Client Script

The `onChange` Client Script automatically sets Urgency to High when Impact changes to High.

## Configuration

| Property | Value |
|---|---|
| Name | Auto set urgency for high impact |
| Table | Incident |
| Type | onChange |
| Field Name | Impact |
| Active | True |

## Script

```javascript
function onChange(control, oldValue, newValue, isLoading) {
    if (isLoading || newValue == '') {
        return;
    }

    if (newValue == '1') {
        g_form.setValue('urgency', '1');
        g_form.addInfoMessage('Urgency set to High for High impact incident.');
    }
}
```

### Expected Result

When **Impact** is changed to **High**, **Urgency** is automatically set to High and an informational message is displayed.

---

# Task 4: Create onSubmit Client Script

The `onSubmit` Client Script validates the **Assigned To** field before saving a High Impact Incident.

## Configuration

| Property | Value |
|---|---|
| Name | Prevent save if Assigned To missing |
| Table | Incident |
| Type | onSubmit |
| Active | True |

## Script

```javascript
function onSubmit() {
    if (g_form.getValue('impact') == '1' &&
        g_form.getValue('assigned_to') == '') {

        g_form.showErrorBox(
            'assigned_to',
            'Assigned To is mandatory for High impact incidents.'
        );

        return false;
    }

    return true;
}
```

### Expected Result

If **Impact = High** and **Assigned To** is empty, the Incident should not be saved and an error message should be displayed.

If Assigned To is populated, the Incident can be saved.

---

# Task 5: Create onCellEdit Client Script

The `onCellEdit` Client Script prevents users from changing the **State** field directly from an Incident list.

## Configuration

| Property | Value |
|---|---|
| Name | Prevent state change via list edit |
| Table | Incident |
| Type | onCellEdit |
| Field Name | State |
| Active | True |

## Script

```javascript
function onCellEdit(sysIDs, table, oldValues, newValue, callback) {

    alert('State cannot be updated using list editing. Please open the Incident.');

    callback(false);
}
```

### Expected Result

When a user attempts to edit State directly from the Incident list:

1. An alert message is displayed.
2. The edit is blocked.
3. The State value remains unchanged.

State changes can still be performed through the Incident form.

---

# Task 6: Testing the Configuration

## Test 1: Mandatory Enforcement

1. Navigate to **Incident → Create New**.
2. Set **Impact** to **High**.
3. Leave **Assigned To** empty.
4. Click **Submit**.

### Expected Result

The Incident should not be saved because Assigned To is required for High Impact incidents.

---

## Test 2: Successful Save

1. Open an Incident form.
2. Select a user in **Assigned To**.
3. Keep Impact as High.
4. Click **Submit**.

### Expected Result

The Incident should save successfully.

Verify:

- Urgency is automatically set to High.
- Urgency follows the configured read-only behavior.
- Assigned To validation is satisfied.
- State restrictions work for list editing.

---

## Test 3: Reverse Condition

1. Open an Incident where Impact is High.
2. Change Impact from **High** to **Medium**.
3. Verify the field behavior.

### Expected Result

- Assigned To should no longer be mandatory.
- Urgency should become editable.
- The UI Policy Action should be reversed.

---

## Test 4: List Edit Blocking

1. Navigate to **Incident → All**.
2. Locate an Incident.
3. Double-click the **State** field.
4. Attempt to change State.

### Expected Result

The following alert should appear:

```text
State cannot be updated using list editing. Please open the Incident.
```

The State value should remain unchanged.

---

## Test 5: Form-Based Update

1. Open the Incident record.
2. Change the **State** field.
3. Click **Update**.

### Expected Result

The State change should be saved successfully through the form.

---

# Configuration Summary

| Component | Configuration |
|---|---|
| UI Policy | High Impact Control |
| Table | Incident |
| Condition | Impact is 1 - High |
| Reverse if false | True |
| UI Policy Action | Urgency |
| Urgency Behavior | Read-only |
| onChange Script | Auto set urgency for high impact |
| onSubmit Script | Prevent save if Assigned To missing |
| onCellEdit Script | Prevent state change via list edit |
| Validation | Assigned To required for High Impact |
| Automatic Update | Urgency set to High |
| List Restriction | State cannot be changed through list editing |

---

# Testing Checklist

- [ ] High Impact Control UI Policy created
- [ ] UI Policy is Active
- [ ] Impact condition is set to High
- [ ] Reverse if false is enabled
- [ ] Required field behavior is configured
- [ ] Urgency UI Policy Action created
- [ ] Urgency becomes read-only for High Impact
- [ ] `onChange` Client Script created
- [ ] Urgency automatically changes to High
- [ ] `onSubmit` Client Script created
- [ ] Assigned To validation works
- [ ] High Impact Incident cannot be saved without Assigned To
- [ ] `onCellEdit` Client Script created
- [ ] State list editing is blocked
- [ ] Form-based State update works
- [ ] Reverse condition works correctly
- [ ] Successful Incident save verified

---

# Benefits

The implementation provides:

- **Improved Data Quality:** Required fields and validation help maintain complete Incident records.
- **Consistent Incident Handling:** Standardized behavior is applied based on Impact.
- **Automated Field Updates:** Urgency is automatically updated when Impact is High.
- **Improved Validation:** Incomplete High Impact Incidents are prevented from being saved.
- **Controlled List Editing:** Direct State changes from the list are blocked.
- **Better User Experience:** Dynamic field behavior provides immediate feedback.

---

# Technologies Used

- **ServiceNow**
- **Incident Management**
- **UI Policies**
- **UI Policy Actions**
- **Client Scripts**
- **JavaScript**
- **Form Validation**

---

# Project Structure

```text
Implement Client Script & UI Policy
│
├── UI Policy
│   └── High Impact Control
│
├── UI Policy Action
│   └── Urgency - Read Only
│
├── Client Scripts
│   ├── Auto set urgency for high impact
│   ├── prevent save if Assigned To missing
│   └── Prevent state change via list edit
│
└── Testing
    ├── Mandatory Enforcement
    ├── Successful Save
    ├── Reverse Condition
    ├── List Edit Blocking
    └── Form-Based Update
```

---

# Conclusion

The **Implement Client Script & UI Policy (Incident)** project demonstrates how ServiceNow UI Policies and Client Scripts can work together to enforce dynamic field behavior, automate updates, and prevent incorrect Incident submissions.

By using `onChange`, `onSubmit`, and `onCellEdit` Client Scripts together with UI Policy Actions, the solution improves Incident data consistency, enforces required information, automates field updates, and controls user interactions.

This provides a lightweight and practical approach to improving Incident form usability and data integrity in ServiceNow.

---

## Project Information

**Project Name:** Implement Client Script & UI Policy (Incident)

**Platform:** ServiceNow

**Module:** Incident Management

**Primary Features:** UI Policy, UI Policy Actions, Client Scripts, Form Validation

**Client Script Types:** onChange, onSubmit, onCellEdit
