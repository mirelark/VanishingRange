# ADR-001-A: VPS Hosting Provider Evaluation Analysis

## Status

Supporting analysis for ADR-001.

Related Decision:

- [ADR-001: VPS Hosting Provider Selection](../ADR-001-VPS.md)

---

# Problem Statement

Selection of a VPS hosting provider for deployment of the application and supporting data-processing pipeline.

The hosting provider must support:

- public-facing application hosting
- persistent storage for source datasets and application data
- sufficient compute resources for application and processing workloads
- operating system configuration and management
- future infrastructure automation workflows

---

# Baseline Requirements

## Resource Requirements

Projected deployment requirements:

- > 250 GB storage for specie's geographic data, image, and plaintext storage
- 16 GB RAM for language-model-related workloads
- high inbound and outboud traffic allowance
- stable network performance

## Operational Constraints

Selection criteria were constrained by:

- cost
- deployment simplicity
- implementation time
- operational flexibility

---

# Evaluation Criteria

The following characteristics were prioritized during provider evaluation:

| Criterion                | Description                                                               |
| ------------------------ | ------------------------------------------------------------------------- |
| Cost Predictability      | Stable monthly pricing without significant usage-based variability        |
| Resource Allocation      | Ability to meet RAM, storage, and bandwidth requirements                  |
| Performance Stability    | Consistent CPU, storage, and network performance characteristics          |
| Infrastructure Control   | Full VM administrative access and custom OS installation support          |
| Operational Simplicity   | Straightforward provisioning and maintenance for a single-node deployment |
| Automation Compatibility | Compatibility with Terraform and infrastructure automation workflows      |
| Provider Reliability     | Confidence in provider operational maturity and uptime consistency        |
| Scalability Flexibility  | Ability to expand resources or deployment architecture later if required  |

---

# Options Evaluated

## Option 1: AWS / Azure

Modern cloud hosting ecosystem providers.

Approximate cost:

- $40+/month

### Advantages

- well-documented ecosystem
- regional redundancy and stronger uptime guarantees
- extensive cloud administration tooling
- managed service ecosystem

### Disadvantages

- higher monthly operational cost
- potential pricing overrun risk
- increased operational complexity relative to project scope

### Notes

- better suited for scalable or distributed infrastructure
- stronger integration options for GPU or managed services
- multi-component management and monitoring tools

### Assessment

Rejected due to operational complexity and pricing overhead relative to single-node deployment requirements.

---

## Option 2: CloudVPS

Smaller VPS-focused hosting provider.

Approximate cost:

- $25/month

### Advantages

- strong cost-per-resource allocation

### Disadvantages

- limited provider history and maturity confidence

### Notes

- no major technical shortcomings identified during evaluation

### Assessment

Met baseline requirements but introduced additional uncertainty regarding operational maturity and long-term reliability.

---

## Option 3: Hetzner Cloud

Dedicated VPS-oriented infrastructure provider.

Approximate cost:

- $35/month

### Advantages

- strong network stability
- strong CPU and storage performance
- infrastructure automation tooling support
- full VM administrative control
- support for custom operating system installation

### Disadvantages

- storage expansion costs comparatively higher
- documentation and support considered more minimalistic

### Notes

- aligns with data and privacy considerations
- suitable for Terraform-oriented workflows

### Assessment

Provided the strongest balance of:

- performance stability,
- operational control,
- infrastructure automation support,
- and predictable pricing.

---

## Option 4: Contabo

High-resource VPS provider focused on low-cost allocations.

Approximate cost:

- $20/month

### Advantages

- high RAM, bandwidth, and storage allocation
- root access
- custom OS installation support

### Disadvantages

- inconsistent performance between regions
- inconsistent performance during peak utilization periods
- basic management interface

### Assessment

Provided strong nominal resource allocation but raised concerns regarding performance consistency and noisy-neighbor effects.

---

## Option 5: Oracle Cloud Free Tier

Free-tier VPS hosting environment.

Approximate cost:

- free

### Advantages

- no infrastructure cost

### Disadvantages

- insufficient guaranteed resources
- unsuitable for sustained public-facing workload requirements

### Notes

- better suited for temporary deployments or experimentation

### Assessment

Rejected due to insufficient resource guarantees for projected workloads.

---

# Trade-off Summary

Primary trade-offs identified during evaluation:

| Provider               | Primary Concern                              |
| ---------------------- | -------------------------------------------- |
| AWS / Azure            | Cost and operational complexity              |
| CloudVPS               | Provider maturity and operational confidence |
| Hetzner                | Storage expansion cost and regional latency  |
| Contabo                | Performance consistency                      |
| Oracle Cloud Free Tier | Insufficient resources                       |

Additional observations:

- lightweight REST architecture reduces sensitivity to moderate latency
- frontend optimization or CDN caching could reduce perceived load latency if required

---

# Decision Drivers

Primary decision drivers:

- capability-to-cost ratio
- infrastructure control
- operational simplicity
- support for automation tooling
- provider reliability reputation

Additional considerations:

- Terraform support
- custom OS installation support
- dedicated-resource stability characteristics
- alignment with privacy and data-protection considerations

---

# Final Recommendation

Selected Provider:

- Hetzner Cloud

Alternate Provider:

- Contabo

---

# References

- [ADR-001: VPS Hosting Provider Selection](../ADR-001-VPS)
