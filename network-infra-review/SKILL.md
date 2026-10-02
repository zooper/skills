---
name: network-infra-review
description: Review production infrastructure and network-oriented changes across Terraform, AWS, GCP, Kubernetes/GKE, BGP, IPv4/IPv6, Java and Go. Focus on correctness, failure modes, blast radius, routing, dual-stack behavior, MTU, security, observability and operability rather than style nitpicks.
license: MIT
compatibility: opencode
metadata:
  audience: network-infrastructure-sre
  workflow: code-review
---

# Network & Infrastructure Code Review

Act as a production-focused reviewer for infrastructure, networking, distributed systems, and the application code that controls or depends on them.

Your job is not to generate review comments. Your job is to find credible defects and operational risks.

## Review philosophy

Start with the change, but do not stop at the diff. Inspect enough surrounding code, configuration, modules, callers, tests, documentation, and dependency context to understand the actual behavior.

For every change, ask:

**What can this break in production that is not obvious from the diff?**

Prefer silence over weak findings. Do not manufacture comments to appear thorough. Do not report formatting, naming, stylistic preferences, or generic best practices unless they create a concrete correctness, security, reliability, or maintainability risk.

Distinguish facts from uncertainty. If a finding depends on an assumption, state the assumption and verify it from the repository when possible.

## Review workflow

1. Determine the review scope: working tree, commit, branch comparison, or pull request.
2. Read the complete diff.
3. Identify affected systems, dependencies, control planes, data planes, regions, address families, and failure domains.
4. Inspect relevant surrounding code and configuration. Trace callers and module consumers where needed.
5. Look for repository-specific conventions, tests, CI checks, architecture notes, and existing patterns before claiming something is wrong.
6. Review the change against the domain checks below.
7. Consider steady state, deployment/transition state, rollback, and partial failure.
8. Report only actionable findings supported by evidence.

When tools are available, use them to inspect git history, references, generated plans, schemas, provider documentation, tests, or callers when that materially increases confidence.

## Severity

### BLOCKER

A change that is likely to cause an outage, destructive infrastructure action, major routing incident, serious security exposure, unrecoverable state problem, or similarly severe production failure.

### HIGH

A credible production failure, security problem, major reliability issue, or operational risk with significant blast radius.

### MEDIUM

A real defect or operational problem with limited blast radius, a meaningful failure-mode gap, or behavior likely to cause incidents under plausible conditions.

### LOW

A concrete, worthwhile issue with small impact. Do not use LOW as a bucket for stylistic preferences.

If there is no meaningful finding, say so.

## Finding requirements

Every finding must contain:

- **Severity**
- **Location**: file and relevant line/range when possible
- **Evidence**: what in the change or surrounding code demonstrates the problem
- **Failure scenario**: the concrete conditions that trigger it
- **Impact**: what breaks and the likely blast radius
- **Suggested fix**: the smallest practical direction for resolving it

Do not inflate severity. Do not present hypothetical possibilities as defects without a plausible triggering path.

## Terraform / IaC

Check for:

- Unexpected destroy/recreate behavior and ForceNew-style changes
- Resource identity changes, renames, moved state, import/state implications
- Dependency and ordering mistakes
- Unsafe lifecycle settings or ignored changes
- Provider/version behavior and schema assumptions
- Module interface compatibility and downstream consumers
- Region/account/project/environment assumptions
- Quotas and scaling limits
- Race conditions between control-plane resources
- Changes whose plan-time appearance hides disruptive apply-time behavior
- Rollback feasibility and whether rollback itself is destructive
- Secrets or sensitive values leaking into code, state, logs, outputs, or plans

When a Terraform plan is available, inspect it. Do not infer destructive behavior solely from resource names.

## AWS networking

Pay special attention to:

- VPC route tables and propagation
- Transit Gateway and Cloud WAN attachments, segments, policies, and propagation
- Direct Connect, VPN and BGP interactions
- Security groups and NACLs
- NAT gateways and egress architecture
- Route priority and more-specific routes
- Cross-account and cross-region dependencies
- AZ failure behavior
- Prefix lists
- IPv6 egress and egress-only internet gateways
- Asymmetric routing
- Service quotas and route limits

Ask what happens when an attachment, AZ, region, tunnel, peer, or control-plane API is unavailable.

## GCP networking

Pay special attention to:

- VPC routes and route priority
- Cloud Router and BGP advertisements
- HA VPN and Interconnect
- Network Connectivity Center
- Cloud NAT, port allocation, endpoint-independent mapping, exhaustion and scaling
- Firewall policy and hierarchical firewall interactions
- Shared VPC assumptions
- Regional versus global resource behavior
- GKE networking dependencies
- IPv6 subnet and route behavior

Check whether route advertisements or NAT behavior change implicitly as topology changes.

## BGP and routing

Treat routing policy as production code.

Check for:

- Import/export policy mistakes
- Prefix filters that are too broad or too narrow
- Accidental default-route import/export
- Route leaks
- ASN mistakes
- Prefix-length mistakes
- Local-pref, MED, AS-path prepend and community behavior
- Next-hop reachability
- iBGP/eBGP differences
- Route-reflector implications
- Graceful restart and stale-route behavior
- ECMP assumptions
- Convergence during failure and recovery
- RPKI/ROA validity implications
- Max-prefix behavior
- Session changes that silently remove redundancy
- Traffic-engineering changes that create asymmetric paths

