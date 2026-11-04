## Epic

**Title:** ECH: Customer visibility into proxy-layer logs (IP filter rejections)

**Summary:**
Elastic Cloud Hosted (ECH) enforces IP traffic filters at the Elastic-managed proxy tier, and customers can't see proxy logs. When legitimate clients are blocked, we can't identify the rejected source IPs ourselves and have to open an Elastic support case. This slows incident response and creates a dependency on a vendor for routine troubleshooting. This epic tracks getting a sustainable, self-service way to see proxy-level events for our deployments, without exposing other tenants' data or Elastic internals.

**Background:**
- IP filters operate at the proxy. Rejected requests never reach Elasticsearch, so deployment logs, monitoring and Elasticsearch audit logs don't show them.
- Elastic's docs describe proxy logs that record IP filter rejections as `status_reason: BLOCKED_BY_IP_FILTER` with a `client_ip` field. This is documented for ECE, and we have not confirmed that ECH exposes the same schema or any customer access.
- Proxy logs are shared infrastructure, so Elastic is likely to restrict access to protect other tenants. The ask is for a per-deployment, field-limited feed.

**Goal / desired outcome:**
We can review traffic filter rejections for our own deployments (source IP, timestamp, deployment, reason) without opening a support case, or at minimum have a documented, SLA-backed support process.

**Out of scope:**
- Access to raw, unfiltered proxy logs
- Changing the current traffic filter rule sets

**Acceptance criteria:**
- Elastic has given a written answer on whether per-deployment proxy log shipping, or an equivalent, is available for ECH
- One of the following is in place:
  - A working feed of filter rejection events (`client_ip`, `status_reason`, timestamp, deployment ID) into our monitoring deployment, **or**
  - A documented support process with an agreed turnaround time, and a feature request on file with our account attached
- Interim client-side detection is documented for the team (logging 403 "Forbidden due to traffic filtering" responses at our clients, load balancers or egress firewalls)
- A short runbook covers "how to identify a blocked IP" using whichever option is live

**Risks / dependencies:**
- Elastic may decline or put this on the roadmap with no date
- Depends on the Elastic account team or CSM and the support entitlement level

**Labels:** elastic-cloud, ech, observability, security, vendor-engagement

---

## Task (single)

**Title:** Engage Elastic on proxy log / IP filter rejection visibility for ECH, and document the interim approach

**Description:**
Work with our Elastic account team to find out whether per-deployment proxy logs, or filter rejection events specifically, can be made available for our ECH deployments, and put an interim detection and escalation process in place.

**Steps:**
1. Email the Elastic account team or CSM, and add a note to the existing support case, with these questions:
   - Can proxy logs scoped to our deployment IDs be shipped to our monitoring deployment or a data stream we own, with a field allowlist (`client_ip`, `status_reason`, status code, timestamp, deployment ID, request method and path)?
   - Is there an existing feature request or roadmap item for customer-visible traffic filter rejection logs? Please attach our account to it.
   - If a native feed isn't possible, can support provide a scheduled, redacted extract of IP filter rejections, and what is the response SLA?
   - Is there any setting or service we can enable today?
2. Record Elastic's answers and any case or feature request IDs in this ticket.
3. Document the interim detection steps (client-side 403 logging, egress IP inventory compared against rule sets, Elastic Cloud audit logs for rule set changes).
4. Write a short runbook for identifying a blocked IP, covering both the current support path and the self-service path if one becomes available.
5. If a feed is approved, set up ingestion, build a Kibana dashboard and an alert for spikes in rejections, and update the runbook.

**Done when:** All epic acceptance criteria are met, or Elastic has formally declined and the support-based process and feature request are documented.

**Priority:** Medium
**Suggested estimate:** 1 to 2 days of effort, plus vendor wait time

I used one task because the work is mostly one thread: ask Elastic, then document the result. If a feed is approved, the ingestion and dashboard work (step 5) could become a follow-up task under the same epic.
