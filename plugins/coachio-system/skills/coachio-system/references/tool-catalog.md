# Funnel Config MCP — Tool Catalog

Tools grouped by concern. ⚠ = destructive / real-effect → requires `confirm=true`.
Read tools and content-config tools run freely. Output mirrors the admin REST schemas.

## Products
- `list_products(type?, status?)` — list with totals (type: course|service|digital|other|agents_skill; status: draft|active|archived).
- `get_product(product_id)` — full product incl. linked courses.
- `create_product(name, slug, type, base_price?, status?, description?, thumbnail_url?, course_ids?)` — slug kebab-case + unique; `course_ids` not allowed for `agents_skill`. A funnel needs an existing product, so create one here first.
- `update_product(product_id, …)` — price/name changes refresh published funnels' landing cache.
- `delete_product(product_id, confirm)` ⚠ — permanent; may affect funnels referencing it.

## Funnels
- `list_funnels(product_id?, status?)` — list with totals.
- `get_funnel(funnel_id)` — full funnel incl. checkout/success config + variables.
- `create_funnel(product_id, title, slug, currency?, zalo_link?)` — starts `draft`.
- `update_funnel(funnel_id, title?, slug?, currency?, zalo_link?, checkout_config?, success_config?, variables?, variables_meta?)` — refreshes landing cache.
- `clone_funnel_tool(funnel_id, slug, title?)` — deep copy into a new draft funnel.
- `set_funnel_status(funnel_id, status, confirm)` ⚠ — publish/unpublish.
- `delete_funnel(funnel_id, confirm)` ⚠ — permanent.

## Landing & SEO
- `get_landing(funnel_id)`, `update_landing_seo(funnel_id, …)`.
- `list_sections(funnel_id)`, `create_section(funnel_id, …)`, `update_section(section_id, …)`, `reorder_sections(funnel_id, ordered_ids)`.
- `delete_section(section_id, confirm)` ⚠.
- Anchors must be unique within a landing; all writes refresh the landing cache.

## Variables
- `get_variables(funnel_id)`, `update_variables(funnel_id, variables, variables_meta?)`.

## Email templates
- `list_email_templates(funnel_id)`, `update_email_template(funnel_id, template_key, enabled?, subject?, html_body?)`, `preview_email_template(funnel_id, template_key)`.
- `test_send_email(funnel_id, template_key, to_email, confirm)` ⚠ — real send to one address.

## Broadcasts
- `list_broadcasts(funnel_id)`, `get_broadcast(cid)`, `create_broadcast(funnel_id, …)`, `update_broadcast(cid, …)`.
- `send_broadcast(cid, confirm)` ⚠ — real dispatch to the audience.

## Leads (read-only)
- `list_leads(funnel_id?, status?, email?, converted?, created_from?, created_to?, …)`.
- `export_leads(…same filters…)` — returns CSV text; deliver to the operator only.

## Discounts
- `list_discounts(…)`, `get_discount(discount_id)` — includes scopes + default owners.
- `create_discount(code, type, value, min_amount?, usage_limit?, scopes?, defaults?)`, `update_discount(discount_id, …)`.
- `set_discount_scope(discount_id, scopes)` — replace the applicability whitelist; empty = global.
- `set_discount_default(discount_id, owner_type, owner_id)` — auto-apply; must be within a non-empty scope (else error).
- `unset_discount_default(discount_id, owner_type, owner_id)`.
- `delete_discount(discount_id, confirm)` ⚠.

## Media
- `list_media(kind?, search?, page?, page_size?)`.
- `upload_media(filename, url? | base64_data?)` — upload from a URL or base64.
- `delete_media(asset_id, confirm)` ⚠.

## Analytics & Orders (read-only)
- `get_funnel_analytics(funnel_id, start_date?, end_date?)` — window capped at 31 days.
- `funnel_orders_summary(…)`, `list_funnel_orders(…)`, `get_funnel_order(order_id)`.

## Capture token
- `get_capture_token(funnel_id)` — current token (generated if absent).
- `rotate_capture_token(funnel_id, confirm)` ⚠ — old token stops working immediately.