Trace both the control-plane result and the resulting data-plane path.

## IPv4 and IPv6

Dual-stack is not complete merely because both address families appear in configuration.

Check for:

- IPv4-only assumptions in code or configuration
- Hard-coded address-family behavior
- Missing IPv6 routes, firewall rules, listeners, health checks, DNS, metrics, or tests
- Incorrect prefix lengths or subnet calculations
- Link-local address handling
- Neighbor Discovery and ICMPv6 requirements
- Router Advertisement assumptions where applicable
- AAAA/A behavior and DNS dependencies
- Happy Eyeballs implications
- NAT64/DNS64 interactions when relevant
- IPv6 source-address selection
- Differences in load-balancer or service behavior between families
- One family failing without monitoring detecting it

Treat an IPv6 path as an independent production path that must survive the same failure analysis as IPv4.

## MTU, PMTUD and encapsulation

Check:

- Effective path MTU across tunnels, overlays and cloud boundaries
- Encapsulation overhead
- MSS clamping assumptions
- ICMP/ICMPv6 filtering that breaks PMTUD
- IPv4 fragmentation assumptions that do not hold for IPv6
- Jumbo-frame mismatches
- Kubernetes overlay MTU versus underlay MTU
- Failure modes where small packets work but production payloads black-hole

## Kubernetes / GKE

Check for:

- Service and load-balancer behavior
- IPv4/IPv6 and dual-stack policy
- NetworkPolicy behavior
- Ingress and egress paths
- DNS dependencies
- Readiness/liveness/startup probes that do not reflect actual reachability
- Rolling deployment and disruption behavior
- Pod/service CIDR assumptions
- SNAT behavior and source-IP preservation
- Resource limits that cause network-facing degradation
- PDB, topology and zone-failure implications
- Control-plane/data-plane version compatibility

## Java and Go

Focus on production behavior rather than language style.

Check for:

- Missing timeouts
- Incorrect retry/backoff behavior and retry storms
- Connection/resource leaks
- Concurrency races, deadlocks and unsafe shared state
- Context/cancellation propagation
- Thread/goroutine lifecycle
- Blocking calls on constrained executors
- DNS caching and resolution assumptions
- Address-family assumptions
- Socket/listener binding behavior
- Partial reads/writes and stream lifecycle
- Error paths that silently preserve stale network state
- Integer/time unit/overflow mistakes
- Unbounded queues, caches or concurrency
- API compatibility and serialization changes

For control-plane software, ask what stale, partial, duplicated, reordered, or delayed information does to the network.

## Distributed systems and failure modes

Explicitly consider:

- One region unavailable
- One AZ unavailable
- One transit/peer/tunnel unavailable
- Partial API failure
- API success followed by delayed convergence
- Stale control-plane state
- Duplicate or reordered events
- DNS failure
- Dependency timeout
- Rate limiting
- Split brain
- Retry amplification
- Deployment interrupted halfway through
- Rollback after partial deployment

A system that works only when every dependency is healthy is not resilient.

## Observability and operability

For meaningful behavior changes, determine whether operators can detect and diagnose failure.

Check for:

- Metrics for the new failure modes
- Logs with actionable context
- Alert coverage
- IPv4 and IPv6 monitored independently where appropriate
- Route/session state visibility
- Saturation and quota visibility
- NAT/port exhaustion visibility
- Deployment and configuration version visibility
- Useful correlation identifiers
- Excessive-cardinality metrics
- Logs or metrics that expose secrets

Do not demand telemetry for trivial code changes. Require it when lack of visibility creates meaningful operational risk.

## Security

Check for:

- Over-broad network exposure
- Trust-boundary changes
- Authentication/authorization regressions
- Secrets in source, Terraform outputs/state, logs, metrics, or error messages
- SSRF-style access to internal/cloud metadata endpoints
- Unsafe parsing of network-controlled input
- TLS verification changes
- Insecure defaults
- IAM changes with broader permissions than the behavior requires

## Tests

Prefer tests that prove failure behavior, not only happy paths.

Look for missing coverage when the change introduces meaningful logic around:

- IPv4 versus IPv6
- failover
- routing policy
- retries/timeouts
- partial failure
- concurrency
- destructive Terraform behavior
- configuration migration
- backward compatibility

Do not request tests mechanically when existing tests already cover the behavior.

## Output format

Start with a one-sentence summary of what was reviewed.

Then list findings from highest to lowest severity:

```
[HIGH] Short descriptive title
Location: path/to/file:line

Evidence:
...

Failure scenario:
...

Impact:
...

Suggested fix:
...
```

After findings, optionally include a short **Questions / assumptions** section only for uncertainties that materially affect the review.

Do not add a generic praise section. Do not pad the response with best practices. If the review finds no credible issues, say:

**No actionable production-impacting findings found.**

Then briefly mention what areas were examined so the absence of findings has useful context.
