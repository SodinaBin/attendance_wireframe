# Mobile Attendance Module - Technical Specification & Implementation Blueprint

> **Source:** Reverse-engineered and adapted from the interactive mobile wireframe prototype (`index.html`).  
> **Target Platform:** Flutter (iOS & Android).  
> **Audience:** AI Coding Agents & Mobile Developers integrating the attendance module into an existing Flutter codebase.

---

## 1. Overview & System Architecture

### 1.1 Purpose & Scope
The Mobile Attendance Module is a dual-capability mobile application module designed for **participants** (employees, students, event attendees) and **session hosts** (session owners and managers). It unifies personal attendance tracking, live QR check-in verification (with geofencing and device-locking), and full session lifecycle management (creation, roster approval, and attendance records auditing).

### 1.2 Mobile-First Navigation & Layout Architecture
The module follows native mobile UX paradigms with a **Bottom Navigation Bar** (2 destinations), a contextual **Bottom-Right Floating Action Button (FAB)** for session creation, and integrates with the host app's **Unified QR Scanner**:

```
+-------------------------------------------------------------+
|               Top App Bar / Status Header                   |
|   (Screen Title, Filter Dropdown, Unified Scan Action)      |
+-------------------------------------------------------------+
|                                                             |
|                      Active Screen Area                     |
|   (List View / Filtered Cards / Subtab Panes / Forms)       |
|                                                             |
|                                        [ + Create FAB ]     |
|                                       (My Sessions Tab)     |
+-------------------------------------------------------------+
|                 Bottom Navigation Bar                       |
|        [ 📅 My Attendance ]     [ 👥 My Sessions ]          |
+-------------------------------------------------------------+
```

#### Key Architecture Principles:
1. **Bottom Navigation (2 Destinations):**
   - **Destination 1: My Attendance:** Personal attendance check-in history, status filters (`All`, `Ongoing`, `Past`, `Late`), and detailed digital attendance slips.
   - **Destination 2: My Sessions:** Session management hub for hosts/managers, with subtabs for QR display, device roster approval, punctuality records, and settings.
2. **Unified App QR Scanner Integration:**
   - Instead of a redundant, centered scanner FAB inside the attendance tab, QR scanning leverages the **Host App's Unified QR Scanner** (accessible from the top bar or global scan action). Scanned payloads are dispatched to the attendance verification handler.
3. **Bottom-Right Creation FAB:**
   - In the `My Sessions` tab, the `+ New Session` action is placed as a native **Floating Action Button** at the bottom-right (`FloatingActionButtonLocation.endFloat`), clearing visual clutter from section headers.
   - The FAB automatically hides during multi-select roster bulk actions or when drilling down into session details.
4. **Theme & Design System Inheritance:**
   - The module does **not** introduce a foreign color palette. It strictly consumes the parent app's `Theme.of(context)` tokens (`colorScheme`, `textTheme`, `cardTheme`), seamlessly supporting both **Light Mode** and **Dark Mode**.

---

## 2. Screen-by-Screen UI/UX Specifications

### 2.1 Tab 1: My Attendance (Personal Attendance)

#### A. List View (`MyAttendanceListView`)
- **Top App Bar / Header:**
  - Title: `"My Attendance"` (bold, 16px).
  - Subtitle: `"Your personal check-in history"`.
  - Actions Row:
    - **Unified Scan Action:** Icon button or pill button (`[⛶ Scan]`) that invokes the host app's unified QR scanner.
    - **Filter Dropdown:** `Filter: All` (`all`), `Ongoing` (`ongoing`), `Past` (`past`), `Late` (`late`).
- **Section Headers:**
  - Filter `all` / `ongoing`: `"Ongoing Sessions ({count})"`.
  - Filter `all` / `past`: `"Past Sessions ({count})"`.
  - Filter `late`: `"Late Arrival Records ({count})"`.
- **Attendance Card Item (`AttendanceCard`):**
  - **Leading Avatar/Icon:** 42×42 rounded/circular avatar showing category icon (`workplace`, `class`, `events`, `others`) or custom uploaded session photo.
  - **Center Content:**
    - Session Name (bold, single-line truncated).
    - Session Time (e.g., `08:00 – 17:00`).
    - Date string (e.g., `Today, 25 Sep` or `22 Sep 2026`).
  - **Trailing Status Badge:**
    - `Checked In`: Standard active container background.
    - `Checked Out`: Primary container or solid contrast badge.
    - `Late`: Outlined warning/dashed badge with bold label.
  - **Tap Interaction:** Pushes `AttendanceRecordDetailScreen` over the bottom navigation bar.

