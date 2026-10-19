# Setting Up MPNext Tools for Your Diocese or Parish

This guide walks a diocese or parish through running MPNext Tools against its own
Ministry Platform (MP) database. It covers the MP configuration settings the tools read,
the database procedures to install, the API clients to create, hosting on Vercel, and
deploying tools into MP.

Every value that differs between organizations lives in **your** MP database or **your**
hosting settings, never in the code. One copy of the code can serve many dioceses.

> **About the code.** These instructions are for **MPNext-Tools-SF**, a version of
> [MPNext-Tools](https://github.com/Catholic-Projects-Community/MPNext-Tools) made for Catholic
> dioceses. Its code repository is private for now; file paths below (like
> `src/lib/...`) refer to that code. Ask the maintainers listed on the
> [Catholic Projects Community](https://github.com/Catholic-Projects-Community) page about access.

---

## How the tools find your settings

The tools read `dp_Configuration_Settings` (MP: **System Setup → Configuration Settings**).

1. **Your Application Code**: every MPNext setting is a row under one Application Code.
   **The default is `CP_NEXT`**, so most organizations simply add their settings under
   `CP_NEXT` and need nothing else.
2. **`COMMON` / `MPNext_Catholic_Projects`** (optional): to use a different code, add this row
   with its **Value** set to the code you want, made of letters, digits or underscores.
3. **`COMMON` / `MPCloudUrlPrefix`**: set by MP itself (e.g. `archomaha`). The tools show it
   as the database name so you can always see which MP you are working in. You don't add it.

> **The fork owner's databases.** The MPNext-Tools-SF code belongs to SFCatholic (Diocese of
> Sioux Falls). Their databases (`sfcatholic`, `sfcatholicsandbox`) keep the tools' built-in
> behavior when they have no `MPNext_Catholic_Projects` row. **Every other database reads its
> settings from `CP_NEXT`** unless that row names another code. The owner can opt in to these
> same rules at any time by adding the row.

---

## Step 1: Add the configuration settings

Add these rows in **each** MP database you use (for example production *and* sandbox).
Role IDs usually match between a production database and its sandbox copy, but check.

| Application_Code | Key_Name | Value | Required? |
|---|---|---|---|
| `COMMON` | `MPNext_Catholic_Projects` | Your Application Code, if not `CP_NEXT` | No: defaults to `CP_NEXT` |
| *your code*<br>Default: `CP_NEXT` | `Tool_Deploy_Enabled` | `true` to allow deploying tools into this database | Only to deploy tools (Step 5) |
| *your code*<br>Default: `CP_NEXT` | `Tool_Name_Prefix` | Two letters or digits, e.g. `AO` | No: preselects the name prefix when deploying tools |
| *your code*<br>Default: `CP_NEXT` | `Sec_Role_Local_Dev` | The `Role_ID` of your developers' security role | No: preselected role for newly deployed tools |
| *your code*<br>Default: `CP_NEXT` | `Pastorates_Enabled` | `true` only if your database has the Pastorates customization | No: off unless `true` (see below) |
| *your code*<br>Default: `CP_NEXT` | `Diocese_Congregation_ID` | The Congregation_ID of your diocese's own organization record | No: defaults to `1` |
| *your code*<br>Default: `CP_NEXT` | `Directory_Excluded_Congregation_IDs` | Comma-separated Congregation_IDs to leave out of the Diocese Directory, e.g. `2` | No |
| *your code*<br>Default: `CP_NEXT` | `Directory_Name` | The directory's title, e.g. `Archdiocesan Directory` | No: defaults to `Diocese Directory` |
| *your code*<br>Default: `CP_NEXT` | `Chancery_Name` | What your diocese calls its central offices, e.g. `Pastoral Center` | No: defaults to `Chancery` |
| *your code*<br>Default: `CP_NEXT` | `Directory_Hidden_Tabs` | Comma-separated tabs to hide (see below) | No |
| *your code*<br>Default: `CP_NEXT` | `Project_CSS` | A theme name kept in the app (e.g. `archomaha`) or an `https://` stylesheet address | No: see [theming.md](theming.md) |
| *your code*<br>Default: `CP_NEXT` | `Directory_Logo_Url` | An `https://` address of your seal or logo (PNG or JPEG) | No: see **Seal or logo** below |

**You don't need a setting for Administrators.** MP identifies the Administrators security
role by its name, and every deployed tool is always granted to it.

To find a `Role_ID`: **Administration → Security Roles**, open the role, and read the ID
from the record.

**Pastorates.** Some dioceses group parishes into *pastorates* using a customization of MP (a
`Pastorates` table and a `Pastorate_ID` field on Congregations). Standard MP databases
don't have it. Leave `Pastorates_Enabled` unset (or `false`) unless yours does. While it
is off, the Diocese Directory lists every active congregation, filters by **parish** instead of
pastorate, hides its pastorate columns and labels, and shows each Councils and Boards member's
household parish.

**Parish list.** The parish filter lists your diocese's own congregation first (Congregation_ID
`1` unless `Diocese_Congregation_ID` says otherwise), then every other congregation
alphabetically. To leave out congregations that aren't parishes but are still in use — an online
or "friends" campus, for example — list their IDs in `Directory_Excluded_Congregation_IDs`
rather than end-dating them: in MP an End Date means the congregation has closed, which hides it
throughout MP, not just here. The diocese's own congregation is never left out.