## Lucky draw
- `list_lucky_events(funnel_id?)` — events with participant/winner counts.
- `get_lucky_event(event_id)` — full event incl. form schema.
- `create_lucky_event(funnel_id, title, slug?, form_schema?, name_field_key?, success_config?, display_config?)` — starts `draft`; `form_schema` is a list of field dicts; one short_text field is the display name.
- `update_lucky_event(event_id, …)` — re-validates the form schema.
- `set_lucky_event_status(event_id, action)` — `open` / `lock` registration.
- `delete_lucky_event(event_id, confirm)` ⚠ — removes prizes/participants/winners too.
- `get_lucky_token(event_id)`, `rotate_lucky_token(event_id, confirm)` ⚠ — public registration token.
- `list_lucky_prizes(event_id)`, `create_lucky_prize(event_id, name, quantity?, sort_order?)`, `update_lucky_prize(event_id, prize_id, …)`, `delete_lucky_prize(event_id, prize_id, confirm)` ⚠.
- `list_lucky_participants(event_id)`, `add_lucky_participant(event_id, display_name, phone?, answers?)`, `remove_lucky_participant(event_id, participant_id, confirm)` ⚠.
- `spin_lucky_prize(event_id, prize_id, confirm)` ⚠ — live draw; returns winner (id, display_name, phone, email, prize, spin_order). Undo with `discard_lucky_winner`.
- `list_lucky_winners(event_id)` — winners incl. id, phone, email (derived from participant; empty when not collected).
- `discard_lucky_winner(event_id, winner_id, confirm)` ⚠ — discard a winner (e.g. absent), removes the participant and frees the prize to redraw.

## Gifts
Gift packages (perks + external links) delivered two ways: **automations** (auto-grant on a
funnel trigger) and **campaigns** (one-off send to a targeted audience). A grant ledger makes
every delivery once-per-recipient; resends re-send the email only, never re-grant perks.

Packages + email composing:
- `list_gifts(include_archived?)`, `get_gift(gift_id)`.
- `create_gift(name, description?, internal_config?, external_items?)` — `internal_config` = `{unlock_agent_skills, studio_credits, enable_api_access}`; `external_items` = list of `{label, url, description?}`. A gift needs ≥1 internal perk or 1 external item.
- `update_gift(gift_id, …, is_archived?)`, `archive_gift(gift_id, confirm)` ⚠ — soft-archive (ledger kept); restore via `update_gift(is_archived=false)`.
- `preview_gift_email(gift_ids, email_subject?, email_html?)` / `test_send_gift_email(to_email, gift_ids, …)` — delivery email; tokens `{{recipient_name}}`, `{{agent_skills_url}}`, `{{studio_url}}` (URL only — wrap each in your own `<a>` button; empty unless the gift grants that perk).
- `preview_gift_account_email(email_subject?, email_html?)` / `test_send_gift_account_email(to_email, …)` — new-account login email; tokens `{{recipient_name}}`, `{{login_email}}`, `{{password}}`, `{{login_url}}` (URL only — wrap in your own `<a>` button). Blank → default template.

Automations (mechanism 1):
- `list_gift_automations()`, `get_gift_automation(automation_id)`.
- `create_gift_automation(trigger_status, gift_ids?, funnel_id?, is_active?, max_total_grants?, email_*?, account_email_*?)` — `funnel_id` null = all funnels. ⚠ non-`purchased` triggers grant paid perks to unpaid leads.
- `update_gift_automation(automation_id, …)`, `delete_gift_automation(automation_id, confirm)` ⚠.

Campaigns (mechanism 2) — lifecycle create → preview → confirm → send:
- `list_gift_campaigns()`, `get_gift_campaign(campaign_id)`.
- `create_gift_campaign(name, gift_ids?, email_*?, account_email_*?, audience_config?, scheduled_at?)`, `update_gift_campaign(campaign_id, …)` (draft only).
- `preview_gift_audience(campaign_id? | gift_ids? + audience_config?)` — recipient breakdown `{matched, already_granted, will_receive, sample}`.
- `confirm_gift_campaign(campaign_id)` — freeze recipient snapshot into send jobs.
- `send_gift_campaign(campaign_id, scheduled_at?)` — send now or schedule (ISO with +07:00).
- `cancel_gift_campaign(campaign_id)`, `retry_gift_campaign(campaign_id)`, `gift_campaign_stats(campaign_id, failed_page?, failed_size?)`.
- `delete_gift_campaign(campaign_id, confirm)` ⚠.

