# Maxim Sticker Collection

Customer-facing sticker collection prototype using the six stickers supplied in the design sheet.

## Package contents

- `index.html` — mobile-first interface, Thai / English / Russian, phone-number entry, collection grid, sticker unlock and promo-code dialog.
- `stickers/sticker-01.webp` … `sticker-06.webp` — six cropped sticker images.
- `Code.gs` — Google Apps Script API for customer lookup, campaign conditions, and single-use promo-code assignment.
- `sheets-template/` — example CSV headers and rows for the three Google Sheets tabs.

## Google Sheets setup

Create one spreadsheet with three tabs. Import the CSV template rows or create the same header rows manually.

### `Users`
`phone | city | orders | active`

- `phone`: customer's Maxim phone number; the backend normalizes Thai formats such as `081...` and `6681...`.
- `city`: city name. If the same phone has active rows in multiple cities, the app asks the customer to select a city.
- `orders`: trusted completed-order count. This must be maintained by an authorized data process; the starter does not connect to Maxim's order system.
- `active`: TRUE/FALSE.

### `Stickers`
`id | title | image | conditionType | conditionValue | conditionText | city | active | sortOrder | startDate | endDate`

Supported `conditionType` values:
- `always` — available to every active customer in the configured city/date window.
- `orders` / `minorders` / `min_orders` / `orders_gte` — unlock when completed orders are greater than or equal to `conditionValue`.
- `orders_exact` — unlock at exactly that order count.
- `orders_gt` — unlock when completed orders are greater than `conditionValue`.

`city` may be blank or `*` for all cities; comma-separated city names are supported. `startDate` and `endDate` are optional. `image` should be a relative path such as `stickers/sticker-01.webp`.

### `PromoCodes`
`code | value | status | stickerId | phone | assignedAt`

Preload the sheet with real, unique, valid Maxim promo codes. Set `value` to `50` and `status` to `available`. Do not deploy with the sample `EXAMPLE50` code. When a customer opens an eligible sticker, the server marks one code as `assigned` and records the sticker ID, phone and timestamp. Re-opening the same sticker returns the same code; the code is not reassigned to another customer.

## Configure and publish

1. Create the Google spreadsheet and its three tabs.
2. Open **Extensions → Apps Script** and paste the contents of `Code.gs`.
3. Replace `PASTE_GOOGLE_SPREADSHEET_ID_HERE` in `Code.gs` with the spreadsheet ID from its URL.
4. Deploy as a **Web app**, executing as the spreadsheet owner. Choose access settings that comply with your organization's policy.
5. Copy the Web app `/exec` URL and replace `PASTE_CUSTOMER_GOOGLE_APPS_SCRIPT_WEB_APP_URL_HERE` in `index.html`.
6. Publish the entire folder through GitHub Pages. Keep `index.html` and the `stickers/` folder together.
7. Test with a test phone and test promo codes before using real customers.

## API actions

The frontend sends JSON via POST:
- `{"a":"stickerState","pin":"66812345678","city":"Chiang Mai"}` — returns the user's sticker state and eligibility.
- `{"a":"openSticker","pin":"66812345678","city":"Chiang Mai","stickerId":"sticker-01"}` — validates eligibility server-side and assigns a single-use code.

The backend serializes claims with a script lock and re-checks eligibility before assigning a code. This is a starter implementation and must be tested in your own Apps Script deployment.

## Security and production caveats

- **Phone-only login is not identity verification.** Anyone who knows a customer's phone number may be able to view or claim their collection. This matches the requested no-SMS flow but is not recommended for valuable or sensitive rewards. Consider an authenticated Maxim link or another verification step before public launch.
- Configure Apps Script access carefully and do not expose spreadsheet credentials in frontend code. The spreadsheet ID is only in server-side Apps Script.
- Order statistics must come from a trusted source; users must not be able to edit their own order count.
- Promo codes must already be valid in Maxim. This app allocates codes; it does not create or activate them in the ride system.
- Review promotion terms, city/time limits and privacy requirements before production use.
