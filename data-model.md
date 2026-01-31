# Stake Admin App — Data Model

**Version:** 1.0  
**Last Updated:** January 13, 2026  
**Status:** Ready for Development

---

## Overview

This document defines all database tables for the Stake Admin App. It serves as the single source of truth for the data model across all features.

**Database:** PostgreSQL (via Supabase)  
**Multi-tenancy:** Row-level security using `stake_id` on all tables

---

## Core Tables

### stakes

Top-level workspace. Each stake is completely isolated.

```sql
CREATE TABLE stakes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  logo_url TEXT,
  vision_statement TEXT,
  zoom_link_hc TEXT,
  zoom_link_sc TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### units

Wards and branches within a stake.

```sql
CREATE TABLE units (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  unit_number INTEGER NOT NULL,
  name VARCHAR(255) NOT NULL,
  unit_type VARCHAR(20) NOT NULL CHECK (unit_type IN ('ward', 'branch')),
  building_photo_url TEXT,
  address TEXT,
  latitude DECIMAL(10, 8),
  longitude DECIMAL(11, 8),
  sacrament_time VARCHAR(50),
  ward_council_schedule VARCHAR(255),
  youth_activities_schedule VARCHAR(255),
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(stake_id, unit_number)
);

CREATE INDEX idx_units_stake ON units(stake_id);
```

### callings_list

Reference table of all possible callings.

```sql
CREATE TABLE callings_list (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  category VARCHAR(50) NOT NULL,
  -- Categories: stake_presidency, high_council, stake_auxiliary, 
  --             unit_leadership, unit_auxiliary, clerk, other
  permission_level VARCHAR(50) NOT NULL,
  -- Permission levels: admin, stake_presidency, executive_secretary,
  --                    high_council, stake_council, stake_council_counselor,
  --                    unit_leader, unit_auxiliary, clerk
  display_order INTEGER,
  is_stake_level BOOLEAN DEFAULT TRUE,
  hc_number INTEGER CHECK (hc_number BETWEEN 1 AND 12),
  requires_monthly_interview BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### users

All app users with their calling and permissions.

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  email VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  phone VARCHAR(50),
  auth_provider VARCHAR(20) NOT NULL CHECK (auth_provider IN ('google', 'apple', 'email')),
  auth_provider_id VARCHAR(255),
  
  -- Calling & Assignment
  calling_id UUID REFERENCES callings_list(id),
  unit_id UUID REFERENCES units(id),
  hc_number INTEGER CHECK (hc_number BETWEEN 1 AND 12),
  
  -- Permissions
  is_approved BOOLEAN DEFAULT FALSE,
  is_admin BOOLEAN DEFAULT FALSE,
  can_edit BOOLEAN DEFAULT FALSE,
  
  -- Push Notifications
  push_token TEXT,
  push_enabled BOOLEAN DEFAULT TRUE,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(stake_id, email)
);