#### B. Detail View (`AttendanceRecordDetailScreen`)
- **Navigation Bar:**
  - Back Button: `"‹ All Attendance"`.
  - Title: `"Attendance Record"`.
  - Trailing: Current status badge (`Checked In`, `Checked Out`, `Late`).
- **Cover Banner (Height: 130px):**
  - Displays session cover photo or default verified slip hallmark banner (`ATTENDANCE RECORD - Verified Participant Slip · Khemra Protocol`).
  - Trailing bottom pill: `"✓ Verified Record Photo"` or `"✓ Verified Slip Cover"`.
- **Header Profile Card:**
  - Session Name (15px, bold).
  - User Avatar with Khemra profile hallmark (`"Alex [Khemra Profile]"`).
  - Subtitle: Session category, date, and scheduled hours.
  - Large status badge.
- **Session Schedule & Times Card:**
  - `Scheduled Time`: e.g., `08:00 – 17:00`.
  - `Check-in Time`: e.g., `08:24 AM`.
  - `Check-out Time`: e.g., `17:02 PM` (or `"Not required for this session"` if `checkoutRequired == false`).
- **Late Arrival Notice Card (Shown ONLY when `status == 'Late'`):**
  - Prominent notice card.
  - Label: `"Late Arrival Notice"`.
  - Reason reported: (e.g., `"Heavy traffic on Monivong Blvd"`).
  - Context: `"Scheduled start: 08:00 AM · Grace ended 08:10 AM"`.
- **Attendees & Coworkers List:**
  - Header: e.g. `"Coworkers in this Shift (34 present)"` / `"Classmates in this Lecture (42 present)"`.
  - Peer row: 22×22 avatar, peer name (self highlighted with `★`), role tag, and check-in timestamp.
  - Footer: `"+ {count} more coworkers checked in"`.
- **Audit Disclaimer Note:**
  - `"Participant Attendance Slip · Verified via Khemra Nexus Core. This record is read-only. For amendments, contact your session manager or administrator."`

---

### 2.2 Tab 2: My Sessions (Session Management Hub)

#### A. List View (`MySessionsListView`)
- **Top App Bar / Header:**
  - Title: `"My Sessions"` (bold, 16px).
  - Subtitle: `"All sessions you own or manage"`.
  - Trailing: Filter dropdown (`Filter: All`, `Ongoing`, `Past`).
- **Section Headers:**
  - Clean headers without redundant buttons: `"Ongoing Sessions ({count})"`, `"Past Sessions ({count})"`.
- **Bottom-Right Floating Action Button (`CreateSessionFab`):**
  - Position: `FloatingActionButtonLocation.endFloat` (above the bottom navigation bar).
  - Icon: `+` (or extended FAB with label `"New Session"`).
  - Action: Launches the 3-step `SessionCreationWizard`.
- **Session Card Item (`SessionCard`):**
  - **Leading Avatar:** 42×42 session icon or uploaded image.
  - **Center Content:**
    - Session Name (bold, single-line truncated).
    - Working hours / schedule (`08:00 – 17:00`).
    - Lifetime status (`Ongoing` or `Ended {date}`).
  - **Trailing Badges:**
    - If Past: `"Ended"` badge.
    - Role Badge: `Owner` (solid primary badge) vs. `Manager` (tonal bordered badge).
  - **Tap Interaction:** Pushes `SessionDetailScreen`.

#### B. Detail View (`SessionDetailScreen`)
- **Navigation Bar:**
  - Back Button: `"‹ All Sessions"`.
  - Title: `"Session Details"`.
  - Trailing: Role indicator (`Owner` / `Manager`).
- **Cover Banner (Height: 130px):**
  - Cover banner with action pills:
    - `"📷 Change Cover"` (triggers image picker).
    - `"✕ Reset"` (resets to default graphic if custom cover is active).
- **Header Card:**
  - Session Name + Host Profile pill (`"Alex [Khemra Profile]"`).
  - Schedule & Lifetime info.
  - Role badge (`Owner` / `Manager`).
- **4 Subtab Segmented Bar:**
  - `QR Code` | `Roster` | `Records` | `Settings`.

---

### 2.3 Session Detail: Subtab Panes

