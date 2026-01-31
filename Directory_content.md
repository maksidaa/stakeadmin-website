# Directory Section — Complete Specification

**Last Updated:** January 8, 2026  
**Status:** Complete

-----

## Overview

The Directory is the **main navigation hub** of the app and serves as the “public” section accessible to all authenticated users. While the Stake Council section (calling tracker, action items, agendas) is restricted to stake leadership, the Directory provides resources and tools for all unit leaders.

### Two-Tier Access Model

|Section          |Who Can Access                                      |What’s There                                                                 |
|-----------------|----------------------------------------------------|-----------------------------------------------------------------------------|
|**Directory**    |All approved users (Bishops, EQ Pres, RS Pres, etc.)|Unit info, resources, training, MP ordination submission, interview questions|
|**Stake Council**|SP, HC, Stake Council members only                  |Calling tracker, action items, agendas, speaking assignments                 |

-----

## Directory Home Screen

**Header:** Stake logo and name (customizable per stake)

### Navigation Structure

The Directory is a grid of feature tiles. Each tile leads to a sub-feature.

#### Feature Tiles (Standard Layout)

|Tile                                            |Icon|Purpose                             |Access          |
|------------------------------------------------|----|------------------------------------|----------------|
|**High Council Members**                        |👥   |View HC roster with assignments     |All users       |
|**High Council/Stake Council Meetings**         |👥   |Meeting schedules                   |SC+ only        |
|**Stake Council Speaking Assignments**          |👤   |SC members speaking in wards        |All users (view)|
|**Branch Speaking Assignments**                 |👤   |Wards providing speakers to branches|All users       |
|**General Resources**                           |ℹ️   |Resource library hub                |All users       |
|**Info Change Request**                         |✏️   |Request updates to app info         |All users       |
|**Temple Recommend Interview Questions**        |📋   |Interview question reference        |All users       |
|**Melchizedek Priesthood Ordination Submission**|🔑   |MP ordination submission form       |Unit leaders    |
|**Melchizedek Priesthood Tracker**              |⚙️   |MP ordination workflow tracker      |PO + HC         |
|**Calling and Release Tracker**                 |⚙️   |Main calling tracker (full-width)   |PO + HC         |
|**Priesthood Ordinances**                       |💙   |Ordinance reference cards           |All users       |
|**Missionary Interview Questions**              |👥   |Mission interview reference         |All users       |
|**Training Documents**                          |📄   |Training materials library          |All users       |

#### Admin-Only Indicators (Bottom of Directory)

|Item                                     |Purpose                            |
|-----------------------------------------|-----------------------------------|
|**New Sign-Ups to Review (X)**           |Count of pending user registrations|
|**Info Change Submissions to Review (X)**|Count of pending change requests   |

These only appear for users with admin privileges (“Can Edit App Data?” = true).

-----

## Feature Specifications

### 1. High Council Members

**Purpose:** Display the HC roster with their unit and auxiliary assignments.

**Header:** “Stake High Council Info”

#### Components

1. **Email All High Council Members** — Button to compose email to all 12 HC
1. **Email All Stake Presidency and High Council Members** — Button to compose email to SP + HC (15 people)
1. **HC Member List** — Scrollable list of all 12 members

#### HC Member Card

Each member displays:

- **“HIGH COUNCIL”** label (blue)
- **Name**
- **Assignments** — Their unit liaison + auxiliary oversight (comma-separated)
- **⋯ menu** — Actions (edit, contact)

#### Example Data

|HC#|Name           |Assignments                              |
|---|---------------|-----------------------------------------|
|1  |Chris Hernandez|Blanco Vista, YW                         |
|2  |Andrew Pinkston|Lockhart, Primary                        |
|3  |Dennis Carroll |Buda, Institute, LDSSA, & Pathways       |
|4  |Chad Moss      |Stake Young Men’s President              |
|5  |Ryan Bedwell   |Bastrop, Missionary Work & EnglishConnect|
|6  |Devin Bedwell  |Communications, Prison Ministry          |
|7  |Gene Massey    |San Marcos, Music and Activities         |
|8  |Stuart Lund    |Cedar Creek, Seminary                    |
|9  |Mark Wilcox    |San Marcos YSA, Self-Reliance            |
|10 |Nick Pace      |Plum Creek, Family History               |
|11 |Nathan Robb    |Kyle, Temple & Indexing                  |
|12 |[Name]         |[Assignments]                            |

#### Data Source