CREATE INDEX idx_users_stake ON users(stake_id);
CREATE INDEX idx_users_calling ON users(calling_id);
CREATE INDEX idx_users_unit ON users(unit_id);
```

---

## Calling Tracker Tables

### calling_submissions

Main table for calling, release, and combined submissions.

```sql
CREATE TABLE calling_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  
  -- Submission basics
  submission_type VARCHAR(20) NOT NULL CHECK (submission_type IN ('calling', 'release', 'calling_release')),
  person_name VARCHAR(255) NOT NULL,
  home_unit_id UUID REFERENCES units(id),
  
  -- Calling details (for callings)
  calling_type VARCHAR(50),
  -- Options: stake_calling, ward_calling, stake_assignment, ward_release
  new_calling VARCHAR(255),
  
  -- Release details
  current_calling VARCHAR(255),
  
  -- Submission info
  request_notes TEXT,
  bishop_is_aware BOOLEAN DEFAULT FALSE,
  org_president_consulted BOOLEAN DEFAULT FALSE,
  submitted_by_user_id UUID REFERENCES users(id),
  
  -- SP Configuration
  sp_notes TEXT,
  assign_calling_to_user_id UUID REFERENCES users(id),
  assign_release_to_user_id UUID REFERENCES users(id),
  assign_denied_to_user_id UUID REFERENCES users(id),
  sustain_in VARCHAR(50) DEFAULT 'home_unit',
  release_in VARCHAR(50) DEFAULT 'home_unit',
  allow_bishop_to_track BOOLEAN DEFAULT FALSE,
  set_apart_by VARCHAR(50),
  can_set_apart_by_hc BOOLEAN DEFAULT FALSE,
  on_hold BOOLEAN DEFAULT FALSE,
  
  -- SP Sustaining
  sp_sustained BOOLEAN DEFAULT FALSE,
  sp1_sustained BOOLEAN DEFAULT FALSE,
  sp2_sustained BOOLEAN DEFAULT FALSE,
  sp_sustaining_complete BOOLEAN DEFAULT FALSE,
  sp_sustaining_complete_at TIMESTAMPTZ,
  
  -- HC Sustaining
  hc_sustaining JSONB DEFAULT '{}',
  -- Stores: {"hc1": true, "hc2": false, ...}
  hc_sustaining_complete BOOLEAN DEFAULT FALSE,
  hc_sustaining_complete_at TIMESTAMPTZ,
  hc_notes TEXT,
  
  -- Extension Stage
  release_extended BOOLEAN,
  release_extended_at TIMESTAMPTZ,
  calling_extended BOOLEAN,
  calling_extended_at TIMESTAMPTZ,
  calling_accepted BOOLEAN,
  calling_accepted_at TIMESTAMPTZ,
  calling_declined BOOLEAN,
  calling_declined_at TIMESTAMPTZ,
  
  -- Ward Sustaining
  sent_to_unit_for_sustaining BOOLEAN DEFAULT FALSE,
  sent_to_unit_at TIMESTAMPTZ,
  sustained_in_unit BOOLEAN DEFAULT FALSE,
  sustained_in_unit_at TIMESTAMPTZ,
  
  -- Set Apart
  set_apart_complete BOOLEAN DEFAULT FALSE,
  set_apart_at TIMESTAMPTZ,
  set_apart_by_user_id UUID REFERENCES users(id),
  
  -- Recording
  release_recorded_in_lcr BOOLEAN DEFAULT FALSE,
  release_recorded_at TIMESTAMPTZ,
  sustaining_recorded_in_lcr BOOLEAN DEFAULT FALSE,
  sustaining_recorded_at TIMESTAMPTZ,
  recorded_by_user_id UUID REFERENCES users(id),
  
  -- Workflow Status
  status VARCHAR(50) NOT NULL DEFAULT 'submitted',
  -- Statuses: submitted, sp_review, sp_sustaining, hc_review, 
  --           ready_for_assignment, assigned, extension, declined, 
  --           pre_sustaining, ward_sustaining, set_apart, recording, complete
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_calling_submissions_stake ON calling_submissions(stake_id);
CREATE INDEX idx_calling_submissions_status ON calling_submissions(status);
CREATE INDEX idx_calling_submissions_assigned ON calling_submissions(assign_calling_to_user_id);
```

### unit_sustainings

Tracks sustaining completion for each unit (stake callings).

```sql
CREATE TABLE unit_sustainings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  submission_id UUID NOT NULL REFERENCES calling_submissions(id) ON DELETE CASCADE,
  unit_id UUID NOT NULL REFERENCES units(id),
  sustained BOOLEAN DEFAULT FALSE,
  sustained_by_user_id UUID REFERENCES users(id),
  sustained_at TIMESTAMPTZ,
  is_home_unit BOOLEAN DEFAULT FALSE,
  
  UNIQUE(submission_id, unit_id)
);

