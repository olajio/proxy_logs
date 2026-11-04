As far as I can tell, ECH doesn't offer self-service access to proxy logs, so this is a feature request to make to Elastic rather than something you can configure. I couldn't find a documented way to ship them to your monitoring deployment.

## Why Elastic keeps them internal
The proxy tier is shared across many customers' deployments. Raw proxy logs therefore contain other tenants' traffic, plus internal routing details. The usual fix is to expose only a per-deployment slice, not the raw logs.

What I can confirm from the docs: traffic filters operate on the proxy, and in the ECE docs IP filter rejections are logged in proxy logs as status_reason: BLOCKED_BY_IP_FILTER, with the client IP in the client_ip field. That is the exact data you needed. I only found this in the ECE and traffic-filtering pages. I haven't confirmed that ECH has the same schema or lets customers see it.

## What to ask Elastic
Put these to your account team or Customer Success Manager, not only to a support case:

1. **Per-deployment proxy log shipping.** Ask whether proxy logs scoped to your deployment IDs can be sent to your monitoring deployment, or to a data stream you own. Ask for a field allowlist (`client_ip`, `status_reason`, status code, timestamp, deployment ID, request path or method). That shows you what you need without exposing other tenants or the internals of the proxy.
2. **Whether a feature request or roadmap item already exists.** Ask if one covers "customer-visible traffic filter rejection logs" and ask them to attach your account to it. Your SEC, OCC and FDIC-type compliance requirements are a strong justification.
3. **A scheduled support export as a fallback.** If a native feed isn't available, ask for a recurring, redacted extract of BLOCKED_BY_IP_FILTER events for your deployments. Make sure it is included in your support entitlement, not handled as a one-off case each time.
4. **Whether you can enable something yourself.** Ask directly whether there is a flag or setting, or an Elastic-run service, that surfaces filter-rejection events, such as an audit-style feed.
5. **A documented SLA for these requests.** If they can only be fulfilled through support, ask for a response time and a case template so each lookup is fast.

## What you can do now
- **Ship Elasticsearch and Kibana logs to a monitoring deployment.** This covers the deployment logs and metrics, but it won't include proxy-level rejections.
- **Use Elasticsearch audit logging for authenticated failures.** Requests blocked at the proxy never reach Elasticsearch, so audit logs won't show them. They still help with auth failures that do get through.
- **Watch the Elastic Cloud console and API audit logs.** They show who changed traffic filter rule sets and when, which helps when an outage follows a rule change.
- **Check on the client side.** A 403 with the "Forbidden due to traffic filtering" message tells you quickly that the proxy blocked the request. Logging that response in your own clients, load balancers or egress firewall gives you the source IP without needing Elastic.
- **Keep a known egress IP inventory.** Compare it against your rule sets proactively, so most blocks are caught before they happen.

Sources: [Traffic filtering | Elastic docs](https://www.elastic.co/docs/deploy-manage/security/traffic-filtering), [IP filtering rules in ECE](https://www.elastic.co/docs/deploy-manage/security/ece-filter-rules)