#### Subtab 1: QR Code (`SessionQrSubtab`)
- **Segmented Mode Switcher:** `Static` | `Dynamic`.
- **QR Card Display:**
  - 160×160 rounded container with generated QR code matrix.
  - Title: `"Static QR Code"` vs. `"Dynamic QR Code"`.
  - Subtitle: `"Permanent scan code"` vs. `"Rotating dynamic token (valid 30s)"`.
- **Dynamic Mode Controls:**
  - `[↻ Regenerate QR Code]` button.
  - Status indicator: `"New rotating QR generated at HH:mm:ss (valid 30s)"`.
  - Security note: `"Single-use rotating token prevents screenshot sharing."`
- **Static Mode Controls:**
  - Description: `"Fixed QR code for flyers, badges, or projector display."`
  - `[⬇ Download / Print QR]` button.

#### Subtab 2: Roster (`SessionRosterSubtab`)
Behavior is dynamically determined by `session.deviceLock`:

##### Case 1: `deviceLock == true` (Enforces Registered Device Approvals)
- **Header:**
  - Heading: `"Device & Member Roster"`.
  - Actions: `[Select]` button and `[+ Add]` button.
- **Filter Chips:** `All` | `Pending` | `Approved`.
- **Selection Mode (`Select` button toggled):**
  - `Select` button toggles to `"Cancel"`.
  - Hides bottom-right FAB to prevent overlapping interactions.
  - **Select All Bar appears:** `Select All ({visibleCount})` checkbox and `"{count} selected"` label.
  - Rows show checkboxes; tapping row toggles selection.
  - **Bulk Action Bottom Bar (Fixed at bottom edge):**
    - `[✓ Approve Selected ({count})]` (Primary filled button).
    - `[✕ Reject Selected ({count})]` (Outlined secondary button).
    - Batch confirms and updates statuses.
- **Normal View Mode:**
  - Approved items show: `[Approved]` badge.
  - Rejected items show: `[Rejected]` badge.
  - Pending items show: `[Approve Device]` actionable button. Tapping immediately approves the device.

##### Case 2: `deviceLock == false` (Open Check-ins)
- **Header:** `"Attendee List"` / `"Recent check-ins"`.
- Chips and selection mode are hidden.
- Displays plain member rows with name, ID, and check-in timestamp.

#### Subtab 3: Records (`SessionRecordsSubtab`)
Behavior is dynamically determined by `session.checkoutRequired`:

##### Case 1: `checkoutRequired == true`
- Header: e.g. `"Today: 4 Present · 1 Late"`.
- Rows display: `In: {inTime} · Out: {outTime}` with audit note: `On Time` or `Grace (Late)`.

##### Case 2: `checkoutRequired == false`
- Header: e.g. `"Total Scans: 5 Attendees"`.
- Rows display: `Scanned at {inTime}` with note: `Checked In`.

- **Action:** `[⬇ Export CSV]` button in header.

#### Subtab 4: Settings (`SessionSettingsSubtab`)
- **Session Name:** Editable text field.
- **Security & Restrictions Group:**
  - `Device Lock Switch`: Toggles device enforcement.
  - `Geofence Switch`: Toggles GPS geofence.
    - If ON: reveals **Radius (meters)** text field (default: `150`).
  - `Working Hours`: Text field (e.g., `08:00 – 17:00`).
  - `Late Grace Interval`: Text field (e.g., `10 minutes`).
  - `Check-out Required Switch`: Toggles mandatory check-out.
- **Integrations:**
  - `Connect Telegram Account Switch`.
- **Save Action:** `[Save Changes]` button.
- **Role-Based Controls:**
  - **If Owner:** `"Owner Controls"` section with `[⇄ Transfer Ownership]` and `[✕ Delete Session]`.
  - **If Manager:** Owner controls hidden; renders disclaimer note: `"You are a Manager on this session. Owner-only actions (Delete Session, Transfer Ownership) are restricted."`

---

### 2.4 Flow 3: Session Creation Wizard (3-Step Flow)

```
[Step 1: Essentials] ───► [Step 2: Optional Details] ───► [Step 3: Review & Create]
```

- **Top Bar:** Back button (`‹ My Session` on Step 1, `‹ Back` on Steps 2-3), Title `"New Session"`, Step indicator (`1 / 3`, `2 / 3`, `3 / 3`).
- **Bottom Dots Indicator:** 3 dots showing active step.