Auto-populated from users table where:

- `calling` = “High Council”
- `hc_number` = 1-12
- Assignments pulled from user profile fields

#### Permissions

- **View:** All authenticated users
- **Edit assignments:** SP only (via Edit Users screen)

-----

### 2. General Resources

**Purpose:** Hub for stake-specific resources, links, and information.

**Header:** Stake logo + name

#### Sub-Feature Tiles

|Tile                                |Icon|Content Type                                     |
|------------------------------------|----|-------------------------------------------------|
|**Unit Info**                       |📍   |Ward/branch details (building, times, directions)|
|**San Antonio Temple Info**         |🏠   |Temple-specific information                      |
|**Family Search Centers**           |👥   |FamilySearch center locations                    |
|**Welfare and Counseling Resources**|⭕   |Welfare contacts and resources                   |
|**Stake Specialists**               |👥   |List of stake specialists                        |
|**Missionary Contact Info**         |👥   |Mission/missionary contacts                      |
|**Important Links**                 |🔗   |Curated external links                           |
|**Stake Meetings Info**             |👥   |Meeting schedules and info                       |
|**Stake Goals**                     |🎯   |Current stake goals (full-width)                 |
|**Submit Stake Calling**            |➕   |Shortcut to calling submission (full-width)      |
|**Allow Push Notifications**        |🔔   |Device notification settings (full-width)        |

#### Admin Configurability

- Admins can add/edit/remove resource tiles
- Each tile can link to:
  - External URL
  - Internal content page
  - Another app feature

#### Unit Info Sub-Feature

When user taps “Unit Info,” they see a list of all units:

**Unit List View:**

- Grid of unit cards with building photos
- Unit name below each photo

**Unit Detail View:**

- Large building photo header
- Unit name and type (Ward/Branch)
- Address
- **“Get Directions”** button (opens Maps app)
- Meeting times:
  - Sacrament Meeting time
  - Ward Council schedule
  - Youth Activities schedule
- **Edit** button (if authorized)

-----

### 3. Priesthood Ordinances

**Purpose:** Quick reference cards for performing priesthood ordinances.

**Header:** “Priesthood Ordinances”

#### Ordinance List

Scrollable list of ordinance types:

1. Baptism
1. Confirmation and Gift of the Holy Ghost
1. Sacrament Prayers
1. Naming and Blessing Children
1. Conferring Aaronic Priesthood & Ordaining to an Office
1. Conferring Melchizedek Priesthood & Ordaining to an Office
1. Setting Apart Members to Callings (No Keys)
1. Setting Apart Bishops and Quorum Presidents (With Keys)
1. Consecrating Oil
1. Administering to the Sick
1. Dedicating a Grave
1. Dedicating a Home
1. Father’s Blessings / Blessings of Comfort

#### Ordinance Detail Card

Each ordinance card contains:

|Section                |Content                                                  |
|-----------------------|---------------------------------------------------------|
|**Can Be Performed By**|Who is authorized (e.g., “Priests (Aaronic Priesthood)”) |
|**INSTRUCTIONS**       |Brief procedural guidance                                |
|**WORDING**            |Full ordinance text with [placeholders] for names/offices|
|**Handbook Link**      |Button linking to relevant General Handbook section      |

#### Example: Sacrament Prayers

```
Can Be Performed By: Priests (Aaronic Priesthood)

INSTRUCTIONS:
Recite exact prayers from Moroni 4:3 (bread) and Moroni 5:2 (water).

WORDING:
Sacrament Prayer on the Bread (Moroni 4:3)
"O God, the Eternal Father, we ask thee in the name of thy Son, 
Jesus Christ, to bless and sanctify this bread..."

Sacrament Prayer on the Water (Moroni 5:2)
"O God, the Eternal Father, we ask thee in the name of thy Son, 
Jesus Christ, to bless and sanctify this water..."

[SCROLL TO SECTION 18.9]
[Handbook Link]
```

#### Example: Setting Apart Bishops (With Keys)

```
Can Be Performed By: Stake President (bishop and EQ Presidents); 
Bishop (Deacons and Teachers quorum presidents)

INSTRUCTIONS:
State office; confer keys and blessings.

WORDING:
[Full name], by the authority of the Melchizedek Priesthood, we set 
you apart as [office] in the [ward or stake], and confer upon you 
the keys of this office, with all the rights, responsibilities, and 
authority pertaining thereto.

A blessing may then be added as the Spirit directs.

In the name of Jesus Christ, amen.

[SCROLL TO SECTION 18.11]
[Handbook Link]
```