Grant tracking / audit:
- `list_gift_grants(page?, size?, gift_id?, funnel_id?, source_type?, email_status?, email?, content?, new_account_created?, date_from?, date_to?)`, `gift_grant_stats(…same filters…)`, `gift_grant_detail(grant_id)`.
- `resend_gift_grant(grant_id)`, `bulk_retry_gift_grants(…filters…)` — re-send failed emails only.

## Preview
- `get_landing_preview(funnel_id)` — returns the admin `preview_url` (open in browser) and a
  draft snapshot; use before publishing. Does not change publish status.

## Courses
- `list_courses(status?, category_id?)`, `get_course(course_id)`.
- `create_course(title, level, slug?, category_id?, thumbnail_url?, description?, short_description?, …, sale_price?, discount_percent?, product_id?, variables?, variables_meta?, success_config?)` — starts `draft`.
- `update_course(course_id, …subset…)` — does NOT change status; use `publish_course`/`set_course_status`.
- `delete_course(course_id, confirm)` ⚠ — permanent.

## Course categories
- `list_course_categories()`.
- `create_course_category(name, slug?, description?, parent_id?)`, `update_course_category(category_id, …)`.
- `delete_course_category(category_id, confirm)` ⚠ — fails if any course still uses it.

## Course publish + status
- `get_course_publish_readiness(course_id)` — checklist (course detail, curriculum, landing, upgrade page, payment) + `can_publish` + `draft_lesson_count` (lessons publishing will auto-promote).
- `publish_course(course_id, confirm)` ⚠ — gated by readiness; raises a tool error describing what's missing if not ready. Cascades to landing/upgrade pages + draft lessons.
- `set_course_status(course_id, status_value, confirm)` ⚠ — `draft`/`archived` only (use `publish_course` to publish); cascades landing/upgrade back to draft.

## Course curriculum
- `get_curriculum(course_id)` — sections + lessons + materials.
- `create_course_section(course_id, title, description?, sort_order?)`, `update_course_section(section_id, …)`, `delete_course_section(section_id, confirm)` ⚠.
- `create_lesson(section_id, title, lesson_type, lesson_status?, youtube_url?, pdf_url?, banner_url?, …)` — type: `text_lesson`|`pdf_lesson`|`video_lesson`; media via URL only, no file-upload tool.
- `update_lesson(lesson_id, …subset…)`, `delete_lesson(lesson_id, confirm)` ⚠.

## Course landing (per-section convenience over a whole-list upsert)
- `get_course_landing(course_id)` — sections + SEO + outline; errors if not created yet.
- `add_course_landing_section(course_id, section_type?, html?, name?, theme_mode?, anchor?, …)` — `section_type`: `hero` (one, kept first), `course_outline` (one), `custom`; auto-creates the landing on first call.
- `update_course_landing_section(course_id, section_id, …subset…)`, `delete_course_landing_section(course_id, section_id, confirm)` ⚠ — hero cannot be deleted.
- `reorder_course_landing_sections(course_id, section_ids)` — hero always kept first.
- `update_course_landing_seo(course_id, seo_title?, seo_description?, og_image_url?, canonical_url?, …)`.
- `set_course_landing_status(course_id, status_value, confirm)` ⚠ — `published` requires hero content.
- `ai_edit_course_landing_section(course_id, section_id, instruction)` — returns a suggestion only, does not save.

## Course email templates
- `list_course_email_templates(course_id)` — `verify`/`receipt` (custom or default) + variable tokens.
- `update_course_email_template(course_id, email_type, subject, html, enabled?)`.
- `preview_course_email_template(course_id, email_type)` — sample render, no send.
- `test_send_course_email(course_id, email_type, to_email, confirm)` ⚠ — real send.

> Exact tool names may vary slightly; list the live tools with the client's tool browser
> if a name does not resolve.
