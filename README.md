# Umstead Grove â€” FSBO Property Website

A professional For Sale By Owner real estate landing page for a home in Durham, NC, built on Firebase Hosting + Firebase Functions + Firestore + Firebase Storage.

---

## ðŸ“ File Structure

```
umsteadgrove/
  index.html      â€” Main property page (hero, gallery, form, FAQ, disclosures)
  admin.html      â€” Lead management dashboard (password-protected via API)
  style.css       â€” All styles
  main.js         â€” Frontend logic (form, gallery, config rendering)
  config.js       â€” â† EDIT THIS FIRST â€” all property details, prices, copy
  hero_bg.png     â€” Replace with your own hero background photo
  house.png       â€” Replace with your own property exterior photo
  kitchen.png     â€” Replace with your interior photos
  living_room.png
  bedroom.png
  bathroom.png
  backyard.png

functions/
  index.js            â€” Main Firebase Functions entry (Nest cam + FSBO exports)
  umsteadgrove.js     â€” All FSBO API endpoints + email templates
  .env.fsbo.example   â€” Secret key template
```

---

## ðŸš€ Setup Instructions

### 1. Fill in Property Details

Edit `umsteadgrove/config.js` and replace all `[PLACEHOLDER]` values:

- `property.address`, `price`, `beds`, `baths`, `sqft`, `lotSize`, `yearBuilt`
- `seller.email` â€” **critical**: this is where lead notifications will be sent
- `hoa` section with your actual HOA info
- `faq` answers specific to your property
- `gallery` â€” replace placeholder photos with real photos

### 2. Replace Placeholder Photos

Drop your actual property photos into `umsteadgrove/` and update `config.js â†’ gallery[]`:

```js
gallery: [
  { src: "front.jpg", alt: "Front exterior", caption: "Curb appeal" },
  // ...
]
```

Photos can be JPG or PNG. Recommended minimum: 1200Ã—800px.

### 3. Set Firebase Secrets

Run these commands from `C:\Projects\jorgeranilla.com` (requires Firebase CLI), because the shared Functions source lives there:

```bash
firebase functions:secrets:set UG_SMTP_HOST
firebase functions:secrets:set UG_SMTP_PORT
firebase functions:secrets:set UG_SMTP_USER
firebase functions:secrets:set UG_SMTP_PASS
firebase functions:secrets:set UG_SELLER_EMAIL
firebase functions:secrets:set UG_ADMIN_EMAILS
```

