# User Guide

This document explains how an end user (clinician, researcher,
student) interacts with **Medical Imaging Dataset Aggregator**.

---

## 1. Signing in

1. Open the application in your browser.  
   Example: `http://<server-host>/`
2. If you already have an account, click **“Log in”**.
3. If you do not, click **“Sign up / Register”** and fill in the form.

<screenshot: auth-page>

Exact labels and field names depend on the final frontend implementation.
Replace placeholders with real screenshots when the UI is stable.

---

## 2. Main screen (Search)

After signing in, you land on the **Search** page. Typically it contains:

- a **free‑text search** input;
- **filters** for:
  - modality;
  - anatomical area;
  - ML task type;
  - tags;
- a table with **search results**.

In the navigation bar you usually see:

- **Search**;
- **My datasets**;
- (for admins) **Admin**.

<screenshot: main-search-page>

---

## 3. Searching for datasets

### 3.1 Basic search

1. Enter a query in the search box.  
   Examples:
   - `pneumonia chest xray`
   - `пневмония грудная клетка КТ`
2. Optionally apply filters:
   - Modality = X‑ray;
   - Area = Chest;
   - Task type = Classification.
3. Click **Search**.

The table will show datasets that match the query:

- title, source, modality;
- number of images;
- relevance score (if the search module is enabled);
- controls to select datasets.

<screenshot: search-results>

### 3.2 Viewing dataset details

Click on a dataset row or a **“Details”** button to see:

- short description;
- link to the original resource;
- list of labels / pathologies (if available);
- example images (if implemented);
- approximate size.

---

## 4. Building a custom collection

### 4.1 Selecting datasets

On the search results page:

1. Select one or more datasets you want to include in the collection.
2. If the UI provides additional options, configure them, for example:
   - maximum number of images per dataset;
   - class balancing rules, etc.

<screenshot: select-datasets-for-collection>

### 4.2 Starting collection build

1. Click **“Build collection”** (exact label may differ).
2. Check the summary information:
   - how many datasets are selected;
   - approximate number of images;
   - approximate archive size.
3. Confirm creation.

The system will create a **collection build request** and either redirect you
to **My datasets** or show a notification.

---

## 5. “My datasets” (history)

The **My datasets** section shows previously built collections.

For each collection you will typically see:

- ID or name;
- original query text or short description;
- creation date;
- status:
  - `Pending` – still processing;
  - `Ready` – archive is available for download;
  - `Expired` – archive has been removed; the collection may be rebuilt;
- actions:
  - **Download ZIP** (if status is Ready);
  - **Regenerate** (if implemented).

<screenshot: my-datasets-page>

Clicking **Download ZIP** downloads an archive that usually contains:

- an `images/` directory with image files;
- a `metadata.csv` file with a table of metadata for each image
  (ID, labels, modality, source dataset, etc.).

---

## 6. Logging out

Click your profile icon or the menu item with your name and choose
**“Log out”**.

---

## 7. Common issues and tips

- **“No results found”**
  - Relax some filters or simplify the query.
  - Check whether the catalog actually contains datasets for that modality / task.
- **“Status stays ‘Pending’ for too long”**
  - There may be high server load or slow external sources.
  - If the wait time is clearly abnormal, contact the administrator.
- **“Download link does not work / collection expired”**
  - The archive may have been deleted by an automatic retention policy.
    Try rebuilding the collection or running the search again.
