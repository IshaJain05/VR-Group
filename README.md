# VR-Group
Maintenance application for VR Group 

Got it—here’s a **clean, payment-free, technician-free** version you can drop into the proposal.

# Architecture (Firebase-first, no payments)

**Client (Web PWA)**

* React/Next.js + Tailwind
* Routes: `/login`, `/resident/dashboard`, `/resident/tickets/new`, `/resident/tickets/[id]`, `/admin/*`
* Push notifications via FCM
* Form validation with Zod/React Hook Form

**Identity & Access**

* Firebase Auth (Phone OTP + Email/Password)
* Custom Claims: `role` (`resident` | `admin`), `buildingId`, `flatId`

**Data & Files**

* Firestore (multi-tenant by `buildingId`)
* Firebase Storage for ticket images/videos & generated docs

**Backend**

* Cloud Functions (TypeScript) for:

  * Ticket lifecycle hooks (onCreate/onUpdate)
  * Admin actions (assign/unassign removed; still can “acknowledge”, “schedule”, “resolve”)
  * Notifications (Email/FCM/optional WhatsApp)
  * Audit logging & analytics export (optional BigQuery)

**Notifications**

* FCM: status changes to residents, new ticket alerts to admins
* Email: ticket created/acknowledged/resolved (SendGrid via CF)
* (Optional) WhatsApp: admin alerts

**Ops & Security**

* Firebase Hosting (CDN + preview channels)
* Firestore/Storage Rules (RBAC via claims)
* Cloud Logging & Error Reporting
* Daily backups (Firestore export to GCS)

---

# Core Flow (no technician, no payments)

**Actors:** Resident, Admin

1. **Onboard & Verify**

* Admin creates/invites resident → Auth with phone/email → assign `buildingId` + `flatId` via custom claims.

2. **Raise Ticket (Resident)**

* Resident selects **category**, **priority**, **preferred date/time window**, adds **description** + **images** → ticket `status='pending'`.

3. **Auto-Notify (System)**

* Cloud Function triggers: send **admin alert** (FCM/email) with quick links.

4. **Acknowledge & Schedule (Admin)**

* Admin reviews details, can:

  * **Acknowledge** (adds ETA, internal notes)
  * **Schedule** date/time window (no technician assignment shown to user)
  * Move status → `in_progress` when work starts.

5. **Resident Updates**

* Resident sees real-time status, ETA, and any admin notes.
* Resident can **add comments/images** if needed; edits are tracked in `activity`.

6. **Resolve & Confirm**

* Admin sets status → `completed`, adds resolution notes & optional completion photo.
* Resident receives completion notification and submits **rating/feedback** (optional).

7. **Analytics & Audit**

* Daily job aggregates metrics: created vs completed, avg response/resolve time, rating trends, category heatmap.
* All actions appended to `audit_logs`.

---

## Suggested Firestore Collections (lean)

```
/buildings/{buildingId}
  name, address

/flats/{flatId}
  buildingId, tower, flatNo, residentUserId, status

/users/{userId}
  role, buildingId, flatId, name, phone, fcmTokens[]

/tickets/{ticketId}
  buildingId, flatId, createdBy, category, priority,
  description, images[], preferredDate, preferredSlot,
  status: 'pending'|'acknowledged'|'scheduled'|'in_progress'|'completed'|'cancelled',
  eta, adminNotes, createdAt, updatedAt, sla:{firstResponseAt,resolvedAt}

/ticket_comments/{ticketId}/comments/{commentId}
  authorId, message, images[], createdAt

/audit_logs/{logId}
  actorId, action, resourceType, resourceId, before?, after?, ts
```

---

## Minimal Rules Sketch

* Residents: CRUD only on **their** tickets & profile; read building- and flat-scoped data.
* Admins: Read all in `buildingId`; update any ticket; write `audit_logs`.
* Storage: ticket-media path scoped to `{ticketId}` with same RBAC.

---

If you want, I can turn this into a one-page PDF diagram (boxes & arrows) matching your branding.
