# ON THE WAY LOGISTICS - Website + Admin Panel + Vendor & Franchise Portals

```
OTW-Logistics/
  START-WINDOWS.bat     Double-click to start everything on Windows (own computer)
  start-mac-linux.sh    Same, for Mac / Linux
  DEPLOY.md             How to put it online: Vercel (frontend) + Render (backend)
  backend/              Node/Express + SQLite server (applications, emails, payments, portals' API)  -> Render
  frontend/             -> Vercel
    (root)              Public site with Franchise & Vendor application forms
    Admin/          Admin panel (applications under Client > Franchise / Vendor)
    vendor/             Vendor Management Portal
    franchise/          Franchise Management Portal
  render.yaml           Render blueprint
  deploy/vps/           Optional all-in-one server deployment (Docker)
  docs/                 Change notes and guides
```

## Start it (easy way)
1. Install **Node.js LTS** from https://nodejs.org (one time).
2. Windows: double-click **START-WINDOWS.bat**. Mac/Linux: run `./start-mac-linux.sh`.
3. Keep that window open. It is the server; if it is closed, applications cannot be received.
   - Website:      http://localhost:8080/
   - Admin panel:  http://localhost:8080/dashboard/  (also linked in the website footer: "Staff / Admin Login")
   - Vendor portal: http://localhost:8080/vendor/  (website footer: "Vendor Login")
   - Franchise portal: http://localhost:8080/franchise/  (website footer: "Franchise Login")
4. Only one copy can run at a time. If another copy (for example an older version from another folder) is still running,
   START-WINDOWS.bat says so and offers to stop it (**Y**); the browser opens only when this copy has started.
   The version is shown in the server window and at the bottom of the admin panel menu (e.g. "Version 12").

Manual way: `cd backend`, `cp .env.example .env`, `npm install`, `npm start`.

## Franchise Territory and Vendor Registration Forms
The Franchise and Vendor forms on the home page and the Services page ask the same questions. The Franchise form asks for: Full Name, Date of Birth, Gmail / Email ID, Contact Number,
WhatsApp Number, State / Union Territory, District, Village Name / City Name, Location / Area Name, PIN Code, Full Address, and
Years of Experience (a whole number from 1 to 10). The Field of Experience text area has been removed. The selected years are saved to the same database and shown in the admin panel under
Client > Franchise, and the Vendor form under Client > Vendor (View form, search, Edit, Excel export and Accept / Reject include the new fields).
- States and districts come from `website/india-districts.js` (28 States + 8 Union Territories). Check it against the official list
  if a new district is created.
- Village / city and area suggestions and the PIN code come from the public India Post directory (`api.postalpincode.in`) in the
  browser. If that service cannot be reached, the visitor types the names by hand. A PIN the visitor types is never overwritten.

## How Franchise / Vendor applications reach the admin panel
1. A visitor fills the **Franchise** or **Vendor** form on the website (home page *or* Services page,
   both use the same `website/partner-forms.js`).
2. The form is sent to the server (`POST /api/partner-applications`). The server gives it an ID
   (FP-000001, FP-000002 … for Franchise; VN-000001, VN-000002 … for Vendor; every ID is used only once) and saves every field in `backend/payments.sqlite`.
3. The visitor sees a confirmation page with their ID and details.
4. In the admin panel the application appears automatically:
   - **Client > Franchise** and **Client > Vendor**: all applications by type, with status (New, Contacted, Under Review, Documents Pending, Accepted, Rejected).
   - **View form** shows everything the applicant typed. **Edit** changes details or status; changes are saved on the server, so all admins see them.
   - The panel checks for new applications every 30 seconds, shows a pop-up when one arrives, and an orange counter in the sidebar of applications still waiting for Accept / Reject.

## Emails to applicants
When email sending is set up in `backend/.env` (Gmail App Password or SMTP), applicants get an email:
1. **Application submitted**: right after a website application, with the application ID and the details they entered.
   The confirmation page tells them it is on its way.
2. **Application approved**: when an admin clicks Approve (or sets Accepted in Edit), with the portal address, their
   Vendor / Franchise ID and temporary password. Re-approving after an undo says their existing login works again.
