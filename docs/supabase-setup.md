# Setting up Supabase

What you need to do so the registration link actually receives applications.
Takes about twenty minutes. Nothing here can be done for you — the account has
to be created by whoever will own it.

---

## Before you start: who owns the account

Create it under a **new email that belongs to the project**, not your personal
one and not the account used for the MSME survey. Something like
`maketples@readydigital…` or a Division address.

This matters because ownership transfers to the Division at final payment
(Section 11 of the scope of work). Transferring a project is easy; untangling
a shared account is not.

---

## 1. Create the project

1. Go to **supabase.com** and sign up with that email.
2. **New project**.
3. Name: `maketples-enb`
4. Database password: generate a strong one and **write it down somewhere safe**.
   You will not be shown it again and it cannot be recovered.
5. Region: **Southeast Asia (Singapore)** — the closest to PNG, so the site
   feels faster than a US or European region.
6. Plan: **Free** is fine while building. Move to **Pro ($25/month)** before
   launch, because free projects are paused after a period of inactivity and an
   official platform going offline is not acceptable.

Wait a couple of minutes for it to finish setting up.

---

## 2. Run the database schema

1. Left sidebar → **SQL Editor** → **New query**.
2. Open `db/schema.sql` from this repository, copy all of it, paste, and press
   **Run**. It should finish with no errors.
3. New query again. Open `db/public-submissions.sql`, paste, **Run**.

Both files are safe to run more than once.

---

## 3. Create the photo bucket

Applications will not send photos until this exists.

1. Left sidebar → **Storage** → **New bucket**.
2. Name: `applications` — exactly that, lowercase.
3. **Public bucket: OFF.** Applicants upload into it, but only the Division may
   look inside.
4. Create it, then open its settings and set:
   - **File size limit:** `2MB` (the form shrinks photos well below this)
   - **Allowed MIME types:** `image/jpeg, image/png`

Those two limits are what stop the open upload being abused.

---

## 4. Make yourself Division staff

1. Left sidebar → **Authentication** → **Users** → **Add user**.
   Use your own email and a password you will remember.
2. Copy the **User UID** it shows.
3. **SQL Editor** → new query:

```sql
update profiles set role = 'division_admin'
where id = 'PASTE-THE-UID-HERE';
```

Without this you cannot see the application queue — the security rules will
correctly treat you as a member of the public.

---

## 5. Connect the form

1. Left sidebar → **Project Settings** → **API**.
2. Copy **Project URL** and the **anon public** key.
3. Open `register.html`, find the block near the bottom of the script:

```js
const SUPABASE = {
  URL: "",
  KEY: "",
  BUCKET: "applications"
};
```

4. Paste the two values in. Save, commit, push.

The amber "preview mode" bar disappears once those are filled in — that is how
you know it is connected.

### Is it safe to put that key in a public file?

Yes. The anon key is designed to be published; it identifies the project, it does
not grant permission. Row level security decides what it can do, and it has been
tested: with that key a visitor can submit an application and nothing else. They
cannot read applications — not even their own.

**What must never go in these files** is the `service_role` key. That one bypasses
every security rule. It belongs only in server-side code, never in a web page.

---

## 6. Test it before sending the link out

1. Open your published `register.html` and submit a real application with
   two or three photos.
2. In Supabase → **Table Editor** → `seller_applications` — your row should be there
   with a reference like `ENB-A1000`.
3. Check `application_products` and `application_photos` for the related rows.
4. **Storage → applications** — your photos, renamed like
   `Kokopo_Vunamami-Growers_frontpage_01.jpg`.

If all four are present, the link is ready to send.

---

## 7. Send the link

```
https://<your-user>.github.io/<repo>/register.html
```

Short enough to send by SMS or WhatsApp. Once the `.com.pg` domain is approved
it becomes something like `maketples.com.pg/register`.

---

## Approving an application

For now, from the SQL editor:

```sql
select approve_application('THE-APPLICATION-UUID');
```

That creates the business, marks it approved, and copies its products across as
drafts awaiting review. The Division panel will do this with a button once it is
connected to the database.

---

## What this does not do yet

- The marketplace, seller portal and Division panel still run on sample data.
  Connecting them is the next step.
- Nobody is notified when an application arrives — check the table, or set up
  a Supabase email trigger later.
- Approved sellers cannot log in yet; accounts are issued manually.

---

## Costs

| | |
|---|---|
| Free tier | $0 — fine for building and testing |
| Pro tier | $25/month — required before launch |
| Storage | Included up to 100GB on Pro; photos are ~150KB each |

Per Section 7 of the scope of work, this is a Division operating cost, not part
of the professional fee.
