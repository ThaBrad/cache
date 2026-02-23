---

### Part 1: Structured Project Requirements

**Product Vision**
A public, continuously updated, comprehensive bibliography of English books published by Adventist companies. Designed as a research tool for scholars and laymen.

**Data & Content**

* **Scale:** ~10,000 entries (currently being manually transcribed from ~100,000 reference photos taken at the James White Library).
* **Current State:** Housed in Citavi (a Swiss citation manager).
* **Data Structure:** Complex citations including Author, Title, Publication Year, Place, Publisher, Series, Genre/Category, and Cover Image.
* **Growth:** The database will grow continuously as he works through the backlog and as new books are published.

**Required Features (UX/UI)**

* **Dynamic Filtering/Categories:** Ability to collapse/expand sections and filter by genres (cookbooks, theology, prophecy, children's literature).
* **Advanced Search:** Global search functionality across *all* data points (author, year, place, etc.).
* **Multiple View Modes:**
1. *List/Row View:* High-density data display (10-20 text rows) without covers.
2. *Grid/Cover View:* Standard catalog view with cover images and basic info.
3. *Gallery/Hover View:* Cover images only, with a tooltip/modal revealing the full citation on hover.



**Business & Operational Constraints**

* **Security:** Users should not be able to easily download/export the entire 10,000-entry database in one go.
* **Release Strategy:** Wants to publish immediately as a "live" work-in-progress rather than waiting until the data entry is 100% complete.
* **Friend's Technical Idea (To Pivot Away From):** Exporting Citavi to Excel/Google Sheets and embedding it into Squarespace or WordPress via custom code blocks.

---

### Part 2: Engineering Solution Draft

Since you are a frontend developer, the best approach is to decouple his data from the presentation layer using a modern headless architecture. This allows him to manage his data easily while giving you total control over the UI, performance, and search features.

#### 1. The Data Layer (Backend & CMS)

Google Sheets is terrible for a 10,000-row relational database that needs to serve images and complex searches. Squarespace is too rigid. WordPress is okay, but bloated for a pure data application.

* **The Solution:** Use a **Headless CMS** or a **Backend-as-a-Service (BaaS)**.
* *Options:* **Sanity**, **Directus**, or **Supabase**. Directus or Sanity are fantastic here because they provide a beautiful, out-of-the-box admin UI.


* **The Workflow:** He exports his current Citavi data to CSV. You write a quick script to ingest that CSV into the new backend. From that point on, he abandons Citavi for web publishing and inputs new books directly into the CMS admin panel. It solves his "live updates" requirement instantly.

#### 2. The Search & Filtering Engine

Querying 10,000 records with multiple filters (genre, year, publisher) and full-text search directly against a standard database can get sluggish and tricky to paginate efficiently on the frontend.

* **The Solution:** Depending on the backend you choose, you might want to pipe the data into a dedicated search service like **Algolia**, **Meilisearch**, or **Typesense**.
* *Why:* These tools are built specifically for the "faceted search" he wants (e.g., ticking checkboxes for "Theology" + "1980-1990" + "Review and Herald Publishing"). They return results in milliseconds, which is critical for smooth toggling between his requested view modes.

#### 3. The Frontend Application

This is your domain. You can build a fast, static-yet-dynamic site.

* **The Stack:** **Next.js** (React) or **Nuxt** (Vue), hosted on **Vercel** or **Netlify**.
* **Execution of Views:** Since the data is fetched via API (either from the CMS or Algolia), you handle the three view modes purely through frontend state (`const [viewMode, setViewMode] = useState('grid')`).
* *List View:* An HTML `<table>` or CSS Grid rows.
* *Cover View:* Standard CSS Grid with cards.
* *Hover View:* CSS Grid of images with absolute positioned tooltips or standard React modals on `onMouseEnter`.



#### 4. Image Management

Hosting 10,000 book covers is a bandwidth trap.

* **The Solution:** Store the images in an S3 bucket or use a CDN like Cloudinary, or leverage Next.js Image optimization (`next/image`). This ensures thumbnails are loaded for the "Hover View" rather than downloading 50 full-resolution images at once.

#### 5. Addressing the "No Bulk Download" Rule

* **The Reality Check:** You'll need to gently explain to your friend that if data is visible on the web, a determined person can scrape it.
* **The Mitigation:** You prevent *casual* bulk downloading by:
1. Not providing an "Export to CSV" button.
2. Using strict pagination (only loading 20-50 items per page/API call) or infinite scroll, rather than dumping the whole JSON payload to the client.
3. Implementing basic rate limiting on the API so a script can't just loop through `page=1` to `page=500` in two seconds.

---
