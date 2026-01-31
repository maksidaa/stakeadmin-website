# Stake Admin App — Permissions & Roles

**Version:** 1.0  
**Last Updated:** January 13, 2026  
**Status:** Ready for Development

---

## Overview

The app uses a **calling-based permission system**. A user's calling determines everything they can see and do. There are no separate "roles" to assign — permissions flow entirely from the calling.

---

## Permission Groups

### PO (Presidency Office)

**Members:**
- Stake President
- Stake Presidency 1st Counselor
- Stake Presidency 2nd Counselor
- Stake Executive Secretary
- Stake Assistant Executive Secretary 1
- Stake Assistant Executive Secretary 2

**Capabilities:**
- Full access to all features
- Can see all data across all units
- Can perform bulk operations (e.g., "Set All HC to Yes")
- Can manage users and stake settings
- Can add/edit/delete assignments and configurations

### HC (High Council)

**Members:**
- HC#1 through HC#12

**Capabilities:**
- Access to Stake Council section
- Can view and act on items assigned to them
- Can sustain (their own vote only)
- Can record interviews and action items
- Cannot see SP-only notes or perform PO bulk actions

### SC (Stake Council)

**Members:**
- Stake Relief Society Presidency + Secretary
- Stake Young Women Presidency + Secretary
- Stake Young Men Presidency + Secretary
- Stake Primary Presidency + Secretary
- Stake Sunday School Presidency + Secretary

**Capabilities:**
- Access to Stake Council section
- Can view SC-scoped action items
- Can view speaking assignments
- Can view meeting agendas
- Limited calling tracker visibility

### Unit Leaders

**Members:**
- Bishops / Branch Presidents
- Bishopric Counselors
- Ward/Branch Executive Secretaries
- Ward/Branch Clerks

**Capabilities:**
- Access to Directory section only
- Can submit calling/release requests
- Can submit MP ordination requests
- Can view unit info and resources
- **Cannot** access Stake Council section

### Unit Auxiliary

**Members:**
- Elders Quorum Presidents
- Relief Society Presidents
- Young Women Presidents
- Primary Presidents
- Sunday School Presidents

**Capabilities:**
- Access to Directory section only
- Can view unit info and resources
- **Cannot** submit callings or access Stake Council

---

## Feature Permission Matrix

### Stake Council Section Access

| Feature | PO | HC | SC | SC Counselor | Unit Leader | Unit Aux |
|---------|----|----|----|----|-------------|----------|
| **Access Stake Council Tab** | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ |

### Calling Tracker

| Action | PO | HC | SC | Unit Leader | Unit Aux |
|--------|----|----|----|----|----------|
| View all submissions | ✓ | After HC stage | ✗ | ◐ | ✗ |
| Submit calling/release | ✓ | ✓ | ✓ | ✓ | ✗ |
| SP Review & Configure | ✓ | ✗ | ✗ | ✗ | ✗ |
| SP Sustain (vote) | SP only | ✗ | ✗ | ✗ | ✗ |
| Send to HC | ✓ | ✗ | ✗ | ✗ | ✗ |
| HC Sustain (own vote) | ✓ | ✓ | ✗ | ✗ | ✗ |
| "Set All HC to Yes" | ✓ | ✗ | ✗ | ✗ | ✗ |
| Assign to SP/HC | ✓ | ✗ | ✗ | ✗ | ✗ |
| Change assignment | ✓ | Own only | ✗ | ✗ | ✗ |
| Mark extended/accepted | ✓ | Assigned | ✗ | ✗ | ✗ |
| View All Stake Business | ✓ | ✗ | ✗ | ✗ | ✗ |
| Mark sustained in unit | ✓ | ✓ | ✗ | ✓ | ✗ |
| Mark set apart | ✓ | If authorized | ✗ | ✗ | ✗ |
| Record in LCR | ✓ | ✗ | ✗ | ✗ | ✗ |
| Delete submission | ✓ | ✗ | ✗ | ✗ | ✗ |

**◐ Unit Leader:** Can only see submissions for their unit when "Allow Bishop to Track" is enabled.

### Ministering Interviews

| Action | PO | HC | SC | Unit Leader |
|--------|----|----|----|----|
| View all interviews | ✓ | ✓ | ✓ | ✗ |
| Record interview | ✓ | ✓ | ✓ | ✗ |
| Edit interview | ✓ | ✓ | ✓ | ✗ |
| Add assignment | ✓ | ✗ | ✗ | ✗ |
| Edit assignment | ✓ | ✗ | ✗ | ✗ |
| Delete assignment | ✓ | ✗ | ✗ | ✗ |
| Delete interview record | ✓ | ✗ | ✗ | ✗ |

### Action Items

| Action | PO | HC | SC | Unit Leader |
|--------|----|----|----|----|
| View SP action items | SP only | ✗ | ✗ | ✗ |
| View HC action items | ✓ | ✓ | ✗ | ✗ |
| View SC action items | ✓ | ✓ | ✓ | ✗ |
| Create SP action item | SP only | ✗ | ✗ | ✗ |
| Create HC action item | ✓ | ✓ | ✗ | ✗ |
| Create SC action item | ✓ | ✓ | ✓ | ✗ |
| Edit/Complete any | ✓ | Own scope | Own scope | ✗ |
| Delete | ✓ | ✗ | ✗ | ✗ |

### Meetings & Agendas