3. **Application rejected**: when an admin clicks Reject (or sets Rejected in Edit), with the reason if one was typed.
- The Approve / Reject window has **Email the applicant** (ticked). The note you type is included in the email; untick it
  to decide without emailing. Undo never emails.
- **Reset password** for a partner emails them the new temporary password.
- **View form** shows the email history of each application (sent, or not sent and why).
- Emails are sent from `GMAIL_USER`; replies go to `SUPPORT_EMAIL`. Set `PUBLIC_URL` so the portal link in the approval
  email works on the applicant's phone (otherwise it points to `localhost`).
- If email is not set up, nothing is sent and the history says so; share login details with Copy / WhatsApp as before.
  Applications entered in the admin panel get no "submitted" email.

## Ban a Vendor / Franchise login (Admin > Client)
- Every vendor / franchise row with a portal login has **🚫 Ban** (also in **View form**). Choose how long: 1, 3, 7 or 30
  days, until a date, or **until I unban them**, and optionally a reason.
- They are signed out at once. When they try to sign in they see "Your … Portal access has been suspended … until …", with
  your reason. Someone without the right password only sees "Incorrect ID or password".
- The application stays Approved; riders, orders and payments are not changed. A timed ban ends by itself; **✓ Unban** ends
  it at once and they sign in with their own password.
- The row shows **🚫 Banned**; View form shows the current ban and the ban history. Every ban and unban is in the audit log.

## Riders under each Vendor / Franchise (Admin > Client)
- Type a vendor or franchise name, ID (VN-… / FP-…), phone, city or district in the **Client** search box. Below the lists,
  **Riders of matching Vendors / Franchises** shows every rider each matching partner registered (approved, pending and
  rejected), with mobile, hub, status, registration date and, from the imported payment Excel, payment weeks, orders and amount.
- Each vendor / franchise row also has **🛵 Riders (N)** to see the same list without searching, and **⬇ Export riders**
  saves that partner's riders to Excel. **View** opens the rider's registration (documents, approve / reject).
- A rider belongs to the partner whose portal login registered it (Rider Applications in the Vendor / Franchise Portal).

## Accept / Reject
- Every application that is still pending has **✓ Accept** and **✕ Reject** buttons in Client > Franchise / Vendor and inside **View form**.
- Clicking one asks you to confirm and lets you add a note (for Reject: the reason). The decision, the note, which admin decided and when are saved on the server and shown to every admin
  (in View form, and when you hover over the status badge).
- Made a mistake? Open **View form** and click **↺ Undo decision**; the application goes back to *Under Review*.
- Use the **All statuses / Pending / Accepted / Rejected** filter next to the search box to work through applications.
- Older records with the status "Approved" are shown as "Accepted" automatically.

If the server is not running, the website shows an error and keeps the applicant's typing; nothing is silently lost.

## Admin sign-in
- Admin accounts and passwords are checked by the server; no passwords are stored in the website code.
- **First start:** the server creates the four admin accounts with **random temporary passwords**. They are shown in the
  server window and saved to `backend/FIRST-ADMIN-PASSWORDS.txt`. Give each admin only their own line; at first sign-in
  they must choose their own password. Then **delete that file**. (The old passwords from earlier versions no longer work.)
- Change your password any time under **Settings**. "Remember me" keeps you signed in for 30 days; otherwise until the browser closes (max 12 hours).
- **Forgot your password?** On the sign-in page choose **Forgot password?**, enter your registered admin email and open the reset link sent to that inbox. The link works once and expires after 30 minutes. Only the registered address receives it, and the page gives the same answer whether or not an address belongs to an admin. Passwords from the first version of the site were removed from the web page and no longer work; use Forgot password.
- **Email not set up or not working?** The server window says so at start-up ("Email: ready" or "Email NOT working" with the
  reason, e.g. the Gmail App Password was refused). Until it works, **Forgot password** asks the Super Admin instead: the
  request appears in **Settings → Admin Accounts** (orange counter on Settings, highlighted row), and the Super Admin clicks
  **Temporary password** to give a one-time password (shown once, copy button). The admin must choose their own password at
  the next sign-in; their other devices are signed out. The answer on the sign-in page is the same for every email address.
