# ADR-001: VPS Hosting Provider Selection

## Status

**Accepted**

## Context

The system requires a publicly accessible hosting environment for deployment of a single-node deployment hosting a multi-stage internal data processing pipeline.

The hosting provider must support:

- sufficient compute resources for application and data processing pipeline
- persistent storage for application data and logs
- stable network performance for public-facing access
- infrastructure-level control for operating system selection and configuration
- compatibility with infrastructure-as-code provisioning

Baseline resource requirements:

- ≥ 8 GB RAM
- ≥ 200 GB storage
- stable outbound bandwidth suitable for web traffic


## Decision

**Selected Provider: Hetzner**

Hetzner will be used as the primary VPS hosting provider for system deployment.

## Rationale

Hetzner was selected based on the following evaluated properties:

- resource allocation meets or exceeds baseline compute and storage requirements
- support for full virtual machine control (including custom OS installation)
- predictable pricing model relative to comparable VPS offerings
- support for standard infrastructure automation workflows
- availability of data center regions suitable for low-to-moderate latency deployment targets

## Alternatives Considered

**AWS / Azure**: Large-scale cloud providers with extensive service ecosystems. Rejected due to higher operational complexity and cost overhead relative to single-node system requirements.

**Contabo**: High-resource VPS provider with competitive pricing. Rejected due to less consistent performance characteristics and variability in service reliability reports.

**Oracle Cloud Free Tier**: Free-tier VPS offering. Rejected due to insufficient guaranteed resources for sustained application workload requirements.

**Smaller VPS providers (various)**: Considered for cost optimization. Rejected due to limited operational history and reduced confidence in long-term reliability and support maturity.

## Consequences

Positive
- sufficient and predictable compute resources for system requirements
- full control over VM configuration and operating system selection
- compatibility with infrastructure-as-code provisioning workflows
- simplified billing model relative to hyperscale cloud platforms

Negative
- increased latency compared to geographically distributed cloud infrastructure
- fewer managed services compared to hyperscale cloud providers
- higher reliance on self-managed reliability and observability

Risks
- single-region deployment introduces limited geographic redundancy
- instance-level failure results in full service disruption without external failover design
- network latency variability depending on user geography

Mitigations
- future consideration of periodic VM snapshotting for recovery scenarios
- future consideration of CDN integration for static assets
- optional multi-region expansion if usage or reliability requirements increase

Implementation Notes
- VPS instance will host the full application stack
- operating system installed separately (see ADR-002-OS)
- infrastructure provisioning may be automated via Terraform-based workflows (future extension)
- system architecture defined in ../architecture/system-architecture.md

References
- Analysis artifact: /analysis/ADR-001-A
- System architecture documentation: ../architecture/system-architecture.md