| Action | PO | HC | SC | SC Counselor |
|--------|----|----|----|----|
| View HC meeting agenda | ✓ | ✓ | ✗ | ✗ |
| View SC meeting agenda | ✓ | ✓ | ✓ | ✓ |
| Edit agenda | ✓ | ✗ | ✗ | ✗ |
| Take attendance | ✓ | ✗ | ✗ | ✗ |
| Add minutes | ✓ | ✗ | ✗ | ✗ |
| Archive meeting | ✓ | ✗ | ✗ | ✗ |

### Speaking Assignments

| Action | PO | HC | SC | Unit Leader | Unit Aux |
|--------|----|----|----|----|----------|
| View all assignments | ✓ | ✓ | ✓ | ✓ | ✓ |
| Edit assignments | ✓ | ✗ | ✗ | ✗ | ✗ |
| Send reminders | ✓ | ✗ | ✗ | ✗ | ✗ |

### MP Ordinations

| Action | PO | HC | SC | Unit Leader |
|--------|----|----|----|----|
| View all ordinations | ✓ | ✓ | ✗ | Own unit |
| Submit ordination | ✓ | ✓ | ✗ | ✓ |
| SP Review & Assign | ✓ | ✗ | ✗ | ✗ |
| Conduct interview | SP only | ✗ | ✗ | ✗ |
| HC Sustain | ✓ | ✓ | ✗ | ✗ |
| Record ordination | ✓ | Assigned | ✗ | ✗ |

### Directory Section

| Feature | PO | HC | SC | Unit Leader | Unit Aux |
|---------|----|----|----|----|----------|
| View all | ✓ | ✓ | ✓ | ✓ | ✓ |
| Edit unit info | ✓ | ✗ | ✗ | ✗ | ✗ |
| Edit resources | ✓ | ✗ | ✗ | ✗ | ✗ |
| Add training docs | ✓ | ✗ | ✗ | ✗ | ✗ |

### User Management

| Action | PO | HC | SC | Unit Leader |
|--------|----|----|----|----|
| View user list | ✓ | ✗ | ✗ | ✗ |
| Approve new users | ✓ | ✗ | ✗ | ✗ |
| Edit user calling | ✓ | ✗ | ✗ | ✗ |
| Grant/revoke access | ✓ | ✗ | ✗ | ✗ |
| Delete user | ✓ | ✗ | ✗ | ✗ |

### Stake Settings

| Action | PO | Others |
|--------|----|----|
| Edit stake name/logo | ✓ | ✗ |
| Edit vision statement | ✓ | ✗ |
| Edit Zoom links | ✓ | ✗ |
| Manage units | ✓ | ✗ |

---

## Visibility Rules

### Calling Tracker Visibility by Stage

| Stage | Who Can See |
|-------|-------------|
| SP Initial Review | PO only |
| SP Sustaining | PO only |
| HC Review | PO + HC |
| Ready for Assignment | PO + HC |
| Assigned | PO + HC + Assigned person |
| Pre-Sustaining | PO |
| Ward Sustaining | PO + Unit bishopric |
| Set Apart | PO + HC (if authorized) + Unit |
| Recording | PO + Clerks |
| Complete | Archived, PO can search |

### "Allow Bishop to Track" Flag

When enabled on a submission:
- The Bishop of that unit can see the submission
- They see current status and progress
- They **cannot** take actions (just visibility)

### Action Item Scope Visibility

| Scope | Visible To |
|-------|------------|
| SP | Stake President, SP1, SP2 only |
| HC | SP + HC members |
| SC | SP + HC + All Stake Council members |

---

## Implementation Notes

### Checking Permissions in Code

```typescript
// Example permission check
function canUserAccessFeature(user: User, feature: string): boolean {
  const calling = user.calling;
  const permissionLevel = calling.permission_level;
  
  // PO has access to everything
  if (['stake_presidency', 'executive_secretary'].includes(permissionLevel)) {
    return true;
  }
  
  // Feature-specific checks
  switch (feature) {
    case 'calling_tracker':
      return ['high_council', 'stake_council'].includes(permissionLevel);
    case 'ministering_interviews':
      return ['high_council', 'stake_council'].includes(permissionLevel);
    case 'sp_action_items':
      return permissionLevel === 'stake_presidency';
    // etc.
  }
}
```

### UI Visibility

- **Hide entire tabs/sections** user cannot access
- **Hide action buttons** user cannot perform
- **Show read-only views** where user has view but not edit access
- **Never show data** from restricted scopes (e.g., SP notes to HC)

### API Security

- **Validate permissions on every API call**
- **Filter query results** by user's permission level
- **Return 403 Forbidden** for unauthorized actions
- **Audit log** sensitive actions (deletes, permission changes)

---

## Special Cases

### HC Member Who Is Also Org President

Example: HC#4 Chad Moss is also Stake YM President

- Gets **HC permissions** (can sustain, view tracker, etc.)
- Gets **monthly interviews** like other org presidents
- Does **not** get duplicate permissions — HC level covers everything

### Executive Secretary Permissions

- Full PO access for operational functions
- **Cannot** vote on SP sustaining (only SP, SP1, SP2 can)
- **Can** perform bulk operations ("Set All HC to Yes")
- **Can** manage users and assignments

### Clerk Permissions

- Can access recording functions
- Can view calling tracker for recording purposes
- **Cannot** modify workflow or assignments
- Primarily focused on LCR recording tasks
