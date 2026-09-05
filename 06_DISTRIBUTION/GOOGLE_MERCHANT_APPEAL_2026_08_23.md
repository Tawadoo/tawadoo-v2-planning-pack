# Google Merchant Center + Google Ads Appeal — Tawadoo

**Date:** August 23, 2026  
**Account:** Tawadoo SARL (tawadoo.ma)  
**Issue:** Merchant Center suspension for misrepresentation → linked Google Ads account suspension

---

## APPEAL EMAIL (copy and submit via Merchant Center "Request a review")

---

Subject: **Appeal for Account Reinstatement — Root Cause Identified and Permanently Resolved**

Dear Google Merchant Center Team,

We are writing to appeal the suspension of our Merchant Center account and the linked Google Ads account suspension for Tawadoo (tawadoo.ma), Morocco's AI-powered marketplace.

**1. Root Cause — Identified and Acknowledged**

We identified the exact cause of the misrepresentation violation: our automated product feed system incorrectly pushed auction-type and negotiable-price listings to Google Shopping. These listings show a "starting price" that is not the final purchase price (buyers must bid or negotiate), which does not meet Google's requirement that the listed price must be the actual price a customer can buy at.

We fully acknowledge this violated Google's Misrepresentation policy. The listings did not accurately represent the purchase experience available to users clicking through from Google.

**2. Immediate Corrective Actions Taken**

- All product feeds have been completely emptied as of August 23, 2026. Zero products are currently submitted to Google Merchant Center.
- The automated feed generation system has been disabled and will not push any products until we have completed a full compliance review.
- All historical feed files (302 files across all channels) have been permanently deleted from our infrastructure.

**3. Permanent Technical Safeguards Implemented**

We have implemented a multi-layer prevention system in our codebase to ensure this never recurs:

a) **Product type routing enforcement:** Our system now enforces strict channel eligibility rules at the code level. Only "buy now" (fixed-price, immediately purchasable) listings may ever enter Google Shopping feeds. Auction, bid, negotiable, and offer-type listings are permanently excluded from all external channels and restricted to our internal platform only.

b) **Pre-feed validation:** Every product entering a Google feed must pass validation checks including: fixed price > 0, product available for immediate purchase, accurate availability status, valid landing page with matching price, and complete required attributes (title, image, brand, condition).

c) **Feed generation gated behind explicit approval:** Feed generation is now disabled by default and requires explicit founder authorization before any products are pushed to external channels. This is a permanent operational control, not just a code check.

d) **Merchant API compliance:** We migrated from the legacy Content API to the current Merchant API format with proper pricing (amountMicros), feedLabel (MA), and contentLanguage (fr) — ensuring all submitted data meets current Google specifications.

**4. Website Compliance**

Our website tawadoo.ma provides:
- Clear contact information (WhatsApp, email, phone)
- Published Terms of Use and Privacy Policy (CNDP-compliant)
- Accurate product pages where the listed price matches what the buyer pays
- Secure checkout (SSL/TLS)
- Clear return and shipping policies (COD via Tawssil delivery partner)
- Physical business address (Morocco)

**5. Request**

We respectfully request:
1. Review of our Merchant Center account for reinstatement — our feed is now empty and our prevention systems are in place
2. Once Merchant Center is reinstated, review of the linked Google Ads account suspension for reinstatement

We are committed to maintaining full compliance with Google's Shopping policies. We will only re-enable product feeds after completing internal QA and ensuring 100% compliance with the Shopping ads product data specifications.

We are happy to provide any additional information, screenshots of our technical controls, or access to our empty feed URLs for your verification.

Thank you for your time and consideration.

Best regards,
Ramzi Hannachi
Founder & CEO, Tawadoo SARL
tawadoo.ma
contact@tawadoo.ma

---

## IMPORTANT NOTES FOR SUBMISSION

1. **Submit via Merchant Center first** — go to Merchant Center → Account issues → "Request a review" button. Paste the core content (sections 1-5) in the appeal form.

2. **Google Ads follows Merchant Center** — per Google's documentation, the Ads suspension is a "linked account suspension." You must resolve the Merchant Center issue FIRST. Once MC is reinstated, the Ads account can be appealed separately.

3. **Timeline:** Google reviews within 5-7 business days typically. Do NOT submit multiple appeals (resets the queue).

4. **If rejected:** Wait the full cooldown period (usually noted in their response), make further visible changes (add more trust signals to the website), then re-appeal with documentation of what changed.

5. **Before re-enabling feeds (future — at production cutover):**
   - Run the GMC 11-item enable checklist (already documented)
   - Submit only 5-10 products initially
   - Wait for approval
   - Then gradually increase
   - Monitor the Diagnostics tab daily for the first 2 weeks
