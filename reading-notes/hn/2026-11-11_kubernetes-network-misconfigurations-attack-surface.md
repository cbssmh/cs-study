1. Title

Kubernetes Security Essentials: Network Misconfigurations – The Invisible Attack Surface

2. Source

Author / Organization: Farooq Shaikh / Dynatrace (Dynatrace Engineering & Security Blog)
Link: https://www.dynatrace.com/news/blog/kubernetes-security-essentials-network-misconfigurations-the-invisible-attack-surface/
Date: 2026-11-05

3. One-line Summary

Dynatrace demonstrates how Kubernetes' default flat networking allows a single compromised frontend pod to enumerate internal services via CoreDNS without triggering audit logs and execute lateral confused-deputy attacks, and how Cilium-enforced NetworkPolicies remediate the blast radius.

4. Key Points

• The Inadequacy of Container Hardening Alone: While container hardening (dropping Linux capabilities, running non-root, read-only root filesystems) prevents host escalation, it does not alter cluster reachability; a completely unprivileged pod on a flat network can still route packets to every internal database and cache in the cluster.
• Stealth Reconnaissance via CoreDNS: Because CoreDNS requires no credentials and uses predictable naming schemes (`<service>.<namespace>.svc.cluster.local`), a compromised pod can enumerate all 16 microservices and internal IP addresses without leaving an audit log entry (as Kubernetes audit logs do not capture DNS lookups by default).
• Direct Datastore Exposure: Demonstrates that without NetworkPolicy definitions, a public-facing web frontend pod can directly establish TCP connections to MariaDB on port 3306 and Redis on port 6379, despite having zero legitimate operational need to query them.
• Confused Deputy Lateral Exploitation: Lacking database credentials, an attacker weaponizes an internal `proxy-service` via Server-Side Request Forgery (SSRF) to target an unexposed `user-auth-service`, exploiting an unescaped SQL query to generate a validly signed RS256 JWT for `userid: 1` with `AD_MANAGER` privileges.
• The CNI Policy Engine Requirement: NetworkPolicy objects in Kubernetes are inert declarative manifests unless enforced by a network plugin with an active policy engine (e.g., Cilium via eBPF); on clusters with basic overlay plugins, policies can exist while traffic remains completely unrestricted.
• Automated Policy Generation via Kimera & MCP: Deploys Dynatrace’s open-source Kimera security toolkit to synthesize 19 namespace-scoped NetworkPolicy resources, ingesting live runtime topology edges via the Model Context Protocol (MCP) and classifying paths into KEEP (declared), DENY (undeclared), and REVIEW.
• Granular Egress Policy with RFC 1918 Exclusions: Configures egress rules on the proxy service allowing internet connectivity (`0.0.0.0/0`) while explicitly denying private IPv4 ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) and cloud instance metadata (`169.254.169.254/32`), preventing internal SSRF pivoting to pod IPs.
• APM vs Network Reachability Observability Blind Spots: Application Performance Monitoring (APM / Smartscape) traces traffic carried by instrumented runtimes, but direct shell-level network probes spawned by attackers do not emit distributed tracing spans, necessitating direct synthetic reachability probes for true policy verification.
• Quantified Attack Surface Reduction: Applying the policy set reduced the number of workloads permitted to reach the authentication service from 15 down to 4 declared callers, blocked direct MariaDB access, and satisfied DORA-67974 compliance audits while keeping legitimate external proxying functional.

5. Deep Dive (Structured Understanding)

Problem
The foundational Kubernetes networking model mandates that every pod gets an IP and that all pods can communicate with all other pods across the cluster without NAT. In default namespace deployments without NetworkPolicy objects (which violates CIS Benchmark 5.3.2 and DORA-67974), the cluster functions as a wide-open flat overlay network. When an internet-facing pod is compromised, container hardening controls (e.g., non-root users, seccomp profiles) fail to stop lateral movement. Furthermore, because internal DNS queries to CoreDNS do not produce Kubernetes API audit logs, attackers can map the entire internal microservice topology silently. Attackers can then leverage internal architectural patterns (such as microservice proxies or utility endpoints) as "confused deputies" to relay malicious payloads to backend services that are completely un-routable from the public internet.

Approach
Dynatrace security researchers modeled an end-to-end compromise scenario using Unguard (a vulnerable microservices application) and Kimera (an open-source Kubernetes security testing framework). From a compromised frontend pod, Kimera executed automated CoreDNS enumeration, discovering 16 services, MariaDB (3306), Redis (6379), and the Kubernetes API server (443). To bypass absent database credentials, Kimera executed an application-layer L7 SSRF attack through `proxy-service`, relaying a UNION-based SQL injection payload into `user-auth-service` to mint an administrative JWT. To close the attack surface, Kimera queried Dynatrace KSPM findings and Smartscape topology edges over the Model Context Protocol (MCP), generated 19 strict NetworkPolicies, and enforced them at the Linux kernel layer via Cilium's eBPF datapath.