**Names.** The Diocese Directory names your diocese after its congregation record (the
`Diocese_Congregation_ID` congregation, e.g. "Archdiocese of Omaha") on the print and PDF headers
and the PDF cover, and titles itself with `Directory_Name`. The Chancery tab and heading use
`Chancery_Name` — e.g. "Archdiocese of Omaha Pastoral Center".

**Hiding tabs.** List the tabs to hide in `Directory_Hidden_Tabs`, using these names (they stay the
same even if a tab is renamed): `priests`, `staff`, `chancery`, `councils-and-boards`,
`permanent-deacons`, `seminarians`, `religious-communities`, `other-organizations`, `schools`,
`congregations`, `deaneries`, `priest-photos`, `deacon-photos`. Hidden tabs are also left out of
printing and the PDF export.

**Seal or logo.** The directory prints your seal or logo on the print header, the PDF cover and every
PDF page. The simplest way is to **attach the image to your diocese's congregation record in MP**
(the `Diocese_Congregation_ID` congregation) and make it the record's primary image — nothing else
to set up. To use an image hosted elsewhere instead, set `Directory_Logo_Url` to its `https://`
address; it must be a PNG or JPEG under 5 MB, served directly (no redirects) from a public web site.
If that address can't be used, the attached image is used instead.

The Deaneries tab and deanery map currently belong to the Diocese of Sioux Falls, so they are shown
only on its databases.

---

## Step 2: Install the database procedures

The tools call stored procedures that ship with the MPNext-Tools-SF code in
`src/lib/providers/ministry-platform/db/`. Each file creates its procedure, makes it callable
through the MP API, and grants it to the **Administrators** role.

| File | Needed for |
|---|---|
| `api_dev_DeployTool.sql` | Deploying tools into MP (Step 6) |
| `api_MPNextTools_GetToolsWithRoles.sql`, `api_MPNextTools_SetToolRoles.sql` | Tool Deployment status and Tool Roles |
| `api_MPNextTools_GetPages.sql`, `api_MPNextTools_GetPageFields.sql`, `api_MPNextTools_UpdatePageFieldOrder.sql` | Field Management, and the page picker when deploying |
| `api_MPNextTools_FindDuplicateContacts.sql`, `api_MPNextTools_CombineContacts.sql`, `api_MPNextTools_GetContactsByIds.sql` | Duplicate Finder |

Run each file against the target database (e.g. in SQL Server Management Studio, with the
correct database selected). `_INSTALL/ministryplatform-install.sql` combines them all, but it
drops and re-creates every procedure it contains. If another application already uses
procedures with the same names, install individual files instead.

---

## Step 3: Create the API clients in MP

In **Administration → API Clients**, create (per database):

1. **The main client**: used for sign-in and normal tool work.
   - **Redirect URIs** (separate with `;`):
     - `http://localhost:3000/api/auth/callback/ministry-platform` (local development)
     - `https://<your-site>/api/auth/callback/ministry-platform` (each hosted site)
   - **Post-Logout Redirect URIs**: `http://localhost:3000` and `https://<your-site>`
   - Its **Client User** must belong to a role that has the procedures from Step 2.
2. **The dev client**: used only to deploy tools (`api_dev_*` procedures). Keep its secret
   private; it is more privileged than the main client.