- Reset emails need email sending set up in `backend/.env` (`GMAIL_USER` and `GMAIL_APP_PASSWORD`, or the SMTP settings). That is the App Password of the sending mailbox, not an admin's own Gmail password. Nobody's password is stored in the site or shown on the dashboard.
- **Phones:** set `PUBLIC_URL` in `backend/.env` to the address admins use on their phones (for example `http://192.168.1.20:8080` on your Wi-Fi, or your domain). Without it the reset link in the email points at `localhost`, which a phone cannot open.
- **See every admin account** (name, registered email, status, whether the password was chosen): run `node admin-accounts.js` in the backend folder. It never shows passwords or password hashes, and it flags any admin email that is not a Gmail address.
- **Change admin names and emails / add admins:** the Super Admin opens **Settings → Admin Accounts**. **Edit** changes an
  admin's full name and registered email (the email is also their sign-in ID; their password stays the same and reset links
  sent to the old address stop working). **+ Add admin** creates a new admin and shows a temporary password once; they choose
  their own password at first sign-in. Emails must be unique. Both actions are in the audit log. For the first-start
  placeholders *Admin 3* / *Admin 4*: click **Edit**, enter the real name and email, then **Temporary password** and give it to
  that person (or they can use the password from `FIRST-ADMIN-PASSWORDS.txt` with the new email if it was never changed).
- **Super Admin:** the earliest-created admin, plus the admin named in `SUPER_ADMIN_EMAIL` in `backend/.env` if set (that adds a Super Admin; it does not remove the earlier one), sees **Settings → Admin Accounts**. The last active Super Admin cannot be switched off. There they can see every admin (never a password), **Send reset link** to any admin's registered email, and switch an admin off or on. The checks are made on the server, so other admins cannot use those actions even by calling them directly. Every admin keeps exactly the access they had before.
- **Locked out? (Windows)** Double-click **`RESET-ADMIN-PASSWORD.bat`** in the project folder. It lists the admins in this
  folder's database (who is still on a temporary password), asks for the email to reset and shows a new temporary password
  once; sign in with it and choose your own. It works while the server is running.
- **Why an admin may be locked out:**
  - The temporary password in `FIRST-ADMIN-PASSWORDS.txt` only works until that admin chooses their own password.
  - **Another copy of the server is still running** (for example from an older extracted folder): the browser then talks to
    that copy and its database. The server window now says **NOT STARTED: port 8080 is already in use** in that case, and on
    a normal start shows the **Database:** path it uses. Close the other window (or restart the computer) and start again.
  - After 10 wrong passwords for one email the sign-in page says "Too many sign-in attempts" for 15 minutes; restarting
    the server clears it.
  - Otherwise set up reset emails (below) and use **Forgot password?**, or run `node reset-admin-password.js admin@example.com` on the server.
- **Audit log:** sign-ins, failed sign-ins (the email typed, never the password), sign-outs, reset links requested / sent / failed, completed resets, password changes and switching admins on / off are recorded in the audit log.
- If email is not set up, or an admin has no access to the registered mailbox, the owner can run `node reset-admin-password.js admin@example.com` in the backend folder. It prints a temporary password, which the admin must change at the next sign-in. Add a new admin with `node reset-admin-password.js new.admin@example.com "Full Name"`.

## Vendor & Franchise Portals: partner logins
1. A partner applies on the website and gets the next ID: **VN-000001, VN-000002 …** (vendor) or **FP-000001, FP-000002 …** (franchise).
   An ID is never given out twice, even if an application is changed from vendor to franchise.
2. An admin clicks **✓ Accept**. The server creates their portal login and shows the admin the **Login ID** and a
   **temporary password** once, with **Copy details** and **Send on WhatsApp** buttons. If email is set up in `backend/.env`,
   the details are also emailed to the applicant automatically.