#### Step 1: Essentials
1. **Session Name:** Text field (e.g., `Software Architecture Lab`).
2. **Start Date / Time:** Date/Time picker (default: `Today, 09:00 AM`).
3. **Session Lifetime:** Segmented toggle:
   - `Fixed Duration`: Reveals End Date / Time picker.
   - `Ongoing`: Hides End Date / Time picker.
4. **CTA:** `[Continue]` -> proceeds to Step 2.

#### Step 2: Optional Details
1. **Expected Participants (Capacity):** Optional numeric field.
2. **Session Photo:** Preview avatar (48×48), `[Choose Photo]`, `[Reset]`.
3. **Access & Management:**
   - `[+ Add Manager]` toggle button.
   - `[+ Upload / Add Roster (optional)]` toggle button.
4. **Telegram Integration Switch.**
5. **Security & Attendance Rules (Collapsible Accordion):**
   - Header with chevron `▶` / `▼` and count hint (e.g., `"2 enabled"`).
   - Collapsed by default.
   - Includes: Device Lock switch, Geofence switch + Location picker + Radius (meters), Working Hours, Late Grace Interval, Check-out Required switch.
6. **CTA:** `[Continue]` -> proceeds to Step 3.

#### Step 3: Review & Create
- Dynamic summary showing only active/configured fields.
- **CTA:** `[Create Session]` -> Instantiates session, prepends to session list, and opens the session detail screen.

---

### 2.5 Unified QR Scanner Integration & Verification Flow

Instead of a standalone centered scan button, check-ins are handled through the host app's unified QR scanner:

1. **Invocation Entrypoint:**
   - Tapping `[⛶ Scan]` in the `My Attendance` header, or scanning an attendance QR via the main app's global scanner bar.
2. **Verification Pipeline:**
   - **Step 1 — QR Payload Parse:** Extracts `sessionId`, token type (`static` vs. `dynamic`), and timestamp.
   - **Step 2 — Dynamic Token Validation:** If dynamic, verifies HMAC signature and confirms `currentTime - tokenTime <= 30s`.
   - **Step 3 — Geofence Verification:** If `session.geofence == true`, fetches current GPS position via `Geolocator` and verifies `distance <= session.radiusMeters`.
   - **Step 4 — Device Lock Verification:** If `session.deviceLock == true`, fetches device fingerprint via `device_info_plus`. If not approved, creates a pending approval record and alerts the user.
   - **Step 5 — Punctuality Check:** Evaluates check-in time against `session.workingHours` and `session.graceInterval`. If late, prompts for `lateReason`.
   - **Step 6 — Persistence:** Prepends record to user's attendance log and displays confirmation feedback.

---

## 3. Host App Theme Integration & Design System Tokens

> [!IMPORTANT]
> The Mobile Attendance Module must **NOT** hardcode hex colors (e.g., `#222`, `#333`, `#e9e9e9`). It must consume the existing Flutter theme via `Theme.of(context)` so it automatically adapts to light, dark, and brand themes.

### 3.1 Flutter Theme Mapping Table

| Wireframe Element | Recommended Flutter Theme Token | Fallback / Behavior |
| :--- | :--- | :--- |
| **Scaffold Background** | `colorScheme.surface` or `scaffoldBackgroundColor` | Clean contrast with cards |
| **Card / Panel Background** | `colorScheme.surfaceContainer` / `cardTheme.color` | Subtle elevation / border |
| **Card Borders / Dividers** | `colorScheme.outlineVariant` / `dividerColor` | 1.0–1.5px subtle line |
| **Primary Text (Headings)** | `textTheme.titleMedium` / `colorScheme.onSurface` | Bold, high emphasis |
| **Muted Text / Subtitles** | `textTheme.bodySmall` / `colorScheme.onSurfaceVariant` | 60% opacity / muted |
| **Primary Buttons & FAB** | `colorScheme.primary` (bg), `colorScheme.onPrimary` (fg) | Host brand primary color |
| **Secondary / Outlined Buttons** | `colorScheme.outline` (border), `colorScheme.primary` (text) | Outlined button style |
| **Bottom Navigation Bar** | `navigationBarTheme` / `colorScheme.surfaceContainer` | Material 3 navigation bar |
| **Active Bottom Nav Item** | `colorScheme.primary` | Highlighted icon & bold label |
| **Inactive Bottom Nav Item** | `colorScheme.onSurfaceVariant` | Muted icon & label |
| **Badge: Owner / Checked Out** | `colorScheme.primaryContainer` / `onPrimaryContainer` | Filled tonal badge |
| **Badge: Manager / Checked In**| `colorScheme.surfaceContainerHighest` / `outline` | Bordered neutral badge |
| **Badge: Late / Warning** | `colorScheme.errorContainer` / `onErrorContainer` | Warning / error accent |
| **Badge: Approved Device** | `colorScheme.tertiaryContainer` / `onTertiaryContainer` | Success green / teal tint |
| **Segmented Control Pill** | `colorScheme.surfaceContainerHighest` + `primary` indicator | Pill-shaped toggle |

