---
mission: NOX-5
title: 'Secure SMS credential management'
role: business
status: ai_drafted
version: 1
author: NoX
ai_drafted: true
---

# Business requirement: Secure SMS credential management

## The request
"Migrate notifications-hub channel configuration to dynamically resolve SMS_API_KEY from Secret Manager. Remove the static fallback key from source code and update channel initialization tests."

## Problem
Text message provider credentials currently include a permanent backup key stored directly in the program code. This creates a compliance and security risk for our customer messaging systems if credentials ever need to be rotated or secured.

## Who is affected
No customers or staff are directly impacted during day-to-day operations.

## What should change
Nothing should look or behave differently for customers or staff. Outbound text messages will continue to send as usual, but the system will retrieve its provider security keys from our secure central vault.

## What "done" looks like
Customer text messages continue to go out promptly and reliably, with all access keys managed securely behind the scenes.

## Examples
None: nothing changes for customers or staff.

## Verification checklist
- [ ] Customers continue to receive text message updates and policy alerts as normal.
- [ ] Support and operations teams see outbound text messaging working without interruption.