3. The partner opens **Vendor Login** / **Franchise Login** (website footer, or `/vendor/` and `/franchise/`), signs in with their
   ID (or email) and the temporary password, and must choose their own password.
4. Their profile starts filled in from their application (they add their own locations, each with a map link). Profile, locations, riders, orders and profile photo are saved
   **on the server under their own account**: they see the same data on any device, and no one else can see it.
5. **✕ Reject** or **↺ Undo decision** switches the login off immediately (they are signed out). Accepting again switches it back on
   with their own password and data.
6. Lost password? Open the application's **View form** and click **🔑 Reset password** (or **Create login** for applications
   accepted before this version). View form also shows whether they have signed in yet. A 🔑 next to "Accepted" means the login is active.
- **Riders**: Rider ID / Partner ID, Rider Name, Mobile Number (10 digits), Hub Pincode, Hub Location, plus two required uploads:
  **Screenshot** (JPG/PNG/WEBP) and **Aadhaar** (JPG/PNG/WEBP or PDF), up to 5 MB each. Files are checked to really be images/PDFs,
  stored in `backend/uploads/` (never in a public folder, never in GitHub) and can only be opened by the partner who uploaded them.
  Replacing a document or removing a rider deletes the old file. Ask partners to upload a **masked Aadhaar** (only the last 4 digits visible).
  **Auto-fill:** choosing the delivery-app **Screenshot** (the menu showing the name and "Partner ID: 12345678") fills in the
  Rider ID / Partner ID and Rider Name. The text is read on this server (Tesseract), not by an outside service. Filled-in fields are
  highlighted; please check them, as a character can occasionally be misread. A Rider ID typed by hand is never overwritten.
  **Mobile Number** accepts exactly 10 digits (starting 6-9); pasting "+91 98765 43210" keeps 9876543210.
- **Payment data → partners:** when an admin imports the payment Excel (admin panel > Payments > Import Excel), every row is matched to
  the partner whose **Riders** list has that rider: by **Rider ID / Partner ID** (the "Rider ID" column), otherwise by rider name when only
  one partner has a rider with that name. Matched rows fill that partner's **Orders** page (and Reports / dashboard totals), and the partner
  gets a **Notification** (orange counter in the sidebar + a pop-up within 30 seconds). Partners only ever see their own riders.
  The admin sees which partners were notified and which riders matched nobody (add those riders in the partner's Riders list, or add a
  Rider ID column to the Excel, then import again; re-importing updates the same records, it never duplicates them).
- **Orders** and **Riders** have a **↻ Refresh** button that loads the latest data from the server (it also updates by itself every minute
  and when you come back to the tab). Changes you just made are saved before refreshing.
- Partners can change their password under **Settings > Account security**. Sign-in is limited to 10 failed attempts per 15 minutes.
- **Helpline / Support** (Settings page) emails the customer's details and photo to `SUPPORT_EMAIL`; the subject says which portal and account it came from.
  To turn email on (support form and login emails), edit `backend/.env`: turn on 2-Step Verification for the Gmail account, create an
  **App Password** (Google Account > Security > App passwords), put it in `GMAIL_APP_PASSWORD`, then restart the server.
  Limits: photo up to 8 MB, 5 support requests per 10 minutes per visitor.
- To test: submit a Vendor or Franchise application on the website, accept it in the admin panel, then sign in with the details shown.

## Putting it online
The website and admin panel must be served by this backend (or point to it). If you host the website
somewhere else, add this line before `partner-forms.js` in `index.html` and `services.html`:
`<script>window.OTW_API_BASE='https://your-backend-address';</script>`, do the same in `dashboard/index.html`
before `app.js`, and set `ALLOWED_ORIGIN` in `backend/.env` to the website address. Use HTTPS.

## Before going live (important)
- Have each of the four admins sign in once and choose their own password, then delete `backend/FIRST-ADMIN-PASSWORDS.txt`.
- Set `PUBLIC_URL=https://your-domain` in `backend/.env`. For safety the server only answers on localhost, IP addresses and the
  addresses you list there (or in `ALLOWED_HOSTS`); other web addresses get "This server is not set up for this web address".
