# CookiePilot CMP — Google Tag Manager template

Google Tag Manager **Community Template** for [CookiePilot](https://cookiepilot.io/en?utm_source=gtm-gallery&utm_medium=marketplace&utm_content=github-readme),
a cookie consent platform for GDPR, ePrivacy and the Polish PKE. One tag on the
*Consent Initialization – All Pages* trigger shows the CookiePilot banner and
sends **Google Consent Mode v2** signals.

- Setup guide: <https://cookiepilot.io/en/integrations/google-tag-manager?utm_source=gtm-gallery&utm_medium=marketplace&utm_content=github-readme>
- Template type: custom tag template (sandboxed JavaScript), **Web** containers.
- Polski opis: [na dole tej strony](#po-polsku).

## What it does

1. **Sets Consent Mode v2 defaults** with `setDefaultConsentState`: every
   non-essential signal is `denied` before any tag fires, with
   `wait_for_update: 500`. `security_storage` is `granted`.
2. **Restores a returning visitor's choice**: reads the `cookiepilot_consent`
   cookie and calls `updateConsentState`, so Google tags see the stored decision
   before they run.
3. **Loads the CookiePilot banner** from `https://cdn.cookiepilot.io/cookiepilot.js`
   with `injectScript`.
4. The banner pushes **`cookiepilot_consent_update`** to the `dataLayer`, so you can
   trigger non-Google tags (Meta Pixel, TikTok, …) on consent changes.

| CookiePilot category | Consent Mode v2 signals |
|---|---|
| `analytics` | `analytics_storage` |
| `marketing` | `ad_storage`, `ad_user_data`, `ad_personalization` |
| `preferences` | `functionality_storage`, `personalization_storage` |
| `necessary` | `security_storage` (always `granted`) |

## Requirements

- A CookiePilot account with a verified domain and its **domain API key**
  (`cp_live_…`). You will find it in the CookiePilot dashboard under
  **Domain → Installation → Google Tag Manager**. A free plan is available.
- A GTM **Web** container on the site.

## Setup

1. In GTM, open **Templates → Search Gallery**, find **CookiePilot CMP** and add it
   to the workspace. (Before Gallery approval: **Templates → New → ⋮ → Import**
   and choose `template.tpl` from this repository.)
2. Create a tag with the **CookiePilot CMP** template.
3. Paste the domain **API key** (`cp_live_…`).
4. Set the trigger to **Consent Initialization – All Pages**.
5. Publish the container.

| Field | Required | Description |
|---|---|---|
| **CookiePilot API key** (`apiKey`) | Yes | The domain's public key, e.g. `cp_live_a1b2c3d4…`, validated against `^cp_live_[A-Za-z0-9]+$`. It identifies the domain and only fetches the banner configuration, so it is safe to expose in the browser. |

Keep this tag as the first consent tag in the container so the defaults are set
before anything else.

## Permissions

The template asks only for the permissions its code uses:

| Permission | Scope | Why |
|---|---|---|
| Accesses consent state (`access_consent`) | Read and write on the 7 Consent Mode signals | `setDefaultConsentState` and `updateConsentState` |
| Reads cookie values (`get_cookies`) | Only `cookiepilot_consent` | Restore a returning visitor's decision |
| Injects scripts (`inject_script`) | Only `https://cdn.cookiepilot.io/*` | Load the banner |

The template itself sends no network requests besides loading the banner, writes
no storage, and reads no other cookie or URL.

## Non-Google tags

On every consent change, and on load for returning visitors, CookiePilot pushes:

```js
dataLayer.push({
  event: 'cookiepilot_consent_update',
  cookiepilot_consent: { necessary: true, analytics: true, marketing: false, preferences: false }
});
```

Create a **Custom Event** trigger on `cookiepilot_consent_update` and fire each
non-Google tag only for the right category (for example Meta Pixel only when
`cookiepilot_consent.marketing` is `true`).

## Limits of a GTM install

CookiePilot can block trackers added after it loads. Tracker `<script>` tags
hard-coded in the page HTML, or tags GTM fires before this template, can run
before blocking starts — no GTM template can stop them afterwards. If your site
has such tags, install CookiePilot directly as the first script in `<head>` (or
use the [WordPress plugin](https://wordpress.org/plugins/cookiepilot/)) instead
of this template. Use one method per domain, not both.

## Removing the template

Pausing or deleting the tag stops the banner and the Consent Mode defaults. It
does not break other tags and does not delete consent records or the visitor's
`cookiepilot_consent` cookie. Without a CMP on the initialization trigger nothing
sets `denied` defaults, so add one back before relying on Consent Mode.

## Tests

`template.tpl` includes tests (`___TESTS___`) for: no stored cookie (denied
defaults + banner load), stored full consent, analytics-only consent and a
malformed cookie. Run them with **Run tests** in the GTM template editor.

## Support

Report problems in this repository's **Issues** tab, or write to
support@cookiepilot.io.

## License

Apache License 2.0 — see [LICENSE](LICENSE).

---

## Po polsku

Szablon Google Tag Managera dla [CookiePilot](https://cookiepilot.io/?utm_source=gtm-gallery&utm_medium=marketplace&utm_content=github-readme).
Jeden tag z regułą **Consent Initialization – All Pages** wyświetla baner
CookiePilot i wysyła sygnały **Google Consent Mode v2**: ustawia domyślne
`denied` przed innymi tagami, przywraca zapisaną decyzję powracającego
odwiedzającego i ładuje baner z CDN.

1. W GTM otwórz **Szablony → Szukaj w galerii**, znajdź **CookiePilot CMP** i dodaj go.
2. Utwórz tag z tego szablonu i wklej klucz API domeny (`cp_live_…`) z panelu
   CookiePilot (**Domena → Instalacja → Google Tag Manager**).
3. Ustaw regułę **Consent Initialization – All Pages** i opublikuj kontener.

Instrukcja krok po kroku: <https://cookiepilot.io/integracje/google-tag-manager?utm_source=gtm-gallery&utm_medium=marketplace&utm_content=github-readme>
