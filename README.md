# VR-Group
Maintenance application for VR Group 

---

#  Architecture

### **Client (Web / PWA)**

* **Frontend:** React / Next.js + Tailwind CSS
* **PWA features:** installable, offline cache, push notifications (via FCM)
* **Routes:**

  * `/login`
  * `/resident/dashboard`
  * `/resident/tickets/new`
  * `/resident/tickets/[id]`
  * `/admin/dashboard`
  * `/admin/tickets/[id]`

---

### **Authentication & Access**

* **Firebase Auth (Phone OTP only)**

  * Residents sign in via phone number → OTP
  * Admins added manually or invited by dev console
* **Custom Claims** for role-based access:

  * `role`: `'resident' | 'admin'`
  * `buildingId`, `flatId`

---

### **Database & Files**

* **Cloud Firestore** – stores residents, flats, and ticket data
* **Firebase Storage** – stores uploaded images/videos of issues

---

### **Backend / Serverless Logic**

* **Cloud Functions (TypeScript)** for automation:

  * `onTicketCreate` → notify admins via **FCM**
  * `onTicketStatusChange` → notify resident via **FCM**
  * `dailyAnalytics` → aggregate stats (optional BigQuery export)
  * `cleanupOldTickets` → archive closed tickets (optional)

---

### **Notifications**

* **Firebase Cloud Messaging (FCM)** only

  * Residents get push when their ticket status changes
  * Admins get push when a new ticket is created
* No email, no SMS — just app/browser push notifications

---

### **Hosting & Security**

* **Firebase Hosting** (with CDN & HTTPS)
* **Firestore Rules** – enforce access by user role and flat ID
* **Storage Rules** – restrict media by ticket ownership
* **Monitoring:** Firebase Console Logs + Crashlytics (for PWA errors)
* **Backups:** Scheduled Firestore export to Cloud Storage

---

#  Core Flow (No Technician, No Email)

### **Actors:** Resident & Admin

---

### 1️⃣ Resident Login

* Resident logs in using **phone number + OTP**.
* System verifies role and maps them to their **flat** and **building**.

---

### 2️⃣ Raise Service Request

* Resident selects:

  * **Service type:** electrician, plumber, carpenter, etc.
  * **Preferred date/time slot**
  * **Description** + optional **images/videos**
* Ticket created in Firestore with status = `pending`.
* Cloud Function triggers push notification → Admin.

---

### 3️⃣ Admin Acknowledges & Updates

* Admin dashboard lists all **pending tickets**.
* Admin can:

  * View ticket details
  * Mark as `acknowledged`, `in_progress`, or `completed`
  * Add **internal notes or ETA**

---

### 4️⃣ Real-Time Tracking (Resident View)

* Resident sees **status updates instantly** through Firestore listeners.
* Gets **push notifications** for any update.

---

### 5️⃣ Completion & Feedback

* Admin marks ticket as `completed`.
* Resident receives a push notification → can submit **feedback rating (1-5)** and short comment.
* Ticket archived as closed.

---

### 6️⃣ Analytics (Admin Dashboard)

* Dashboard shows:

  * Total requests per category
  * Avg. response/resolution time
  * Pending vs completed tickets
  * Feedback averages

---

#  Firestore Collections (Simplified)

```
/users/{userId}
  role, name, phone, buildingId, flatId, fcmTokens[]

/buildings/{buildingId}
  name, address

/flats/{flatId}
  buildingId, flatNo, residentUserId

/tickets/{ticketId}
  buildingId, flatId, createdBy, category,
  description, images[], preferredDate, preferredSlot,
  status, adminNotes, createdAt, updatedAt

/audit_logs/{logId}
  actorId, action, ticketId, timestamp
```

---

#  Advantages of This Setup

* **No email dependencies** — faster OTP-based sign-in.
* **100 % Firebase-native** — Auth, DB, Storage, Hosting, FCM all under one console.
* **Real-time updates** via Firestore listeners — no page refreshes.
* **Push notifications only** — mobile-first and instant.
* **Easily expandable** later (technician role, payments, analytics exports, etc.).

---