Key Insight
1. Container Isolation Does Not Equal Network Isolation: Platform teams often over-invest in container image hardening while neglecting network micro-segmentation. A hardened container stripped of all Linux capabilities and operating as an unprivileged user retains raw TCP/UDP socket creation capabilities, allowing it to traverse the entire flat cluster network unimpeded.
2. The Stealth of Unlogged CoreDNS Service Discovery: Security Operations Centers (SOCs) rely heavily on Kubernetes API server audit logs to detect anomalous activity. Because CoreDNS resolves service names internally without authenticating requests or logging to the API audit stream, internal reconnaissance is completely invisible to standard cloud SIEM pipelines.
3. Network Policies Block Exploitation Chains, Not Code Flaws: Applying network segmentation did not patch the underlying SQL injection vulnerability in the authentication service. Instead, it decoupled the attack chain by eliminating unauthorized network paths, reducing reachable callers from 15 to 4. Defense-in-depth requires network isolation to neutralize zero-day vulnerabilities in un-patchable backend code.

Result / Impact
Demonstrated the step-by-step containment of a multi-stage confused deputy attack in Kubernetes, proving that pairing runtime topology data with Cilium-enforced NetworkPolicies systematically neutralizes lateral movement without disrupting legitimate microservice communications.

6. Why It Matters

• Platform Engineering & Kubernetes Architecture: Demonstrates why default-allow networking is the single most dangerous architectural default in Kubernetes, establishing CNI-level policy enforcement (Cilium/Calico) as an indispensable platform baseline.
• Security & DevSecOps / Cloud-Native Security: Bridges the gap between static compliance audits (DORA-67974, CIS 5.3.2) and dynamic attack simulation, proving how tools like Kimera can validate security controls via executable techniques.
• Distributed Systems & Microservice Design: Highlights the systemic risks of the "confused deputy" pattern in service meshes and internal API proxies, mandating strict RFC 1918 and cloud metadata (`169.254.169.254`) egress filtering.
• Reliability & Observability: Exposes the critical distinction between APM instrumentation (which traces application requests) and low-level network packet reality, showing that runtime verification requires both telemetry layers.

7. Critical Analysis

• Reliance on Under-the-Hood CNI Enforcement: The authors rightly note that NetworkPolicy is a Kubernetes API abstraction that relies entirely on CNI plugin enforcement. In unmanaged or legacy clusters running standard cloud-provider basic overlays (e.g., standard AWS VPC CNI without policy controllers or basic Flannel), applying 19 NetworkPolicy manifests will pass compliance scans (reporting PASSED on DORA-67974) while providing zero actual network packet filtering, creating a dangerous false sense of security.
• Policy Generation Scalability in Dynamic Topologies: While automated LLM-assisted policy drafting via Kimera works well for a static 16-service demo, generating and maintaining hundreds of NetworkPolicy manifests across high-velocity microservice clusters with frequent ephemeral deployments introduces severe GitOps configuration sprawl and policy-drift overhead.
• The DNS Egress Hole: The generated policy set explicitly permits UDP/TCP 53 to CoreDNS (`kube-dns`). While necessary for application functionality, unconstrained DNS access leaves a covert channel open for DNS tunneling and data exfiltration, which L3/L4 NetworkPolicies cannot inspect without deep L7 DNS filtering (e.g., Cilium DNS visibility policies).

8. Connections

• Cilium eBPF Network Enforcement: Directly utilizes Cilium’s kernel-space eBPF packet-filtering engine (`bpf_lxc`) to enforce L3/L4/L7 ingress and egress rules without the connection-tracking overhead of legacy iptables.
• The Confused Deputy Problem (Hardy, 1988): A classic computer security concept where an unprivileged entity tricks a privileged intermediary into misusing its authority, instantiated here through microservice SSRF.
• AWS/GCP Instance Metadata Service (IMDSv2): Parallels cloud infrastructure hardening by explicitly blocking `169.254.169.254/32` in egress policies to prevent SSRF credential theft from the underlying node.

9. Keywords

• Kubernetes Security
• NetworkPolicy
• Flat Network Architecture
• Cilium eBPF
• Confused Deputy
• CoreDNS Enumeration
• Server-Side Request Forgery (SSRF)
• Micro-segmentation

10. TL;DR

• Dynatrace demonstrated how Kubernetes' default flat networking allows a compromised pod to silently discover all internal services via CoreDNS and execute lateral confused-deputy attacks.
• An unprivileged frontend pod used an internal proxy to relay SQL injection into an unexposed authentication backend, minting an administrative JWT without database credentials.
• Enforcing 19 CNI-backed NetworkPolicies via Cilium and Kimera eliminated unauthorized paths, reducing exposed authentication callers from 15 to 4 and closing private IP egress routes.
