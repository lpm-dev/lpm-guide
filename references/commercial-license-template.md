# Commercial License Template

A starting point for the `LICENSE` of a Marketplace package. LPM.dev doesn't supply, review, or endorse license terms; the author owns them. The Marketplace Seller Agreement requires them to define use, modification, seat or organization scope, redistribution, subscription expiry, refunds, and commercial restrictions, and to match the package page. This isn't legal advice: tell the author to have a lawyer review it for their jurisdiction.

## How to Use

1. Save the text below as `LICENSE` at the package root. Fill every `{{...}}` placeholder and delete the bracketed options that don't apply.
2. In `package.json`, set `"license": "SEE LICENSE IN LICENSE"`. That earns the `license` quality check (5 points), and the root `LICENSE` file earns `license-file` (2). `"UNLICENSED"` earns 0 for `license`.
3. Keep section 4 if the package has ever sold a commercial plan ("Commercial license (allows bundling)"). Licenses bought from an archived commercial plan stay valid.
4. List bundled third-party code and its licenses in the notices file named in section 7.
5. LPM.dev enforces payment, not these terms. A buyer keeps install access unless the purchase is fully refunded or reversed, and sellers can't revoke a license on LPM.dev. The author enforces breach termination (section 6d) directly.

## Template

```text
{{PACKAGE}} Commercial License
Copyright (c) {{YEAR}} {{AUTHOR}}. All rights reserved.

This license covers {{PACKAGE}} (the "Software"), sold by {{AUTHOR}} ("Licensor")
through the LPM.dev Marketplace. "Licensee" is the person or organization that
bought a license. The "Term" is perpetual for a one-time purchase and each paid
billing period for a subscription. By installing or using the Software, Licensee
accepts these terms.

1. LICENSE GRANT
   Subject to payment, Licensor grants Licensee a non-exclusive, non-transferable
   license to install, use, and modify the Software within the purchased plan:
   a. Individual license: one named person per purchased seat. Seats may not be
      shared. An organization may reassign a seat to another of its members.
   b. Organization license: all members of the purchasing organization.

2. END PRODUCTS
   Licensee may include the Software, modified or not, in its own applications,
   websites, and services ("End Products") and distribute End Products to end
   users, as long as the Software can't be used or obtained separately from them.

3. RESTRICTIONS
   Except as section 4 allows, Licensee may not: (a) redistribute, sublicense,
   sell, or publish the Software or its source code on its own, or as part of a
   library, SDK, component kit, template, or package for other developers;
   (b) share seats or access outside the licensed scope; (c) remove copyright or
   license notices.

4. BUNDLING (only with a Commercial License purchase)
   If Licensee bought a plan marked "Commercial license (allows bundling)",
   Licensee may also bundle the Software inside Licensee's own packages
   distributed through the LPM.dev Marketplace. Bundled copies may be used only as
   part of that package. Its customers get no separate right to use the Software.
   If the Commercial License ends, Licensee's already-published releases may stay
   available, but Licensee may not publish new releases that bundle the Software.

5. VERSIONS AND SUPPORT
   This license covers every version of the Software made available to Licensee
   through LPM.dev during the Term. Licensor isn't obliged to release updates or
   provide support [, except: {{SUPPORT_TERMS}}].

6. TERM AND TERMINATION
   a. One-time purchase: perpetual, unless the payment is fully refunded or reversed.
   b. Subscription: for each paid billing period. When the subscription ends,
      Licensee [OPTION A: may keep using copies obtained during the Term in End
      Products already distributed, but may not start new End Products or install
      new versions] [OPTION B: must stop using the Software and delete all copies].
   c. A full refund or reversal ends this license immediately. Refunds follow the
      LPM.dev Marketplace Seller Agreement and applicable law.
   d. This license ends automatically if Licensee breaches it and doesn't cure the
      breach within 30 days of notice. On termination, Licensee must stop using
      and delete the Software, except as section 6(b) allows.

7. THIRD-PARTY COMPONENTS
   Third-party code in the Software is licensed under its own terms, listed in
   {{NOTICES_FILE}}.

8. NO WARRANTY
   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
   IMPLIED, INCLUDING MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AND
   NON-INFRINGEMENT.

9. LIMITATION OF LIABILITY
   TO THE EXTENT THE LAW ALLOWS, LICENSOR ISN'T LIABLE FOR INDIRECT, INCIDENTAL,
   SPECIAL, OR CONSEQUENTIAL DAMAGES, OR FOR LOST PROFITS OR DATA. LICENSOR'S
   TOTAL LIABILITY IS LIMITED TO THE AMOUNT LICENSEE PAID FOR THE SOFTWARE IN THE
   12 MONTHS BEFORE THE CLAIM.

10. GOVERNING LAW
    These terms are governed by the laws of {{JURISDICTION}}. The courts of
    {{VENUE}} have exclusive jurisdiction.

Contact: {{CONTACT_EMAIL}}
```
