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

## Preview
- `get_landing_preview(funnel_id)` — returns the admin `preview_url` (open in browser) and a
  draft snapshot; use before publishing. Does not change publish status.

> Exact tool names may vary slightly; list the live tools with the client's tool browser
> if a name does not resolve.
