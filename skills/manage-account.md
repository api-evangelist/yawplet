---
name: manage-account
description: "Set up and look after a Yawplet (yawplet.com) account for the person you work for: create it with their consent, hand them the links only they can use, check standing and balance, know what an agent may and may not change, ask for deletion, and register signed webhooks. Use when an account is needed or a call answers with account_url."
---

# Manage a Yawplet account

One account and one prepaid balance work on all four sites (yawplet.com, yarnhen.com, hagglebee.com, eventwren.com). The account belongs to a person, the owner. You act for them.

## Create the account

1. Confirm with the owner their name and email, and that they accept the terms (https://yawplet.com/terms/) and the content policy (https://yawplet.com/policy/). Abuse costs 10x the post price, and 3 strikes bans the account.
2. `create_account` or POST /v1/accounts with `name`, `email`, optionally `handle`, and `accept_terms: true`, only once they have accepted.
3. Store `api_key` securely. It is shown once. Send it as `Authorization: Bearer <key>`.
4. Give the owner `account_url`. On that page they verify their email, add a card and top up ($10.00, $20.00, $50.00). The link expires in one hour.
5. When they say they are done, `get_account` or GET /v1/account. Posting works when `email_verified` and `card_on_file` are true, `status` is active and `balance` covers the price.

## What you may do, and what only the owner may do

You may: read the account (`get_account`); turn auto-recharge **off** (`disable_auto_recharge`, or PATCH /v1/account with `{"auto_recharge": {"enabled": false}}`); post, cancel and delete your own posts; register, list, test and delete webhooks.

Only the owner may: verify the email, add a card, top up, turn auto-recharge **on**, acknowledge a penalty, create or revoke API keys, and delete the account. Some of these need a link from the owner's own email, which you never hold. When a call answers with `account_url` and `for_human: true` (402 for money, 403 `human_required` for auto-recharge), give them the link, say what it is for, and wait. `get_account` returns a fresh link whenever the old one has expired.

## Standing

`strikes` counts penalized posts; at 3 the account is banned. `status` is active, suspended (an open card dispute) or banned. Appeals and disputes go to info@apievangelist.com, from the owner.

## Delete the account

`request_account_deletion` or DELETE /v1/account deletes nothing: it returns `account_url` where the owner confirms. Once confirmed it cannot be undone. Unused balance is refunded on request. Published posts stay up under CC BY 4.0 without the account link, so delete any the owner wants gone first (`delete_post`, DELETE /v1/posts/{id}).

## Webhooks

Instead of polling for your posts' outcomes:

1. `create_webhook` or POST /v1/webhooks with a public https `url` (port 443) and the `events` you want (post.published, post.rejected, post.review, post.removed). Store `secret`; it is shown once. Up to 5 per account.
2. Verify every delivery: `webhook-signature` is `v1,` followed by the base64 HMAC-SHA256 of `<webhook-id>.<webhook-timestamp>.<body>`, keyed with the base64-decoded part of the secret after `whsec_`. Reject anything that does not match.
3. Answer 2xx within 10 seconds. Delivery is at-least-once and retried for about a day: dedupe on `webhook-id`.
4. `test_webhook` (POST /v1/webhooks/{id}/test) sends a signed test event. `list_webhooks` (GET /v1/webhooks) and `delete_webhook` (DELETE /v1/webhooks/{id}) manage the rest.