---

## 4. Data Models & Entity Schemas (Dart)

```dart
// ---------------------------------------------------------------------------
// Enums
// ---------------------------------------------------------------------------

enum UserRole { owner, manager }

enum LifetimeType { ongoing, fixedDuration }

enum SessionStatus { ongoing, past }

enum AttendanceType { workplace, classSession, events, others }

enum AttendanceStatus { checkedIn, checkedOut, late }

enum DeviceApprovalStatus { approved, pending, rejected }

enum QrMode { staticQr, dynamicQr }

// ---------------------------------------------------------------------------
// Entities
// ---------------------------------------------------------------------------

class AttendanceSession {
  final String id;
  String name;
  String? image;
  UserRole role;
  LifetimeType lifetime;
  SessionStatus status;
  String? startDate;
  String? endDate;
  String? endedOn;
  int? participants;
  
  // Security & Restrictions
  bool deviceLock;
  bool geofence;
  double radiusMeters;
  String? workingHours; // e.g. "08:00 – 17:00"
  String? graceInterval; // e.g. "10 minutes"
  bool checkoutRequired;
  bool telegramConnected;
  QrMode qrMode;

  AttendanceSession({
    required this.id,
    required this.name,
    this.image,
    required this.role,
    this.lifetime = LifetimeType.ongoing,
    this.status = SessionStatus.ongoing,
    this.startDate,
    this.endDate,
    this.endedOn,
    this.participants,
    this.deviceLock = false,
    this.geofence = false,
    this.radiusMeters = 150.0,
    this.workingHours,
    this.graceInterval,
    this.checkoutRequired = false,
    this.telegramConnected = false,
    this.qrMode = QrMode.staticQr,
  });
}

class UserAttendanceRecord {
  final String id;
  final String sessionId;
  final String sessionName;
  final AttendanceType type;
  final String? image;
  final String sessionTime;
  final String date;
  final bool isOngoing;
  AttendanceStatus status;
  String? checkInTime;
  String? checkOutTime;
  String? lateReason;
  String? requiredTime;
  String? deviceName;
  String? geofenceStatus;
  bool checkoutRequired;

  UserAttendanceRecord({
    required this.id,
    required this.sessionId,
    required this.sessionName,
    required this.type,
    this.image,
    required this.sessionTime,
    required this.date,
    required this.isOngoing,
    required this.status,
    this.checkInTime,
    this.checkOutTime,
    this.lateReason,
    this.requiredTime,
    this.deviceName,
    this.geofenceStatus,
    this.checkoutRequired = true,
  });
}

class PeerAttendee {
  final String name;
  final String time;
  final String? role;
  final bool isSelf;

  PeerAttendee({
    required this.name,
    required this.time,
    this.role,
    this.isSelf = false,
  });
}

class RosterMember {
  final String id;
  final String name;
  final String code; // e.g., "EMP-1021"
  final String? deviceModel;
  DeviceApprovalStatus status;
  final String? checkInTime;

  RosterMember({
    required this.id,
    required this.name,
    required this.code,
    this.deviceModel,
    this.status = DeviceApprovalStatus.approved,
    this.checkInTime,
  });
}

class AttendanceAuditLog {
  final String id;
  final String memberName;
  final String inTime;
  final String? outTime;
  final String note; // e.g., "On Time", "Grace (Late)", "Checked In"

  AttendanceAuditLog({
    required this.id,
    required this.memberName,
    required this.inTime,
    this.outTime,
    required this.note,
  });
}
```

---

## 5. Flutter Scaffold & Widget Implementation Pattern

### 5.1 Main Module Container Skeleton

