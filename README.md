# GHL lead source capture demo

A small working demo by No BS AI showing how to keep Meta ad and boosted post source data on GoHighLevel contacts and opportunities.

## What is here

- `index.html`: a lead form page. It reads UTM params, `fbclid` and the referrer, stores a first touch (set once) and a last touch (updated each visit) in localStorage, writes the Meta `_fbc` cookie value, and fills hidden form fields. Submitting shows the webhook payload. No data is sent anywhere.
- `n8n/ghl-lead-source-upsert.json`: an n8n workflow. Webhook, then look up the contact in GHL by email, then build the upsert body, then call the GHL API contact upsert with `source`, tags and custom fields. First touch fields are only written for new contacts so they are never overwritten.

## Try it

Open the page with test UTMs:

```
?utm_source=facebook&utm_medium=paid_social&utm_campaign=2026-11_coach-webinar_lead&utm_content=video-a_hook1&fbclid=TEST123
?utm_source=facebook&utm_medium=boosted_post&utm_campaign=2026-11_boost_testimonial-post&utm_content=post-1103
```

## Using the n8n workflow

1. In GHL, create contact custom fields with these keys: `first_touch_source`, `first_touch_medium`, `first_touch_campaign`, `first_touch_date`, `last_touch_source`, `last_touch_medium`, `last_touch_campaign`, `utm_content`, `utm_term`, `fbclid`, `fbc`, `referrer`, `landing_page`.
2. Create a GHL Private Integration token with contacts read and write scope.
3. In n8n, add a Header Auth credential: name `Authorization`, value `Bearer <token>`. Attach it to both HTTP nodes.
4. Set the `GHL_LOCATION_ID` environment variable in n8n.
5. Import the JSON, activate, and point your form at the webhook URL.
6. In GHL, add a workflow on "Opportunity created" (or "Appointment booked") that copies the first touch fields onto the opportunity and sets opportunity source.

## UTM convention

| Parameter | Rule | Ads Manager | Boosted post |
|---|---|---|---|
| utm_source | platform | facebook / instagram | facebook / instagram |
| utm_medium | traffic type | paid_social | boosted_post |
| utm_campaign | yyyy-mm_offer_goal | 2026-11_coach-webinar_lead | 2026-11_boost_testimonial-post |
| utm_content | creative or post id | {{ad.name}} | post-1103 |
| utm_term | audience | {{adset.name}} | followers |

Limit: boosted posts that send people to Messenger, Instagram DM or instant forms never hit a web page, so UTMs cannot be captured there. Source must be set in the GHL workflow or integration for those.

MIT license.