#### Data Model

```
priesthood_ordinances
├── id (uuid)
├── stake_id (foreign key) — NULL for global/default ordinances
├── name (string) — "Sacrament Prayers"
├── display_order (integer)
├── can_be_performed_by (text) — "Priests (Aaronic Priesthood)"
├── instructions (text)
├── wording (text) — supports [placeholder] syntax
├── handbook_section (string) — "18.9"
├── handbook_url (string) — full URL to handbook
└── is_custom (boolean) — true if stake-specific addition
```

#### Permissions

- **View:** All authenticated users
- **Edit:** None (static content from Handbook)
- **Add custom:** Admin only (for stake-specific guidance)

-----

### 4. Temple Recommend Interview Questions

**Purpose:** Reference for conducting temple recommend interviews.

**Header:** “Stake Temple Interview Schedule” / “Temple Recommend Questions”

#### Content

- Source reference: “From HB Section 26.3.3.1”
- **“Las Preguntas En Español”** button — Spanish version toggle
- Numbered list of all temple recommend interview questions

#### Content Type

**Static** — These questions come from the General Handbook and should not be editable. The app displays them for convenient reference during interviews.

#### Multi-Language Support

- English (default)
- Spanish (toggle button)
- Consider: Other languages based on stake demographics

-----

### 5. Missionary Interview Questions

**Purpose:** Reference for conducting missionary interviews.

**Header:** “Interview Questions for Prospective Missionaries”

#### Content

- Source reference: “See General Handbook, 24.4.2”
- Links to or embeds Church content
- Full list of missionary interview questions

#### Implementation Options

1. **External link** — Opens Church website/Gospel Library app
1. **Embedded content** — Display questions in-app (current approach)
1. **Hybrid** — Show questions in-app with “View in Gospel Library” link

#### Content Type

**Static/External** — Content comes from official Church sources.

-----

### 6. Training Documents

**Purpose:** Searchable library of training materials for leaders.

**Header:** “Training Documents”

#### Components

1. **Search bar** — Filter documents by title
1. **”+” button** — Add new document (admin only)
1. **Document list** — Scrollable list of training resources

#### Document List Item

Each document shows:

- **Title** (tappable)
- **⋯ menu** — Actions (view, edit, delete)

#### Example Documents

- Bishopric and Ward Council Agenda Preparation Guide
- Covenant Path Progress Tool Walk Through Activity
- Extending Callings and Releases
- Giving and Following Up on Assignments: The Savior’s Way
- Huddle Training: Church Video and Powerpoint
- Huddle Training: Texas Austin Mission Video
- Ministering Interviews The Savior’s Way
- Ministering Through Setting Apart
- Sacred Funds, Sacred Responsibilities Video
- Stake Building Baptism Policy
- Stake TFH Webpage
- Strengthening New And Returning Members

#### Document Types

Documents can be:

- **External links** — YouTube videos, Google Docs, Church website
- **PDF uploads** — Stored in app
- **Rich text content** — Created in-app

#### Data Model

```
training_documents
├── id (uuid)
├── stake_id (foreign key)
├── title (string)
├── description (text, optional)
├── document_type (enum) — "link" | "pdf" | "content"
├── url (string, optional) — for external links
├── file_url (string, optional) — for uploaded PDFs
├── content (text, optional) — for rich text content
├── category (string, optional) — for grouping
├── display_order (integer)
├── created_by_user_id (foreign key)
├── created_at (timestamp)
└── updated_at (timestamp)
```

#### Permissions

- **View:** All authenticated users
- **Add/Edit/Delete:** Admin only (“Can Edit App Data?” = true)

-----

### 7. Info Change Request

**Purpose:** Allow users to report errors or request updates to app information.

**Header:** “Info Change Submissions”

#### Form Fields

|Field             |Type              |Required|
|------------------|------------------|--------|
|**Add Screenshot**|Image picker      |No      |
|**Add Screenshot**|Image picker (2nd)|No      |
|**Change Request**|Text area         |Yes     |

#### Submit Flow

1. User fills out form and taps **Submit**
1. Submission goes to admin review queue
1. Admin sees count on Directory: “Info Change Submissions to Review (X)”
1. Admin reviews and takes action (update info, delete request)

#### Data Model

