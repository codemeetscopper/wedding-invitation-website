# Wedding RSVP — Setup Guide

Two things to do: **(1)** connect responses to a Google Sheet, **(2)** put the page online.

---

## Step 1 — Collect responses in a Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) → **Blank** spreadsheet. Name it e.g. *Wedding RSVPs*.
2. In row 1, add these headers (order matters — **7 columns**):
   `Timestamp | Name | Attending | Guests | Meal | Message | Side`
3. Menu: **Extensions → Apps Script**. Delete anything there and paste the code below.
4. Click **Save** (disk icon).
5. Click **Deploy → New deployment**.
   - Gear icon → **Web app**
   - *Execute as:* **Me**
   - *Who has access:* **Anyone**
   - **Deploy** → authorize (click through the "unverified app → Advanced → Go to project" prompts; it's your own script).
6. Copy the **Web app URL** it gives you (ends in `/exec`).
7. Open `index.html`, find `PASTE_YOUR_GOOGLE_APPS_SCRIPT_URL_HERE` near the bottom, and paste your URL there.

### Apps Script code

```javascript
function doPost(e) {
  const lock = LockService.getScriptLock();
  lock.waitLock(20000);
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
    const p = e.parameter;

    // which link the guest used: ?g=bride -> "Bride", ?g=groom -> "Groom", plain link -> "General"
    const group = (p.group || '').toString();
    const side  = (group === 'Bride' || group === 'Groom') ? group : 'General';

    sheet.appendRow([
      new Date(),
      p.name      || '',
      p.attending || '',
      p.guests    || '',
      p.meal      || '',
      p.message   || '',
      side                 // <-- the Side column
    ]);
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'success', side: side }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}
```

> **Changing the code is not enough.** The `/exec` URL keeps running the old code until you publish a
> new version: **Deploy → Manage deployments → ✏️ Edit → Version: _New version_ → Deploy.**
> (Keep the same deployment so your URL doesn't change.)

To see just one side's replies, click the **Side** column → *Data → Create a filter* → filter for `Bride` or `Groom`.

---

## Step 2 — Put the page online (free public link)

**Easiest: Netlify Drop**
1. Put `index.html` (and your photo/video file, if any) together in one folder.
2. Go to [app.netlify.com/drop](https://app.netlify.com/drop).
3. Drag the folder onto the page. Done — you get a link like `your-name.netlify.app`.
4. (Optional) Free account → **Site settings → Change site name** to make the URL nicer.

**Alternatives:** GitHub Pages or Cloudflare Pages both host the same file for free.

---

## Step 3 — Two share links: bride's side & groom's side

You host **one** site, then share **two links** that differ only by a tag at the end.
Say Netlify gives you `https://jones-anamika.netlify.app` — share:

| Give to… | Link |
|---|---|
| **Bride's family & friends** | `https://jones-anamika.netlify.app/?g=bride` |
| **Groom's family & friends** | `https://jones-anamika.netlify.app/?g=groom` |

- Guests who open the bride's link see a small **"Bride's Guest"** badge on the RSVP form,
  and their reply lands in the **Bride** tab of your sheet. Groom's link → **Groom** tab.
- Anyone who opens the plain link (no `?g=…`) still works fine — those replies go to a
  **General** tab, so nothing is ever lost.
- The links are just the site URL with `?g=bride` or `?g=groom` added — nothing else to set up.

> Tip: when messaging people, paste the whole link including `?g=bride`. If you shorten it
> with a URL shortener, shorten the *full* tagged link so the tag is preserved.

---

## The couple artwork

The page displays **`couple.png`** — the watercolor couple with a true
transparent background (the baked-in checkerboard from the generated file
was removed programmatically; the original is backed up in `/images`).

- **Upload `couple.png` together with `index.html`** when you host the site
  (the `/images` folder itself is not needed online).
- To swap the artwork, replace `couple.png` with any transparent-background
  PNG — it blends into the page automatically.
- If `couple.png` is missing, the page falls back to a built-in SVG
  illustration of the couple.

## Customizing the page

Open `index.html` and search for `★ EDIT`:

- **Names / date / venue** — in the invite section markup.
- **Seal monogram** — the `J·A` text inside the seal SVG.
- **Meal options** — currently Veg / Non-Veg radio buttons.
- **Colors** — the `:root { … }` variables at the top (sampled from your invitation).
- **Map** — the Google Maps links use "Eden Convention Centre Valakom Muvattupuzha";
  edit both the `href` and the iframe `src` if the pin lands wrong.

---

## Testing before you share

- Open the page, fill it in, hit **Send RSVP**, then check the Google Sheet — a new row should appear.
- Until you paste the Script URL, submissions just log to the browser console (harmless), so set that up first.
