# SIF 2026 · Al Marwan Developments

Bilingual (EN / AR, RTL) registration landing page for the **Sharjah Investment Forum 2026** (14–15 October 2026, JRCC, Sharjah),
where Al Marwan Developments is a Silver Sponsor. Same design system as the Hawa and District 11 pages: Radikal and Noto Kufi Arabic,
the sand / cream / bronze palette, thin-line forms. The only logos are Al Marwan (English or Arabic, following the page language) and SIF.

## Files
| File | What it is |
| --- | --- |
| `index.html` | The landing page. Fonts, logos and code are all inside the file, so it can be uploaded on its own. |
| `thank-you.html` | One thank-you page for English and Arabic visitors: "Thank you" and "شكراً لك" together, the visitor's language first. Offers *Add to calendar* (Google, Apple / Outlook) and *Directions* to JRCC. Search engines are told not to list it. |

Arabic: add `?lang=ar` to the URL, or use the عربي / EN button (it switches in place and remembers the choice).

## The form
Fields follow the official SIF registration form. **Only email and mobile number are required**; everything else is optional:
first name, last name, registration type, gender, company / organisation, job title, nationality (every country, in the page language),
company industry and sectors of interest (tap-to-select chips). The reCAPTCHA is replaced by a hidden bot trap (`sif_hp`).

The registration type, industry and sector options are sensible defaults, not copied from the SIF site. Edit them in `index.html`
(the `<option>` and `.chip` lines) and their Arabic in the `AR` object in the script.

## Go-live settings
Edit `CONFIG` at the top of the `<script>` in `index.html`:

| Key | What it does |
| --- | --- |
| `formEndpoint` | URL that receives registrations (e.g. the Google Apps Script below). **Empty for now: demo mode.** Registrations are not saved (they are logged in the browser console), but the redirect to the thank-you page still happens so the flow can be tried. |
| `thankYouPage` | `thank-you.html`. |
| `metrikaId` | Yandex Metrika counter number, if you paste its tag. Registrations reach the goal `lead`. |
| `eventStart` / `eventEnd` | Countdown target: the start of 14 October (00:00 GST). During the forum the countdown becomes "open now", and afterwards it is hidden. |

Paste your tracking tags (GTM, Meta Pixel, Metrika, Snap, TikTok) between the `TRACKING` comments in the `<head>` of **both** files.

## Tracking
- **Conversion = a visit to `thank-you.html`.** It is one address for both languages, so a single goal ("page URL contains `thank-you.html`") counts every registration.
- `dataLayer` events: `generate_lead` on each registration (only when `formEndpoint` is set, so test entries don't count), also sent to gtag, Meta (`CompleteRegistration`), Snap and TikTok if they're installed; `contact_click` (call / email in the footer); `sif_thank_you` on the thank-you page, with `language`.
- UTM, gclid, fbclid and other click IDs are kept for the visit and sent with each registration.

Each registration is posted as form fields: `project`, `sponsor`, `registration_type`, `first_name`, `last_name`, `name`, `email`,
`phone` (one international number), `country_code`, `gender`, `company`, `job_title`, `nationality` (in English, whichever language
the page was in), `industry`, `sectors` (separated by `; `), `language`, `page`, `referrer`, `submitted_at` and any UTM / click IDs.

### Saving registrations to a Google Sheet
In a new Google Sheet: **Extensions → Apps Script**, paste this, then **Deploy → New deployment → Web app**
(execute as *Me*, access *Anyone*). Put the web-app URL in `formEndpoint`.

```js
var COLS = ['submitted_at','language','registration_type','first_name','last_name','email','phone','country_code','gender',
  'company','job_title','nationality','industry','sectors','page','referrer',
  'utm_source','utm_medium','utm_campaign','utm_content','utm_term','gclid','fbclid','ttclid','sccid'];
function doPost(e) {
  var sh = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Registrations')
        || SpreadsheetApp.getActiveSpreadsheet().insertSheet('Registrations');
  if (sh.getLastRow() === 0) sh.appendRow(COLS);
  var p = e.parameter;
  sh.appendRow(COLS.map(function (c) { return p[c] || ''; }));
  return ContentService.createTextOutput('ok');
}
```