- Use **HTTPS**. If the server runs behind an HTTPS proxy/host, set `TRUST_PROXY=true` in `backend/.env` so sign-in cookies are marked Secure,
  (`PUBLIC_URL` also makes the login emails link to the right address).
- Back up `backend/payments.sqlite` **and the `backend/uploads/` folder** regularly; they hold all applications, accounts, portal data and rider documents.
- Keep `backend/.env` and `payments.sqlite` out of GitHub (the included `.gitignore` already does this).

## Code backup on GitHub
- Make the repository **Private**. The code includes rider names and phone numbers (`backend/data/rider-master-directory.json`,
  `backend/data/rider-payment-database.xlsx`), hub manager contacts (`backend/data/hubs-seed.json`) and the admins' email addresses.
- `.gitignore` keeps these out of GitHub: `backend/.env`, the database (`payments.sqlite*`), `backend/uploads/` (Aadhaar, selfies),
  `FIRST-ADMIN-PASSWORDS.txt`, `ADMIN-TEMPORARY-LOGIN-CREDENTIALS.txt`, `node_modules/`, logs and ZIP files.
- GitHub holds the **code only**. Back up `backend/.env`, `backend/payments.sqlite` and `backend/uploads/` separately
  (stop the server first, then copy them to a USB drive or Google Drive).
- After each change: GitHub Desktop > write a summary > **Commit to main** > **Push origin**.

## Rider Applications (Admin Portal only; Vendor for own riders)
- **Admin Portal** is the only place with the full Rider Applications management: view every application, inspect rider details,
  view selfies and Aadhaar documents, approve or reject (with a reason for rejection), and see the history.
  Statuses are **PENDING**, **APPROVED** and **REJECTED**.
  - **Add New Member** (Admin Riders page) registers a rider: Rider ID, Rider Name, Mobile Number, Hub Pincode, Hub Location, a
    **selfie / screenshot** and an **Aadhaar card** (both required; checked by real content, 5 MB at most each).
  - **Client > Client Members** lists all applications with summary cards, filters (Rider ID, Rider Name, Hub Location, Status),
    View / Approve / Reject and document viewers. Applied By shows Admin, Vendor or Franchise for records created before this change.
- **Vendor Portal > Rider Applications** lists only the riders that vendor registered, with Approve / Reject for those riders only.
- **Franchise Portal has no Rider Applications.** Its menu, page, buttons and approval controls were removed. The rider API refuses
  franchise requests with 403, and a franchise sign-in cannot open the admin panel or its rider data.
- Existing rider application records, documents and approval history are kept. Records created by franchises stay visible to the
  admin, with their original source. Nothing was deleted.
- Rider ID is unique among live applications (PENDING or APPROVED); a rejected rider can be registered again. The database enforces this.
- Documents are stored in `backend/uploads/registrations/` (owner-only, never public) and are opened only by a signed-in account that
  may see that record. Aadhaar views are written to the audit log. **Back up `backend/uploads/` with the database.**
- Tests: `cd backend && npm test`.

## Reports dashboard (15-day and 365-day)
Admin Portal > **Reports**. The page keeps its four existing counts and Download Excel card; the new dashboard sits under them.
- **15-Day Report**: periods are 1–15 and 16–end of each month, clipped to the selected range. Each period shows start and end dates, riders, total / Prime / Club / Large orders, percentages, a bar chart and a rider table.
- **365-Day Report**: a yearly view with one bar group per month (Jan–Dec), monthly statistics and percentages. Start and end dates override the year.
- **Filters**: start and end date, month, year, rider name or ID, order ID and hub. They apply to the five summary cards, charts, tables and both exports together.
- **Exports**: Export to Excel (`.xlsx`: Yearly Summary, 15-Day or Monthly Summary, Rider Details) and Download PDF (logo, period, filters, summary, category breakdown, chart and tables). Both use the current filter results.
- **Data source**: `GET /api/admin/reports` (admin sign-in only) reads the order records of every active vendor and franchise partner in the database, the same records the payment import writes. Each rider-day counts as one order record.
- **Definitions**: Total Orders = sum of the orders on each record (never less than Prime + Club + Large). Total Riders = distinct rider IDs with orders in the range. Percentage = category ÷ Total Orders × 100, or 0% when there are no orders. Duplicate rider-day records are counted once.
- **Order ID**: the database has no per-order ID. The displayed Order ID (`ORD-<Rider ID>-<YYYYMMDD>`) is derived from the rider and date, which is the existing unique key.
- **Known limits**: (1) Orders entered only in a browser's Daily Work form are not in the database and are not included. (2) The Large Orders field is stored on each record but nothing currently fills it, so the Large figures show 0 until a source is defined. (3) Prime and Club are taken from the imported columns, so they may not add up to Total Orders; the remainder is shown as unclassified in the totals.
- Tests: `cd backend && npm test` (includes `reports.test.mjs`).