**Gmail tip**: Use a [Gmail App Password](https://support.google.com/accounts/answer/185833) â€” not your regular Gmail password. Enable 2FA on your account first.

**SendGrid alternative**: Set `UG_SMTP_HOST=smtp.sendgrid.net`, `UG_SMTP_PORT=587`, `UG_SMTP_USER=apikey`, `UG_SMTP_PASS=your-sendgrid-api-key`.

### 4. Install Function Dependencies

```bash
cd C:\Projects\jorgeranilla.com\functions
npm install
```

### 5. Deploy

```bash
# From project root
firebase deploy --only hosting:umsteadgrove
```

Or deploy individually:
```bash
# From C:\Projects\umsteadgrove.com
firebase deploy --only hosting:umsteadgrove

# From C:\Projects\jorgeranilla.com, because functions live there
firebase deploy --only functions:umsteadgroveSubmit,functions:umsteadgroveLeads,functions:umsteadgroveUpdateLead,functions:umsteadgroveDownload,functions:umsteadgroveSendUpdate
```

---

## ðŸ—„ï¸ Database Schema

**Firestore Collection:** `umsteadgrove_leads`

Each document contains:

| Field | Type | Description |
|---|---|---|
| `fullName` | string | Buyer's full name |
| `email` | string | Buyer email (lowercase) |
| `phone` | string | Buyer phone |
| `contactPreference` | string | email / phone_call / text |
| `preApprovalStatus` | string | yes / in_process / no / prefer_not |
| `lenderName` | string | Optional lender name |
| `budget` | string | Optional budget range |
| `hasBuyerAgent` | string | yes / no |
| `agentName` | string | Optional agent name |
| `agentContact` | string | Optional agent email/phone |
| `showingDateTime` | string | ISO datetime or empty |
| `message` | string | Optional buyer message |
| `consentUpdates` | boolean | Opted in to email/text updates |
| `ackNoContract` | boolean | Acknowledged no-contract disclaimer |
| `ackNoSensitive` | boolean | Acknowledged no-sensitive-data policy |
| `ackPrivacy` | boolean | Acknowledged privacy notice |
| `fileUploaded` | boolean | Whether a pre-approval was uploaded |
| `fileStoragePath` | string | Internal Storage path (not exposed publicly) |
| `leadStatus` | string | New / Contacted / Showing Scheduled / Viewed / Follow-Up / Offer Received / Not Interested |
| `notes` | string | Seller's private notes |
| `dateSubmitted` | Timestamp | Server-side submission time |

---

## ðŸ“‹ Managing Leads

### Admin Panel

Visit: `https://umsteadgrove.com/admin.html`

> **Note:** The admin panel uses Firebase Google Authentication. The backend only accepts signed-in Google accounts listed in `UG_ADMIN_EMAILS`. 
> To access it properly, you'll need to add auth headers. For a simpler workflow, use the Firebase Console directly.

**Quick lead management via Firebase Console:**
1. Go to [Firebase Console](https://console.firebase.google.com) â†’ `jorgeranilla-site`
2. Click **Firestore Database**
3. Navigate to `umsteadgrove_leads`
4. Click any document to view details and edit `leadStatus` / `notes`

### Export Leads to CSV

From `admin.html`, click **â¬‡ Export CSV**. This exports all currently-filtered leads to a CSV file.

You can also export from Firebase Console:
1. Firestore â†’ `umsteadgrove_leads` â†’ click â‹® menu â†’ **Export collection**

---

## ðŸ“§ Sending Bulk Email Updates

Use the Firebase Functions REST API or create a simple admin script:

```js
// Example: Send price update to all opted-in leads
fetch('https://us-central1-jorgeranilla-site.cloudfunctions.net/umsteadgroveSendUpdate', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer YOUR_ADMIN_PASSWORD'
  },
  body: JSON.stringify({
    type: 'price_update',
    payload: { newPrice: '$629,000', note: 'Price reduced for quick sale.' }
  })
});
```

**Email types:**
- `price_update` â€” payload: `{ newPrice, note }`
- `open_house` â€” payload: `{ date, time, address }`
- `follow_up` â€” no payload needed
- `showing_confirm` â€” payload: `{ confirmedDateTime }`

Only leads who checked "I consent to receive updates" receive these emails.

---

## ðŸ”’ Security Notes

- Uploaded pre-approval letters go to Firebase Storage and are **never publicly accessible**
- Downloads require admin authentication and return a 15-minute signed URL
- Form includes honeypot spam protection (`_honey` field)
- File uploads validated server-side: only PDF/JPG/PNG under 10MB accepted
- No SSNs, bank accounts, or tax return data is collected or requested
- Admin panel API protected by Firebase Google Auth and the `UG_ADMIN_EMAILS` allowlist

---

## âš–ï¸ Legal & Compliance

- Fair Housing language is included on the main page and in all emails
- Privacy notice explains data use is limited to property sale communications
- All required FSBO disclaimers are present:
  - Seller is not a licensed real estate agent
  - Form submission does not create a contract or guarantee a showing
  - Buyer is encouraged to work with their own professionals

---

## ðŸ› ï¸ Customization

All site copy lives in `umsteadgrove/config.js`. To update:

| What to change | Where |
|---|---|
| Price | `config.js â†’ property.price` |
| Photos | Replace files + update `config.js â†’ gallery[]` |
| Highlights | `config.js â†’ highlights[]` |
| FAQ | `config.js â†’ faq[]` |
| HOA info | `config.js â†’ hoa` |
| Seller email | `config.js â†’ seller.email` + Firebase secret `UG_SELLER_EMAIL` |
| Nearby points | `config.js â†’ neighborhood.nearbyPoints[]` |
| Google Maps embed | `config.js â†’ neighborhood.googleMapsEmbed` |

---

## ðŸ“Š Analytics

The site fires `gtag` events for:
- `cta_click` â€” all CTA button clicks
- `form_submit` â€” buyer form submission

Add your Google Analytics 4 tag to `index.html` `<head>` to enable tracking:

```html
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```