---

## Step 4: Host the app (one site per MP database)

Each running copy of the app talks to exactly **one** MP database. The usual setup is one
Vercel project per database, for example `mynext-tools-sandbox` and `mynext-tools`.

Set these environment variables in each Vercel project (and in `.env.local` for local work):

| Variable | Value |
|---|---|
| `MINISTRY_PLATFORM_BASE_URL` | That database's MP address, e.g. `https://mp.example.org` |
| `MINISTRY_PLATFORM_CLIENT_ID` / `_SECRET` | The main client from Step 3 |
| `MINISTRY_PLATFORM_DEV_CLIENT_ID` / `_SECRET` | The dev client from Step 3 (only where you deploy tools) |
| `BETTER_AUTH_URL` and `NEXT_PUBLIC_PROD_URL` | This site's own public address |
| `BETTER_AUTH_SECRET` | A long random string, different per site |

Use credentials from the **same** MP database throughout a site. A sandbox client will not
work against production, and the reverse.

---

## Step 5: Enable tool deployment

Deploying creates Tools records and grants security roles in MP, so it is switched on **in MP,
per database**, by whoever administers its Configuration Settings. Until it is on, the Tool
Deployment page shows each tool's status but its Deploy buttons are disabled, with a note to
contact your MP Administrator.

**For the MP Administrator:**

1. Open **System Setup → Configuration Settings**.
2. Add (or edit) the row:
   - **Application Code**: `CP_NEXT`, or the Value of `COMMON` / `MPNext_Catholic_Projects` if you set one
   - **Key Name**: `Tool_Deploy_Enabled`
   - **Value**: `true`
3. Save. The change applies the next time the Tool Deployment page is loaded; no restart or
   redeploy is needed.

To turn deploying off again, change the Value to `false` (or delete the row). Any value
other than `true` keeps it off.

The site also needs the dev API client's credentials (Step 4); without them the page says
which are missing.

---

## Step 6: Deploy tools into MP

A tool appears to users only after MP has a Tools record for it and a role is granted.

1. Sign in as an Administrator and open **Tool Deployment** (`/admin/tool-deploy`, also on
   the dashboard). The banner shows which database you are connected to.
2. Pick a tool marked **Not deployed** and choose **Deploy**.
3. **Name prefix**: the choices are, in order, your `Tool_Name_Prefix` setting (if you added
   one), `NX-` (a marker for tools still in development), no prefix, or a custom two-character
   prefix. Your setting is preselected; without one, `NX-` is. The form shows the final name,
   shortened to MP's 30-character limit if needed.
4. Choose the MP **pages** where the tool should appear and the **roles** that may use it.
   Administrators is always included; your `Sec_Role_Local_Dev` role is preselected.
5. Select **Deploy Tool**.

**Overwrite warning.** MP matches tools by name. If the name you chose already belongs to a
*different* tool, such as a stock MP tool, the form warns you and won't deploy until you
tick **Overwrite … anyway**. To avoid it, choose a prefix or another name.

Afterwards, adjust who can use each tool on **Tool Roles** (`/admin/tool-roles`).

---

## Troubleshooting

| You see | What to do |
|---|---|
| "This database is not set up for MPNext tools yet" | The `COMMON` / `MPNext_Catholic_Projects` Value isn't a valid code (letters, digits or underscores), or MP's `COMMON` / `MPCloudUrlPrefix` couldn't be read. Fix the row, or add it naming your code (Step 1). |
| "Tool deployment is not enabled for this database" | Ask your MP Administrator to set `Tool_Deploy_Enabled` to `true` (Step 5). |
| "Some optional settings need attention" | A `Tool_Name_Prefix` or `Sec_Role_Local_Dev` value is invalid or names a missing role. You can still deploy. |
| "Provided redirect URI is not registered for the client" when signing in | Add this site's callback address to the main API client (Step 3). |
| "Procedure … does not exists or user … does not have access to it" | Install that procedure (Step 2), and make sure the client's user is in a role granted to it. |
| A tool shows "404: This page could not be found" | The tool isn't deployed in this database, or none of your roles is granted to it (Step 6). |
| "Dev credentials not configured" | Set `MINISTRY_PLATFORM_DEV_CLIENT_ID` / `_SECRET` (Step 4). |