## Changes in this version
- Version 12: Vendor and Franchise Portal **Settings > Helpline / Support** show and send to **supportontheway@gmail.com**
  (`SUPPORT_EMAIL` in `backend/.env`; the portals read the address from the server, so what they show is where messages go).
- Version 11: riders added by Vendors / Franchises (Rider Applications > Add New Member) now show on the admin **Riders** page
  in **Waiting for approval** (View, selfie, Aadhaar, Added by, Approve / Reject), with a count on the Riders menu, a pop-up
  when a new one arrives, and "N new waiting for approval" on the Dashboard. Approved riders move into the Riders list with
  an **Added by** line. (They were already listed under Client > Client Members.)
- Version 10: START-WINDOWS.bat finds an older copy of the server that is still running and offers to stop it, instead of
  opening the browser on the old copy. The version number is shown in the server window and the admin panel menu.
- Emails are sent from **ontheway5579@gmail.com** (`GMAIL_USER` in `backend/.env`); add that account's Google App Password as
  `GMAIL_APP_PASSWORD` to switch sending on. The home page partner strip now includes **Zepto** (`website/assets/partners/zepto.png`).
- Admin sign-in help (15 Nov 2026): new **RESET-ADMIN-PASSWORD.bat**; the server no longer says "running" when port 8080 is
  already taken by another copy (it says NOT STARTED and why), and it shows the database path at start-up.
- QA pass (15 Nov 2026): see `QA-TEST-REPORT.md` for the full list. In short: pages no longer go blank from the request
  limit; the weekly Excel import summary works; partners see correct order totals and get a notification after each import;
  the Admin Dashboard shows real figures; approved riders appear in Admin > Riders; Admin > Documents stores the files on the
  server; Hub Import Excel and Google Drive sync are back in the Admin Portal; Rider Applications fill Rider ID and name from
  the screenshot; the admin panel works without internet.
- Franchise Option / Vendor Option buttons (home page and Services page): they now open the matching form and scroll it into view on desktop, laptop, tablet and phone. Nav links behave the same way.
- Franchise Application form: Full Name, Mobile Number, Email Address, State, District, City / Location, Complete Address, Investment Capacity, Preferred Franchise Location (required); Previous Business Experience and Additional Message (optional). Button: Submit Application.
- Vendor Registration form: Company / Business Name, Owner Name, Mobile Number, Email Address, State, District, City, Complete Business Address (required); Additional Information (optional). Button: Submit Registration. Type of Service, GST Number and PAN Number are removed from the form, the server and the admin panel.
- The server checks required fields, the 10-digit mobile number (starts 6-9) and the email address for website submissions, and returns the message to the form. Rules live in `backend/partner-application.js` (tests: `backend/test/partner-applications.test.mjs`).
- Admin panel > Client: two separate lists, **Franchise Applications** and **Vendor Registrations**. Each has search, a status filter (Pending / Approved / Rejected) and **Export Excel** of the rows currently shown. Approve is the same as the former Accept (it still creates the portal login).

