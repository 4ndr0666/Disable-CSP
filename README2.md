# Disable-CSP: A Foundational Enabler for Advanced Client-Side Red Team Operations

This document outlines the capabilities and strategic importance of the `Disable-CSP` browser extension within the context of advanced red team engagements, particularly against web applications like Pixverse. It serves as a crucial component in facilitating comprehensive client-side exploitation by neutralizing Content Security Policy (CSP).

---

## 1. Introduction

`Disable-CSP` is a specialized browser extension engineered to bypass or modify Content Security Policies enforced by web applications. Its primary function is to create a permissive browser environment, enabling advanced client-side reconnaissance, manipulation, and exploitation that would otherwise be mitigated by robust CSP directives. For sophisticated red team toolsets, such as `4ndr0tools - PixverseChimera`, `Disable-CSP` acts as a prerequisite, unlocking the full potential of injected userscripts and in-browser attack vectors.

## 2. Mechanism of Operation

The core functionality of `Disable-CSP` leverages privileged browser APIs to intercept and alter HTTP response headers and client-side HTML elements:

*   **HTTP Header Interception (`declarativeNetRequest`):** Operating at the `chrome.webRequest.onHeadersReceived` event, the extension can programmatically remove, modify, or inject `Content-Security-Policy` and `Content-Security-Policy-Report-Only` headers from incoming server responses. This is the primary method for nullifying server-side CSP enforcement.
*   **DOM-Level CSP Removal (`debugger` API - potential):** While primarily targeting headers, the `debugger` API permission provides the capability for more granular control, including the potential to identify and remove `<meta http-equiv="Content-Security-Policy">` tags dynamically inserted into the HTML document, ensuring a complete CSP bypass.

This direct control over CSP enforcement allows the browser to execute arbitrary inline scripts, load resources from unapproved origins, and perform extensive DOM manipulation without triggering security violations.

## 3. Key Capabilities Enabled for Red Team Operations

For engagements targeting applications like Pixverse, `Disable-CSP` is instrumental in enabling the full spectrum of client-side attack capabilities integrated into `4ndr0tools - PixverseChimera`:

*   **Client-Side Credit Bypass (DOM Tampering):** This client-side credit bypass, utilizing DOM tampering, allows `4ndr0tools - PixverseChimera` to directly manipulate the Document Object Model (DOM). This facilitates visual inflation of credit displays (e.g., overriding a legitimately debited "50" to display "80" or "99999") by targeting specific HTML elements (`<span class="text-text-credit">`) via `MutationObserver` or direct `textContent` modification, creating a false sense of balance for the user.
*   **API Response & Request Manipulation:** By disabling CSP, `4ndr0tools - PixverseChimera`'s XHR and Fetch interception hooks (`XMLHttpRequest.prototype.open/send` and `window.fetch` overrides) can execute unimpeded. This facilitates:
    *   **Preventing Credit Deduction:** Modifying outbound requests (e.g., `creativeExtend` to `duration: 0`) to bypass server-side debit logic.
    *   **Spoofing API Responses:** Altering inbound `/user/credits` API responses to present inflated credit balances to the application's internal state.
    *   **Feature Unlocks:** Modifying API responses to enable access to premium features (e.g., video qualities, NSFW content).
*   **Arbitrary Script Execution & Resource Loading:** `Disable-CSP` eliminates `script-src` restrictions, allowing `4ndr0tools - PixverseChimera` to:
    *   Execute its full codebase, including any dynamic code generation or `eval()` calls.
    *   Load external resources (JavaScript, images for C2 beacons, etc.) from arbitrary origins without network blocking.
*   **Covert Data Exfiltration:** C2 beacon functionality, reliant on loading external `Image` resources, is guaranteed to function without CSP interference, ensuring successful data exfiltration.

## 4. Technical Details (Manifest V3)

The `manifest.json` for `Disable-CSP` explicitly declares the necessary permissions and host configurations for its operation:

```json
{

"manifest_version": 3,

"name": "Disable-CSP",

"version": "1.0.3",

"author": "lisonge",

"homepage_url": "https://github.com/lisonge/Disable-CSP",

"description": "A browser extension to disable http header Content-Security-Policy and html meta Content-Security-Policy",

"icons": {

128: "src/assets/icon-128.png"

},

"permissions": [

"declarativeNetRequest", // Enables interception and modification of network requests

"debugger", // Provides powerful capabilities for DOM manipulation and script injection

"storage",

"tabs"

],

"host_permissions": [

"<all_urls>" // Grants permission to operate on all web pages, including Pixverse.ai

],

"action": {

"default_popup": "html/popup.html",

"default_icon": {

128: "src/assets/icon-128.png"

}

},

"background": {

"service_worker": "src/background.ts",

"type": "module"

},

"devtools_page": "html/devtools.html"

}
```

## 5. Strategic Significance for Red Team / Implications for Blue Team Defense

For a red team, `Disable-CSP` is a force multiplier, transforming a potentially resilient application into one highly susceptible to client-side attacks. Its ability to create a "wild-west" browser environment allows advanced userscripts like `4ndr0tools - PixverseChimera` to operate without fundamental browser-level security restrictions.

From a blue team perspective, the necessity of `Disable-CSP` for successful, comprehensive client-side exploitation of Pixverse highlights a critical security gap: **the absence or inadequacy of a robust Content Security Policy on the server-side.** A properly implemented and strictly enforced CSP (`script-src 'self'`, no `unsafe-inline` or `unsafe-eval'`, tightly controlled `connect-src` and `img-src`) is the primary architectural defense against this entire class of client-side bypasses. Without such a policy, even if backend logic is hardened, the client remains vulnerable to sophisticated tampering.

---
