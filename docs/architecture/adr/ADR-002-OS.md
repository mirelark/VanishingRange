# ADR-002: Operating System Selection

## Status

Accepted

## Context

The system is a single-node, VPS-hosted deployment for a RESTful SPA application with supporting backend services. The operating system must support:

- lightweight deployment and runtime footprint
- stable package management and system updates
- compatibility with standard web service tooling (e.g., reverse proxy, runtime environments)
- low operational overhead for a solo-maintained system
- open-source and cost-free licensing

## Decision

**Selected Operating System: Void Linux**

Void Linux will be used as the base operating system for the initial and current deployment environment.

## Rationale

Void Linux was selected based on the following system-level properties:

- minimal base system footprint
- independent distribution with a straightforward packaging system
- rolling release model with frequent upstream updates
- optional musl or glibc support depending on runtime requirements
- native support for lightweight init system (runit)

## Alternatives Considered

**Ubuntu**: Widely adopted distribution with strong ecosystem support. Rejected due to larger default system footprint and opinionated package management approach (e.g., Snap ecosystem).

**Debian**: Stable and conservative release model. Rejected due to slower update cadence relative to desired system learning and iteration cycle.

**Arch Linux**: Highly flexible rolling release distribution. Rejected due to higher maintenance burden and reliance on user-managed configuration decisions at system level.

**Fedora Server**: Enterprise-oriented distribution with strong defaults. Rejected due to opinionated system configuration and shorter lifecycle per release.

**CentOS Stream**: Upstream of RHEL ecosystem. Rejected due to continuous integration-style update model and enterprise-oriented constraints not aligned with project scope.

**NixOS**: Declarative system configuration model. Rejected for initial implementation due to learning curve and divergence from conventional Linux system management patterns used in target deployment environments.

**Alpine Linux**: Minimal distribution with strong container usage profile. Considered; however, musl-based ecosystem compatibility risks were identified for some workloads.

**BSD-based systems**: Considered for completeness. Rejected due to reduced ecosystem alignment with target tooling and deployment patterns.

**Windows Server**: Not selected due to mismatch with target deployment ecosystem and increased resource overhead relative to requirements.

## Consequences

Positive
- lightweight system baseline
- predictable package management model
- rolling release provides timely updates
- minimal default system services
- lightweight init system (runit)

Negative
- rolling release model introduces update management responsibility
- smaller ecosystem compared to larger mainstream distributions
- increased need for manual validation during upgrades

Risks
- system instability introduced via upstream updates
- operational overhead during maintenance windows

Mitigations
- staged updates prior to production application
- minimal host-level customization to reduce configuration drift

Implementation Notes
- Void Linux installed directly onto VPS instance
- Deployment performed on Hetzner-provisioned virtual machine (see [ADR-001](../ADR-001-VPS)
- No additional OS-level customization beyond baseline package installation at this stage

## References
Analysis artifact: /analysis/ADR-002-A
Void Linux documentation: https://voidlinux.org
