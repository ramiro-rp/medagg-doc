# Admin Guide

This guide is for users with the **Admin** role in
Medical Imaging Dataset Aggregator.

---

## 1. Admin responsibilities

- keep the **dataset catalog** and vocabularies up to date;
- monitor **user requests** and collection statuses;
- manage **disk space** by removing stale ZIP archives;
- help diagnose problems with search and collection building.

---

## 2. Accessing the admin area

1. Sign in with an account that has admin privileges.
2. Open the **Admin** section in the main menu.

If you do not see the admin section, your account does not have the
required permissions.

<screenshot: admin-main-page>

---

## 3. Managing datasets

### 3.1 Creating a dataset entry

1. Go to **Admin → Datasets**.
2. Click **“New dataset”**.
3. Fill in the form. Recommended minimum:
   - title;
   - external source URL;
   - short description;
   - anatomical area;
   - modality (one or more);
   - ML task type(s);
   - estimated number of images;
   - approximate size (MB);
   - tags.
4. Save changes.

The dataset will become available in search results (if marked as active).

<screenshot: admin-dataset-create-form>

### 3.2 Editing and deactivating

1. In **Admin → Datasets**, find the target dataset.
2. Click **Edit** and adjust the fields you need.
3. To temporarily hide the dataset from search without losing its history,
   set it as **inactive** (e.g. `Active = false`).

<screenshot: admin-dataset-edit-form>

---

## 4. Managing vocabularies

The admin UI may include sections like:

- **Anatomical areas**;
- **Modalities**;
- **ML task types**;
- **Tags**.

Through these sections an admin can:

- add new values (e.g. a new modality or tag);
- correct names and descriptions;
- deactivate obsolete entries.

Consistent vocabularies are important for filters and the search module.

---

## 5. Monitoring requests and collections

The **Admin → Requests / Logs** section shows user queries and collections.

For each record you may see:

- user;
- query text or filter set;
- creation time;
- collection status (pending / ready / failed / expired);
- archive size (if available);
- error details (if any).

<screenshot: admin-requests-page>

This information helps to:

- understand which scenarios are most common;
- diagnose failures linked to particular queries or datasets;
- decide which old large collections to delete.

---

## 6. Storage management

Built collections are stored as ZIP files in a directory accessible to the
backend (local SSD or mounted volume). The system may implement an automatic
**retention policy** (removing archives older than N days).

Typical admin tasks:

- monitor disk usage;
- remove old or test collections (via UI or scripts, depending on implementation);
- ensure that retention policies behave as expected and do not delete
  important data.

---

## 7. Security

- never share admin credentials with third parties;
- restrict access to the admin UI from untrusted networks;
- keep OS and Docker images up to date with security patches;
- if you suspect a data leak or privilege misuse, inform the team lead / DevOps
  and rotate secrets (DB passwords, tokens, etc.) if needed.