```
info_change_requests
├── id (uuid)
├── stake_id (foreign key)
├── submitted_by_user_id (foreign key)
├── description (text)
├── screenshot_1_url (string, optional)
├── screenshot_2_url (string, optional)
├── status (enum) — "pending" | "resolved" | "rejected"
├── admin_notes (text, optional)
├── resolved_by_user_id (foreign key, optional)
├── created_at (timestamp)
└── resolved_at (timestamp, optional)
```

-----

## Bottom Navigation

The app uses a bottom tab bar with 4 tabs:

|Tab               |Icon|Destination                                                |
|------------------|----|-----------------------------------------------------------|
|**Stake Council** |👥   |Stake Council section (calling tracker, action items, etc.)|
|**Directory**     |📱   |Directory hub (this section)                               |
|**Share This App**|🔗   |QR code and sharing options                                |
|**Edit Users**    |✏️   |User management (admin only)                               |

### Tab Visibility

- **Stake Council:** Only visible to SC members and above
- **Directory:** Visible to all authenticated users
- **Share This App:** Visible to all authenticated users
- **Edit Users:** Only visible to admins (“Can Edit App Data?” = true)

-----

## Share This App Tab

**Purpose:** Enable users to invite others to the app.

**Header:** “Share This App”

### Components

1. **QR Code** — Large, scannable QR code with stake logo embedded in center
1. **Instructions:** “SCAN THIS QR CODE TO OPEN THE [STAKE NAME] DIRECTORY APP”
1. **Copy App Link** button — Copies shareable link to clipboard
1. **Share Link Via Email** button — Opens email composer with app link

### How Sharing Works

1. Existing user shares via QR or link
1. New person scans/taps to open app
1. New person creates account (email/Google/Apple)
1. Registration goes to admin review queue
1. Admin approves and assigns calling
1. New user gets access based on assigned role

-----

## Edit Users Tab

**Purpose:** Manage app users and their permissions.

**Header:** “Users”

### User List Screen

#### Components

- **Search bar** — Search users by name
- **”+” button** — Add new user manually
- **Filter/sort button** — Organize user list

#### User List Item

Each user card shows:

- **Calling** (blue label) — e.g., “STAKE YOUNG MEN 1ST COUNSELOR”
- **Name** — e.g., “Corey Humrich”
- **Ward** — e.g., “Bastrop Ward”
- **⋯ menu** — Actions (view, edit, delete)

### User Detail/Edit Screen

**Header:** “[User Name]”

#### User Info (Read-Only at Top)

- Name
- Current calling
- Email button (envelope icon) to contact user

#### Editable Fields

|Field                                   |Type    |Notes                                            |
|----------------------------------------|--------|-------------------------------------------------|
|**Ward or Branch**                      |Dropdown|User’s home unit                                 |
|**Calling**                             |Dropdown|Determines app permissions                       |
|**If High Council Member, What Number?**|Dropdown|HC#1-12 (only shows if calling is “High Council”)|

#### Permission Toggles

|Toggle                  |Purpose                        |
|------------------------|-------------------------------|
|**Allow Access to App?**|User can log in and use the app|
|**Can Edit App Data?**  |Admin-level editing privileges |

#### Status Indicator (Read-Only)

- **Receive Push Notifications Turned On?** — Shows device notification status

### New Sign-Ups Review

When new users register:

1. They appear in “New Sign-Ups to Review” queue
1. Admin opens their profile
1. Admin verifies identity, assigns calling
1. Admin toggles “Allow Access to App?” to ON
1. User can now access the app

### Permissions

- **View user list:** Admins only
- **Edit users:** Admins only
- **Approve new users:** Admins only

-----

## Data Models Summary

### Resources Table

```
resources
├── id (uuid, primary key)
├── stake_id (foreign key → stakes)
├── category (enum) — "general" | "training" | "welfare" | "temple" | "missionary"
├── title (string)
├── description (text, optional)
├── icon (string, optional) — icon name or URL
├── resource_type (enum) — "link" | "content" | "feature"
├── url (string, optional) — external link
├── content (text, optional) — inline content
├── feature_route (string, optional) — internal app route
├── display_order (integer)
├── is_active (boolean, default true)
├── created_at (timestamp)
└── updated_at (timestamp)
```

### Units Table