- Hub Management (Admin Portal): the old hub form and its yellow "Hub rule" banner are replaced by a Hub Details form with the 15 fields (In-Charge, Phone, Hub Name, Location, Address, Pincode, City, District, State, Contact Number, Email, Manager, Opening Date, Status, URL). Hubs are stored on the server in the `hubs` table (API `/api/admin/hubs`, admin sign-in only), not in the browser. The 100 existing hubs load once on the first start; hubs saved in a browser by the older version move to the server once.
- Hub Management **Import Excel**: `.xlsx`, `.xls` or `.csv`. The header row is found automatically (title rows above it are fine). Preview shows the records to import, duplicates and errors; the import only fills blank fields of existing hubs and never overwrites saved values. Hub Code, POD and Region are kept for existing data.
- Tests: `cd backend && npm test` (includes `hubs.test.mjs` and `hub-excel.test.mjs`).
- Admin sign-in: admins can reset their own password from the sign-in page. A single-use link (30 minutes, sent only to the registered email) lets them choose a new password; old links stop working and other devices are signed out. Email settings are read from `backend/.env` and are never shown or stored. Admin email matching ignores capitals and stray spaces. The sign-in page is set up for phones (no zoom on tap, works at phone width). Vendor, franchise and rider-application logins are unchanged.
- New `node admin-accounts.js` lists the admin accounts (registered emails and status) without any passwords.
- Rider Applications are now Admin Portal only (plus own-rider approval for vendors). The Franchise Portal's rider application
  menu, page and controls were removed; the franchise Riders page is unchanged. The Admin Client page's older "Client Members" card is now "Client Member Assignments".
- Vendor & Franchise portal forms are checked before saving (Profile, Add Location, Add Rider, Helpline / Support, Profile Selfie):
  names (people, business, location, state / district / city) cannot contain numbers; numbers typed into a name are removed with a message.
  Contact / phone / mobile numbers must be exactly 10 digits starting with 6-9 ("+91 98765 43210" becomes 9876543210); PIN codes 6 digits;
  email checked. Problems are shown in red under the field. The support form is also checked on the server.
- Profile selfie (both portals): the captured photo no longer grows past the window. Clean square preview with a round guide
  (exactly what the avatar shows), Capture → check → Save Selfie / Retake, clear messages when the camera is blocked or busy.
- Admin payment Excel imports are shared with the matching vendor / franchise portals (Orders filled in) and those partners get a notification.
- Riders: Rider ID / Partner ID and name are filled in from the app screenshot (read on the server); Mobile Number accepts exactly 10 digits.
  The first start after this update downloads about 60 MB of text-recognition files (done automatically by the start script).
- Riders: new fields (Rider ID / Partner ID, Mobile Number, Hub Pincode, Hub Location) and private Screenshot + Aadhaar uploads in both portals.
- Vendor & Franchise portals: **Location URL** (e.g. Google Maps link) is required when adding a location, shown as "Open map ↗" in My Locations. Only http/https links are accepted.
- Vendor & Franchise portals: ↻ Refresh button on Orders and Riders (plus automatic update every minute / on returning to the tab).
- Vendor / Franchise IDs count up from VN-000001 / FP-000001 using a stored counter, so no ID is ever given out twice.
- Real logins: accepting a vendor / franchise application creates their portal login (temporary password shown once, copy / WhatsApp / email);
  first sign-in forces a new password; each partner's portal data is stored on the server under their account; reject / undo switches the login off;
  admins can reset partner passwords. Admin sign-in moved to the server (hashed passwords, secure session cookies, forced password change,
  Settings > Change password, reset script); the admin passwords and the trusted X-Admin-Email header are gone; admins start with random first-run passwords.
  Security review fixes: applicants can't plant admin notes, field length caps, application rate limit, unknown web addresses refused, support tickets only carry a partner ID when signed in, safer error messages. Franchise portal IDs now use FP- (FR- still accepted).
  On phones, the portal menu now closes after choosing a page.