CREATE INDEX idx_unit_sustainings_submission ON unit_sustainings(submission_id);
```

### mp_ordinations

Melchizedek Priesthood ordination submissions.

```sql
CREATE TABLE mp_ordinations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  
  -- Submission basics
  person_name VARCHAR(255) NOT NULL,
  home_unit_id UUID REFERENCES units(id),
  proposed_office VARCHAR(20) NOT NULL CHECK (proposed_office IN ('elder', 'high_priest')),
  current_office VARCHAR(50),
  
  -- Prerequisites
  bishop_interview_complete BOOLEAN DEFAULT FALSE,
  lcr_record_created BOOLEAN DEFAULT FALSE,
  to_be_ordained_by VARCHAR(255),
  target_ordination_date DATE,
  bishop_comments TEXT,
  submitted_by_user_id UUID REFERENCES users(id),
  
  -- SP Assignment
  assigned_to_interview_user_id UUID REFERENCES users(id),
  sustain_in VARCHAR(50) DEFAULT 'home_unit',
  assigned_to_oversee_user_id UUID REFERENCES users(id),
  sp_comments TEXT,
  
  -- Interview
  stake_interview_complete BOOLEAN DEFAULT FALSE,
  stake_interview_date DATE,
  stake_interview_by_user_id UUID REFERENCES users(id),
  
  -- HC Sustaining
  hc_sustaining JSONB DEFAULT '{}',
  hc_sustaining_complete BOOLEAN DEFAULT FALSE,
  
  -- Sustaining
  sustained BOOLEAN DEFAULT FALSE,
  sustained_at TIMESTAMPTZ,
  sustained_in VARCHAR(50),
  
  -- Ordination
  ordained BOOLEAN DEFAULT FALSE,
  ordained_date DATE,
  ordained_by_name VARCHAR(255),
  ordained_by_record_number VARCHAR(50),
  ordained_by_birthdate DATE,
  ordained_by_priesthood_office VARCHAR(50),
  stake_representative_user_id UUID REFERENCES users(id),
  
  -- Recording
  ordination_recorded_in_lcr BOOLEAN DEFAULT FALSE,
  record_signed_by_sp BOOLEAN DEFAULT FALSE,
  certificate_printed BOOLEAN DEFAULT FALSE,
  
  -- Ratification
  needs_ratification BOOLEAN DEFAULT FALSE,
  ratified BOOLEAN DEFAULT FALSE,
  ratified_at TIMESTAMPTZ,
  
  -- Status
  status VARCHAR(50) NOT NULL DEFAULT 'submitted',
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_mp_ordinations_stake ON mp_ordinations(stake_id);
CREATE INDEX idx_mp_ordinations_status ON mp_ordinations(status);
```

---

## Ministering Interviews Tables

### interview_assignments

Defines who interviews whom and how often. Position-based, not person-based.

```sql
CREATE TABLE interview_assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  
  -- Position being interviewed
  interviewee_position VARCHAR(255) NOT NULL,
  interviewee_type VARCHAR(50) NOT NULL,
  -- Types: stake_presidency, high_council, stake_auxiliary, 
  --        bishop, bishopric_counselor, eqp, other
  interviewee_unit_id UUID REFERENCES units(id),
  
  -- Who conducts the interview
  interviewer_position VARCHAR(255) NOT NULL,
  is_external_interviewer BOOLEAN DEFAULT FALSE,
  
  -- Schedule
  frequency VARCHAR(20) NOT NULL DEFAULT 'quarterly',
  -- Options: monthly, quarterly, biannual, as_needed
  
  -- Settings
  is_active BOOLEAN DEFAULT TRUE,
  exclude_from_tracking BOOLEAN DEFAULT FALSE,
  notes TEXT,
  display_order INTEGER,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_interview_assignments_stake ON interview_assignments(stake_id);