```
units
├── id (uuid, primary key)
├── stake_id (foreign key → stakes)
├── unit_number (integer) — 1-9 (display order)
├── name (string) — "Bastrop Ward", "San Marcos YSA Branch"
├── unit_type (enum) — "ward" | "branch"
├── building_photo_url (string, optional)
├── address (string)
├── latitude (decimal, optional) — for maps
├── longitude (decimal, optional) — for maps
├── sacrament_time (string) — "12:00 PM"
├── ward_council_schedule (string) — "2nd, 3rd, 4th Sunday 7:45 AM"
├── youth_activities_schedule (string) — "Wednesday at 6 PM"
├── hc_liaison_user_id (foreign key → users, optional)
├── bishop_user_id (foreign key → users, optional)
├── created_at (timestamp)
└── updated_at (timestamp)
```

-----

## Design Recommendations

### Current Issues

1. **Tile overload** — Directory has 13+ tiles; can be overwhelming
1. **No categorization** — All resources at same level
1. **Inconsistent icons** — Mix of emoji and icon styles
1. **No search** — Can’t search across all Directory features

### Recommended Improvements

1. **Group tiles into categories:**
- Quick Actions (Submit Calling, Push Notifications)
- Leadership Tools (HC Info, Meetings, Tracker)
- Reference (Ordinances, Interview Questions)
- Resources (Training, Links, Unit Info)
1. **Add global search** — Search across all Directory content
1. **Role-based tile visibility** — Hide irrelevant tiles based on calling
- Bishop sees: Unit Info, Submit Calling, Training
- RS President sees: Resources, Training
- HC sees: Everything
1. **Favorites/pinning** — Let users pin frequently-used tiles to top
1. **Badge counts** — Show counts on tiles that have pending items
- “Training Documents” → (3 new)
- “Speaking Assignments” → (1 upcoming)

-----

## Permissions Matrix (Directory Features)

|Feature                       |All Users|Unit Leaders|SC|HC|PO    |Admin     |
|------------------------------|---------|------------|--|--|------|----------|
|High Council Members          |✓ View   |✓           |✓ |✓ |✓     |✓ Edit    |
|General Resources             |✓        |✓           |✓ |✓ |✓     |✓ Edit    |
|Unit Info                     |✓        |✓           |✓ |✓ |✓     |✓ Edit    |
|Priesthood Ordinances         |✓        |✓           |✓ |✓ |✓     |✓         |
|Temple Recommend Questions    |✓        |✓           |✓ |✓ |✓     |✓         |
|Missionary Interview Questions|✓        |✓           |✓ |✓ |✓     |✓         |
|Training Documents            |✓        |✓           |✓ |✓ |✓     |✓ Add/Edit|
|Info Change Request           |✓ Submit |✓           |✓ |✓ |✓     |✓ Review  |
|MP Ordination Submission      |✗        |✓           |✗ |✓ |✓     |✓         |
|MP Ordination Tracker         |✗        |◐ Own unit  |✗ |✓ |✓     |✓         |
|Calling Tracker               |✗        |◐ If allowed|✗ |✓ |✓     |✓         |
|Speaking Assignments          |✓ View   |✓           |✓ |✓ |✓ Edit|✓ Edit    |
|Share This App                |✓        |✓           |✓ |✓ |✓     |✓         |
|Edit Users                    |✗        |✗           |✗ |✗ |✗     |✓         |

Legend: ✓ = Full access, ◐ = Conditional access, ✗ = No access

-----

## Implementation Notes

### Static vs Dynamic Content

|Content                       |Type   |Source                   |Editable                   |
|------------------------------|-------|-------------------------|---------------------------|
|Priesthood Ordinances         |Static |General Handbook         |No (except stake additions)|
|Temple Recommend Questions    |Static |General Handbook 26.3.3.1|No                         |
|Missionary Interview Questions|Static |General Handbook 24.4.2  |No                         |
|Training Documents            |Dynamic|Stake-managed            |Yes (admin)                |
|General Resources             |Dynamic|Stake-managed            |Yes (admin)                |
|Unit Info                     |Dynamic|Stake-managed            |Yes (admin)                |
|HC Member List                |Dynamic|User database            |Auto-populated             |

### External Links

Some features open external content:

- Handbook links → churchofjesuschrist.org
- Gospel Library content → Gospel Library app (if installed) or web
- Training videos → YouTube
- Maps/Directions → Apple Maps or Google Maps

### Offline Considerations

For users in areas with poor connectivity:

- Cache ordinance wording locally
- Cache interview questions locally
- Cache unit info (address, times)
- Show “offline” indicator for features requiring network

-----

*End of Directory Specification*