```dart
class AttendanceModuleRoot extends ConsumerStatefulWidget {
  const AttendanceModuleRoot({super.key});

  @override
  ConsumerState<AttendanceModuleRoot> createState() => _AttendanceModuleRootState();
}

class _AttendanceModuleRootState extends ConsumerState<AttendanceModuleRoot> {
  int _currentIndex = 0;

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);
    final isMySessionsTab = _currentIndex == 1;

    return Scaffold(
      body: IndexedStack(
        index: _currentIndex,
        children: const [
          MyAttendanceListView(),
          MySessionsListView(),
        ],
      ),
      // Floating Action Button placed to the bottom right for session creation
      floatingActionButton: isMySessionsTab
          ? FloatingActionButton(
              onPressed: () => Navigator.of(context).push(
                MaterialPageRoute(builder: (_) => const SessionCreationWizard()),
              ),
              backgroundColor: theme.colorScheme.primary,
              foregroundColor: theme.colorScheme.onPrimary,
              child: const Icon(Icons.add),
            )
          : null,
      floatingActionButtonLocation: FloatingActionButtonLocation.endFloat,
      // Native Bottom Navigation Bar
      bottomNavigationBar: NavigationBar(
        selectedIndex: _currentIndex,
        onDestinationSelected: (index) => setState(() => _currentIndex = index),
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.calendar_today_outlined),
            selectedIcon: Icon(Icons.calendar_today),
            label: 'My Attendance',
          ),
          NavigationDestination(
            icon: Icon(Icons.groups_outlined),
            selectedIcon: Icon(Icons.groups),
            label: 'My Sessions',
          ),
        ],
      ),
    );
  }
}
```

---

## 6. Development Roadmap & Task Checklist

Use this checklist inside the Flutter repository:

### Phase 1: Models, Theming & Dependency Integration
- [ ] Implement Dart entity models (`AttendanceSession`, `UserAttendanceRecord`, `RosterMember`, etc.).
- [ ] Connect with existing host app `ThemeData` tokens (no hardcoded colors).
- [ ] Setup repository contracts and mock providers for development.

### Phase 2: Shell, Navigation & Creation FAB
- [ ] Build `AttendanceModuleRoot` with `NavigationBar` (2 destinations: `My Attendance` & `My Sessions`).
- [ ] Implement the bottom-right `FloatingActionButton` on the `My Sessions` tab.
- [ ] Build reusable UI widgets (`StatusBadge`, `SegmentedPillControl`, `CoverBannerWidget`).

### Phase 3: Flow 1 — My Attendance
- [ ] Build `MyAttendanceListView` with category filter dropdown and header `[⛶ Scan]` trigger.
- [ ] Build `AttendanceCard` with status badges (`Checked In`, `Checked Out`, `Late`).
- [ ] Build `AttendanceRecordDetailScreen` with cover banner, profile card, timestamps, conditional late notice, and peer roster.

### Phase 4: Flow 2 — My Sessions Management
- [ ] Build `MySessionsListView` with filter dropdown (`All`, `Ongoing`, `Past`) and session cards.
- [ ] Build `SessionDetailScreen` with cover changer and 4 subtabs:
  - [ ] **QR Subtab:** Static vs. Dynamic QR switcher with 30s rotating countdown.
  - [ ] **Roster Subtab:** Dynamic view based on `deviceLock`, filter chips (`All`, `Pending`, `Approved`), and multi-select mode with bulk action bar (`Approve Selected` / `Reject Selected`).
  - [ ] **Records Subtab:** In/Out audit logs with punctuality notes and CSV export trigger.
  - [ ] **Settings Subtab:** Security rules switches, Telegram switch, and Owner controls vs. Manager disclaimer.

### Phase 5: Flow 3 — Session Creation Wizard
- [ ] Build 3-step wizard with step dots indicator.
- [ ] **Step 1:** Session Essentials form (Name, Start Date, Lifetime toggle, End Date).
- [ ] **Step 2:** Capacity, Photo picker, Manager assignment, Roster attachment, Telegram toggle, and collapsible Security Rules accordion.
- [ ] **Step 3:** Dynamic review summary showing only configured rules, followed by `Create Session` execution.

### Phase 6: Unified Scanner Integration
- [ ] Hook into the host app's unified QR scanner callback (`onQrCodeScanned(String payload)`).
- [ ] Implement the verification pipeline: Dynamic HMAC token check -> GPS geofence radius check -> Device lock check -> Punctuality calculation.
- [ ] Add check-in success snackbar/dialog and record persistence.
