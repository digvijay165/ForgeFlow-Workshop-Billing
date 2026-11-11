# Turning on cloud sync, login and staff accounts

Without this setup ForgeFlow still works fully, but only on one device with no login.
These steps take about 10 minutes and are free (the Supabase free plan is plenty for a workshop).

## 1. Create a Supabase project

1. Go to [supabase.com](https://supabase.com) and sign up.
2. Click **New project**. Pick any name (e.g. `forgeflow`), set a database password (save it somewhere), and choose the region **Mumbai (ap-south-1)**.
3. Wait about a minute for the project to start.

## 2. Create the tables

1. In the project, open **SQL Editor** → **New query**.
2. Open [`supabase/schema.sql`](supabase/schema.sql) from this repo, copy everything, paste it in, and click **Run**.
3. You should see "Success. No rows returned".

## 3. Point login emails at your website

1. Open **Authentication** → **URL Configuration**.
2. Set **Site URL** to `https://digvijay165.github.io/ForgeFlow-Workshop-Billing/`
3. Under **Redirect URLs**, add the same address.

(Optional) If you don't want new accounts to confirm their email first, turn off
**Authentication → Providers → Email → Confirm email**.

## 4. Paste your keys into `config.js`

1. Open **Project Settings** → **API**.
2. Copy the **Project URL** and the **anon public** key.
3. Put them in [`config.js`](config.js):

```js
window.FORGEFLOW_CONFIG = {
  supabaseUrl: 'https://YOUR-PROJECT.supabase.co',
  supabaseAnonKey: 'eyJhbGciOi...',
};
```

The anon key is designed to be public. Your data is protected by the database rules from step 2:
only people in your shop can read it, and staff can't delete or change settings.
**Never** paste the `service_role` key here.

Commit and push. GitHub Pages updates in about a minute.

## 5. First sign-in (owner)

1. Open the website and choose **Create an account**.
2. Confirm your email if asked, then sign in.
3. Enter your shop name and click **Create my shop**.
   Any bills already saved in that browser are copied into the cloud.

## 6. Add staff

1. **Business settings → Team → Invite staff**, then enter their email.
2. They open the website, choose **Create an account** with that email, and they're in.

Staff can make bills, take payments, record purchases and update stock.
They can't delete anything, see reports or expenses, or change settings.

## 7. Save bills to OneDrive (optional)

This lets anyone using ForgeFlow paste the link to **their own** OneDrive Excel file
(**Business settings → Your data → ☁ Save bills to a OneDrive Excel link**). After they sign in with
Microsoft once, every saved bill rewrites that workbook online. You only do this setup once, and it's free.

1. Go to [portal.azure.com](https://portal.azure.com) and sign in with any Microsoft account
   (a personal Outlook/Hotmail account works).
2. Search for **App registrations** → **New registration**.
   - **Name:** `ForgeFlow`
   - **Supported account types:** *Accounts in any organizational directory and personal Microsoft accounts*
   - **Redirect URI:** choose **Single-page application (SPA)** and enter
     `https://digvijay165.github.io/ForgeFlow-Workshop-Billing/`
3. Click **Register**. On the next page, open **Authentication** and add a second SPA redirect URI:
   `https://digvijay165.github.io/ForgeFlow-Workshop-Billing/index.html`
4. Open **API permissions** → **Add a permission** → **Microsoft Graph** → **Delegated permissions**,
   tick **Files.ReadWrite.All**, and click **Add permissions**.
5. Go back to **Overview** and copy the **Application (client) ID**. Paste it into [`config.js`](config.js):

```js
  onedriveClientId: '00000000-0000-0000-0000-000000000000',
```

Commit and push. The client ID is not a secret. There is no client secret to create.

**For each user:** in OneDrive, open (or create) an Excel file → **Share** → **Copy link**, then paste
that link into ForgeFlow and sign in. Use an empty file kept just for ForgeFlow, because every save
replaces everything in that workbook with the latest bills. If the file is open for editing at the
moment a bill is saved, OneDrive may refuse the write; it is updated again on the next save.

## Good to know

- **Offline:** bills made without internet are saved on the device and sent when it reconnects.
  The sidebar shows "Saved to cloud" or how many changes are waiting.
- **Bill numbers:** if two devices make a bill at the same moment while offline, they can get the same number.
  Edit one of them if that happens.
- **Uploaded MRP Excel lists** belong to the shop that uploaded them. A shop can keep several lists
  (for example one per company); searches and bills look through all of them. Only the owner can add,
  replace, rename or remove a list; staff get them on every device, and other shops can never see them.
  No list is built in: each shop uploads its own.
- **Updating an existing project:** run `supabase/schema.sql` again (it is safe to re-run) to add or update the price-list tables. Do this before publishing a new version of the site.
