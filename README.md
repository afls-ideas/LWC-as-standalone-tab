# Patient Journey Simulator (LSC Mobile LWC)

A self-contained, deployable Lightning Web Component that renders a full-screen,
touch-first **patient journey funnel** with live what-if intervention modeling
for pharma sales reps. Built to run as a **custom tab / App Page in the Life
Sciences Cloud (LSC) mobile app**.

- **Component:** `lscMobileInline_patientSimulator` (master label **"Pat Sim"**)
- **App Page:** `Pat_Sim` (label **"Pat Sim"**)
- **API version:** 66.0
- **Verified on:** LSC mobile app **v262.3.0**

---

## What's in this package

```
patient-simulator/
├── sfdx-project.json
├── manifest/
│   └── package.xml
└── force-app/main/default/
    ├── flexipages/
    │   └── Pat_Sim.flexipage-meta.xml          # App Page hosting the LWC
    └── lwc/
        └── lscMobileInline_patientSimulator/   # the component bundle
            ├── lscMobileInline_patientSimulator.js
            ├── lscMobileInline_patientSimulator.html
            ├── lscMobileInline_patientSimulator.css
            └── lscMobileInline_patientSimulator.js-meta.xml
```

The component is **self-contained** — all data is mock, so it always renders
cleanly with no dependencies on org data, Apex, or custom objects.

---

## ⚠️ The one thing that makes mobile work: `lightning__UrlAddressable`

If you take nothing else from this README: the LWC **must** declare the
`lightning__UrlAddressable` target, or it renders **completely blank** in the LSC
mobile app (plain gray, no error, no "Coming Soon").

**Why:** the LSC mobile app does not embed the tab's page inline. It navigates to
a route and imports the LWC as a standalone URL-addressable module
(`@salesforce/urlAddressable/c__<component>`). That module is only generated when
the component declares `lightning__UrlAddressable`. Without it the import fails
*before any markup renders*, so no amount of CSS/layout tweaking will help.

The console signature (attach Safari Web Inspector to the mobile web view):

```
[MHC] routeTo received from native: {"attributes":{"componentName":"c__lscMobileInline_patientSimulator"},"type":"standard__component"}
[MHC] errorNavigate ... "code":"LWR4023","message":"Error importing view with name \"default\"", failure was: LWR3009:
      Unable to resolve bare specifier: @salesforce/urlAddressable/c__lscMobileInline_patientSimulator/v0_0_1
```

This is already fixed in the bundled `*.js-meta.xml`:

```xml
<targets>
    <target>lightning__Tab</target>
    <target>lightning__AppPage</target>
    <target>lightning__HomePage</target>
    <target>lightning__UrlAddressable</target>
</targets>
```

Two more conventions this component follows so it behaves like the other working
LSC mobile inline components (e.g. `lscMobileInline_sentimentMap`):

- **Name prefix `lscMobileInline_`** — matches the LSC mobile inline naming
  convention.
- **A `mobileHeight` Integer property** in `targetConfigs`, wired to the root
  element's height, so the funnel gets a real pixel height on mobile (the mobile
  container hands the component a zero/auto height).

---

## Prerequisites

- Salesforce CLI (`sf`) installed and authenticated to your target org.
- The target org is a Life Sciences Cloud org and you're testing in the LSC
  mobile app (v262.3.0+).

Authenticate if needed:

```bash
sf org login web --alias my-org
```

---

## 1. Deploy the metadata

From this `patient-simulator/` directory:

```bash
# deploy everything in the package
sf project deploy start --source-dir force-app --target-org my-org

# ...or deploy from the manifest
sf project deploy start --manifest manifest/package.xml --target-org my-org
```

This creates the `lscMobileInline_patientSimulator` component and the `Pat_Sim`
App Page. Deploying does **not** activate the page — do that next.

---

## 2. Activate the page in Lightning App Builder

Activation is what creates the navigable **custom tab**.

1. **Setup → Lightning App Builder**.
2. Open the **Pat Sim** page (filter the list by **"P"** if you don't see it).
3. Click **Save**, then **Activation...** (top-right).
4. In the **Activation** dialog:
   - **PAGE SETTINGS** — set the **Name** (this becomes the tab, e.g.
     `Pat Sim`), choose an **Icon**, and pick **Activate for all users** (or
     admins only).
   - **LIGHTNING EXPERIENCE** — add the page to one or more Lightning apps for
     desktop.
   - **MOBILE NAVIGATION** — add the page to the mobile navigation menu. **This
     is the step that surfaces the LWC in the LSC mobile app.**
5. **Save** the dialog, then **Save** the page.

---

## 3. Add the tab to a Lightning app (Navigation Items)

To place the tab in a specific app's nav bar (e.g. **Life Sciences Commercial**):

1. **Setup → App Manager →** your app **→ Edit** (or in App Builder: **App
   Settings → Navigation Items**).
2. Under **Available Items**, search for the tab (e.g. type `pat`) and select
   **Pat Sim**.
3. Click **▶** to move it into **Selected Items**, then use **▲ / ▼** to order
   it.
4. **Save**.

---

## 4. Verify on mobile

1. **Cold-restart** the LSC mobile app — fully swipe-close it, don't just
   background it. The app caches the LWC bundle aggressively; a pull-to-refresh
   is **not** enough to pick up a redeploy.
2. Open the **Pat Sim** tab. The patient journey funnel should render
   full-screen.

Adjust the height via the **Mobile Height** property on the component in App
Builder if needed.

---

## Troubleshooting

| Symptom | Cause / fix |
| --- | --- |
| Blank gray screen, no error | Missing `lightning__UrlAddressable` target (see top). Confirm via Safari Web Inspector → `LWR4023`/`LWR3009`. |
| Blank after a redeploy | Stale cached bundle. **Cold-restart** the mobile app (swipe-close + reopen). |
| Component not in App Builder palette | `isExposed` is `false`, or `lightning__AppPage` target missing. |
| Tab created but not visible to users | Tab visibility is Hidden for the profile — set it in **Setup → Profiles → Tab Settings** or via a permission set. |
| Page has no URL / not navigable | Page deployed but never **activated** — re-open in App Builder → **Activation...**. |
| Renders on desktop but blank on mobile | Almost always the `lightning__UrlAddressable` gap, not CSS. |