CREATE INDEX idx_interview_assignments_interviewer ON interview_assignments(interviewer_position);
```

### interviews

Log of actual interviews conducted.

```sql
CREATE TABLE interviews (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  assignment_id UUID NOT NULL REFERENCES interview_assignments(id) ON DELETE CASCADE,
  interview_date TIMESTAMPTZ NOT NULL,
  conducted_by_user_id UUID REFERENCES users(id),
  recorded_by_user_id UUID REFERENCES users(id),
  notes TEXT,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_interviews_assignment ON interviews(assignment_id);
CREATE INDEX idx_interviews_date ON interviews(interview_date DESC);
```

---

## Action Items Tables

### action_items

Assignments from meetings with due dates and ownership.

```sql
CREATE TABLE action_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  
  scope VARCHAR(10) NOT NULL CHECK (scope IN ('sp', 'hc', 'sc')),
  -- sp = Stake Presidency only
  -- hc = High Council (SP + HC)
  -- sc = Stake Council (SP + HC + Auxiliaries)
  
  title VARCHAR(500) NOT NULL,
  description TEXT,
  source_meeting_tag VARCHAR(50),
  -- Examples: [SMC], [HC], [SP], [SC]
  
  due_date DATE NOT NULL,
  follow_up_user_id UUID REFERENCES users(id),
  
  status VARCHAR(20) NOT NULL DEFAULT 'open',
  completed_at TIMESTAMPTZ,
  
  notes TEXT,
  
  created_by_user_id UUID REFERENCES users(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_action_items_stake ON action_items(stake_id);
CREATE INDEX idx_action_items_scope ON action_items(scope);
CREATE INDEX idx_action_items_status ON action_items(status);
CREATE INDEX idx_action_items_due ON action_items(due_date);
```

### action_item_assignees

Junction table for multiple assignees per action item.

```sql
CREATE TABLE action_item_assignees (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  action_item_id UUID NOT NULL REFERENCES action_items(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id),
  
  UNIQUE(action_item_id, user_id)
);

CREATE INDEX idx_action_item_assignees_item ON action_item_assignees(action_item_id);
CREATE INDEX idx_action_item_assignees_user ON action_item_assignees(user_id);
```

---

## Meetings Tables

### meetings

High Council and Stake Council meetings with agendas.

```sql
CREATE TABLE meetings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  
  meeting_type VARCHAR(50) NOT NULL,
  -- Types: high_council, stake_council, presidency, other
  
  meeting_date DATE NOT NULL,
  
  -- Agenda fields
  opening_prayer_user_id UUID REFERENCES users(id),
  closing_prayer_user_id UUID REFERENCES users(id),
  hymn VARCHAR(100),
  discussion_leader_user_id UUID REFERENCES users(id),
  discussion_topic TEXT,
  ministering_experience_user_id UUID REFERENCES users(id),
  
  -- Content
  agenda_content TEXT,
  minutes TEXT,
  parking_lot_items TEXT,
  
  is_archived BOOLEAN DEFAULT FALSE,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_meetings_stake ON meetings(stake_id);
CREATE INDEX idx_meetings_date ON meetings(meeting_date DESC);
CREATE INDEX idx_meetings_type ON meetings(meeting_type);
```

### meeting_attendance

Tracks who attended each meeting.

```sql
CREATE TABLE meeting_attendance (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  meeting_id UUID NOT NULL REFERENCES meetings(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id),
  attended BOOLEAN DEFAULT FALSE,
  
  UNIQUE(meeting_id, user_id)
);

CREATE INDEX idx_meeting_attendance_meeting ON meeting_attendance(meeting_id);
```

---

## Speaking Assignments Tables

### speaking_assignments

Stake Council speaking assignment dates and topics.

```sql
CREATE TABLE speaking_assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  assignment_date DATE NOT NULL,
  topic TEXT,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_speaking_assignments_stake ON speaking_assignments(stake_id);
CREATE INDEX idx_speaking_assignments_date ON speaking_assignments(assignment_date);
```

### speaking_assignment_units

Per-unit speaker assignments for each date.

```sql
CREATE TABLE speaking_assignment_units (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  speaking_assignment_id UUID NOT NULL REFERENCES speaking_assignments(id) ON DELETE CASCADE,
  unit_id UUID NOT NULL REFERENCES units(id),
  speaker_user_id UUID REFERENCES users(id),
  reminder_sent BOOLEAN DEFAULT FALSE,
  
  UNIQUE(speaking_assignment_id, unit_id)
);

CREATE INDEX idx_speaking_assignment_units_assignment ON speaking_assignment_units(speaking_assignment_id);
```

### branch_speaking_assignments

Wards providing speakers to branches.

```sql
CREATE TABLE branch_speaking_assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  branch_id UUID NOT NULL REFERENCES units(id),
  providing_ward_id UUID NOT NULL REFERENCES units(id),
  month INTEGER NOT NULL CHECK (month BETWEEN 1 AND 12),
  year INTEGER NOT NULL,
  speaker_name VARCHAR(255),
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(stake_id, branch_id, month, year)
);

CREATE INDEX idx_branch_speaking_stake ON branch_speaking_assignments(stake_id);
```

---

## Directory & Resources Tables

### resources

Stake-specific resources and links.

```sql
CREATE TABLE resources (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  
  category VARCHAR(50) NOT NULL,
  -- Categories: general, training, welfare, temple, missionary, stake_council_tools
  
  title VARCHAR(255) NOT NULL,
  description TEXT,
  icon VARCHAR(50),
  
  resource_type VARCHAR(20) NOT NULL,
  -- Types: link, content, feature, pdf
  
  url TEXT,
  content TEXT,
  feature_route VARCHAR(100),
  
  display_order INTEGER,
  is_active BOOLEAN DEFAULT TRUE,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_resources_stake ON resources(stake_id);
CREATE INDEX idx_resources_category ON resources(category);
```

### training_documents

Training materials library.

```sql
CREATE TABLE training_documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  
  title VARCHAR(255) NOT NULL,
  description TEXT,
  
  document_type VARCHAR(20) NOT NULL,
  -- Types: link, pdf, content
  
  url TEXT,
  file_url TEXT,
  content TEXT,
  
  category VARCHAR(100),
  display_order INTEGER,
  
  created_by_user_id UUID REFERENCES users(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_training_documents_stake ON training_documents(stake_id);
```

### info_change_requests

User-submitted requests for app info updates.

```sql
CREATE TABLE info_change_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stake_id UUID NOT NULL REFERENCES stakes(id),
  
  submitted_by_user_id UUID REFERENCES users(id),
  description TEXT NOT NULL,
  screenshot_1_url TEXT,
  screenshot_2_url TEXT,
  
  status VARCHAR(20) NOT NULL DEFAULT 'pending',
  -- Statuses: pending, resolved, rejected
  
  admin_notes TEXT,
  resolved_by_user_id UUID REFERENCES users(id),
  resolved_at TIMESTAMPTZ,
  
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_info_change_requests_stake ON info_change_requests(stake_id);
CREATE INDEX idx_info_change_requests_status ON info_change_requests(status);
```

---

## Row-Level Security Policies

All tables use stake-based isolation:

```sql
-- Example RLS policy for users table
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

CREATE POLICY users_stake_isolation ON users
  USING (stake_id = current_setting('app.current_stake_id')::uuid);

-- Apply similar policies to all tables with stake_id
```

---

## Indexes Summary

Key indexes for performance:

| Table | Index | Purpose |
|-------|-------|---------|
| All tables | `stake_id` | Multi-tenant isolation |
| `calling_submissions` | `status` | Filter by workflow stage |
| `calling_submissions` | `assign_calling_to_user_id` | User's assigned items |
| `interviews` | `assignment_id, interview_date DESC` | Latest interview lookup |
| `action_items` | `scope, status, due_date` | Dashboard queries |
| `meetings` | `meeting_date DESC` | Chronological listing |

---

## Migration Notes

When migrating from Glide:

1. **Export all Glide tables** to CSV
2. **Create stakes record** first
3. **Import units** with unit_number mapping
4. **Import users** and assign callings
5. **Import feature data** (callings, interviews, etc.)
6. **Verify counts** match between systems