- Admin sidebar simplified: Vendor Portal, Franchise Portal and Partner Applications buttons removed; applications are managed under Client.
- Added the Franchise Management Portal at /franchise/. Vendor and franchise portals now keep separate browser data (they used the same storage keys and would have overwritten each other); support emails say which portal and account sent them.
- Added the Vendor Management Portal at /vendor/, run by the same server and start script. Its support email endpoint now lives in `backend/server.js` (upgraded to patched multer 2.5 / nodemailer 10.2, with an 8 MB photo limit and spam limit).
- Accept / Reject buttons for Franchise and Vendor applications, with optional note, undo, decision history and status filter.
- Services page forms now send applications to the server (before, they only saved in the visitor's own browser and never reached the admin panel).
- Home and Services pages share one submit script, so they cannot drift apart again. The Services page also gained the missing Vendor "Full Address" field and "Staff / Admin Login" link.
- Phone number check works again (the old pattern was ignored by current browsers, so any text was accepted).
- If the postal PIN lookup fails, applicants can type their PIN code instead of leaving it blank.
- Admin panel: "+ New Vendor" / "+ New Franchise" on the Client page now actually save; admin edits and status changes are saved on the server (before, they were undone on the next refresh); counts update correctly; real status shown instead of always "Active"; auto-refresh, new-application pop-up and sidebar counter.

## Payment Management: Excel import and Weekly Payments
- **Import Excel** (Admin panel > Payment Management) reads the sheet, finds the header row by itself (title rows above it are fine) and matches the columns by name:
  WEEK, RIDER (and Rider ID), DAYS, LM, PRIMARY, CLUB and the amount column (e.g. *Weekly Amount*, *Payment Completed*, *Net Payable*, *Payout Amount*). Three layouts work: one row per rider per week,
  one row per rider per day (grouped into Monday-Sunday weeks, days counted, amounts added up) and one column per date.
- **The weekly amount is never calculated or defaulted.** It is the number in the sheet's amount column. A blank cell is shown as **—**; a real 0 is shown as ₹0.00.
  The panel also lists which sheet column was used for each field, and every row that was not imported, with its row number and the reason.
- Rows are saved in the `weekly_payments` table of `backend/payments.sqlite` (one row per rider and week; the original cell text and the original row are kept). They stay after a refresh, on every computer, and are shared by all admins.
- **No duplicates:** a rider + week is stored once. Importing the same file again changes nothing ("This file had already been imported"); importing a corrected sheet for a week updates those rows; a sheet without Rider IDs is matched to the same week's saved rows by name.
  Another week is simply added next to the existing weeks.
- The search box finds rider name, Rider ID and week (`19/10/2026`, `2026-10-19` or text from the sheet).
- Sample sheets and tests: `backend/test/fixtures/*.xlsx`, `cd backend && npm test` (includes `weekly-payments.test.mjs`). Manual Adjustment is unchanged; its entries are listed in the weekly table marked **Manual**.
- Payments imported with older versions were only kept in each browser (not in the database). They stay in Daily Payment Records; import the sheet once more to put the weeks into the database.
- Franchise Portal > Orders includes the same 15-column Weekly Payments history table. It reads from the shared database and only returns rows for approved riders registered to the signed-in franchise account. Search, cycle filters, totals and Back / Next pagination are available there.

## Bundled rider payment data workbook

- The supplied workbook is included at `backend/data/rider-payment-database.xlsx` as the source/reference payment dataset.
- The live application database is SQLite at `backend/payments.sqlite`; the backend creates and upgrades its tables automatically when the server starts.
- The workbook currently contains rider names and payment figures but does not provide the rider registration fields (Rider ID, mobile number, hub details, required documents, and approval status). It is therefore kept as a source workbook and is **not** silently converted into approved rider registrations.
- Payment imports are matched against existing rider registrations. To resolve unmatched rows, register/approve the riders using their correct Rider IDs, or add an explicit verified Rider ID-to-registration mapping. Do not match solely by similar names.
- Back up `backend/payments.sqlite` and `backend/uploads/` together. Keep the bundled workbook as a separate source copy.


### Applicant document uploads (Franchise / Vendor)
The website now collects Fresher / Experienced status, Aadhaar Card uploads, and (for Experienced applicants) 3 or 6 monthly salary slips. See `backend/README.md` for accepted formats, security controls, and the persistent-storage requirement for deployment.
