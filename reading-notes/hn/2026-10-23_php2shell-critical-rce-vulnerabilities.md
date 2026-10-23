1. Title

CVE-2026-109618 & CVE-2026-109619: PHP2Shell Critical Vulnerability – What It Is and What to Do

2. Source

Author / Organization: Anis Bouchekhima and Jasper Juergensen / Dynatrace (Dynatrace Engineering Blog)
Link: https://www.dynatrace.com/news/blog/cve-2026-109618-cve-2026-109619-php2shell-critical-vulnerability-what-it-is-and-what-to-do/
Date: 2026-10-22

3. One-line Summary

Emergency security updates were released for two CVSS 9.2 unauthenticated remote code execution vulnerabilities in the core PHP runtime (dubbed "PHP2Shell"), where request-parsing memory allocation failures and uncleaned worker state allow attackers to corrupt internal Zend engine structures before application code or authentication ever executes.

4. Key Points

• Ubiquitous Core Runtime Impact: Affects the core Zend Engine rather than third-party plugins, spanning all standard server deployment topologies—PHP-FPM, Apache mod_php, php-cgi, FrankenPHP, and LiteSpeed—with confirmed working exploits dating back to PHP 5.4.
• CVE-2026-109618 (RCE via OOM Parsing Corruption): An unauthenticated attacker sends a crafted multipart/form-data POST request that triggers an abrupt per-request Out-of-Memory (OOM) error; PHP's non-local `longjmp()` unwinding bypasses standard cleanup, leaving partially initialized structures that PHP misinterprets during error handling to execute arbitrary commands or leak raw memory (API keys, session tokens).
• CVE-2026-109619 (RCE via Stale Worker State): An attacker sends a crafted fatal request leaving a dangling, freed memory pointer in persistent worker process memory; a subsequent request to that same worker reclaims and shapes the heap to hijack control flow.
• Default Exposure via Upload Progress: The stale-state vulnerability is weaponized out of the box because `session.upload_progress.enabled` is active by default in PHP, allowing exploitation even on endpoints that never invoke `session_start()`.
• Embargo Leak and Active In-the-Wild Exploitation: Technical exploit details leaked across social media and public repositories ahead of coordinated disclosure, prompting active exploitation in the wild (with attackers modifying payloads to bypass WAF signatures) and leading European government bodies (such as the Dutch Parliament) to preemptively take public portals offline.
• The Unsupported Version Crisis: While official emergency patch releases exist for PHP 8.2 through 8.5 (versions 8.2.35, 8.3.36, 8.4.27, and 8.5.12), unsupported older versions (PHP 8.1, 8.0, and 7.4) have no binary updates, forcing enterprise teams to manually apply source patches or rely on distribution package backports.
• Defense-in-Depth Configuration Gaps: The PHP Foundation explicitly warns that configuration tweaks cannot fully eliminate vulnerability; however, temporary mitigations include disabling `session.upload_progress.enabled=0`, setting `pm.max_requests=1` to respawn PHP-FPM workers after every single request, enforcing strict request body limits, and dropping multipart traffic at reverse proxies.
• Detection via Runtime Observability: Requires deep process-level inspection (such as Dynatrace DQL) to audit running PHP binaries against vendor backport databases, alongside real-time monitoring of segmentation faults (signal 11), unexpected OOM spikes, and suspicious post-exploitation shell spawning (`whoami`, `uname`, `id`) from web server user accounts.

5. Deep Dive (Structured Understanding)

Problem
Web runtimes written in C must balance high-performance memory management with absolute boundary safety. When PHP receives an HTTP request, its internal Zend engine parses incoming headers and multipart form data before user-land scripts or application-level authentication filters run. In handling allocation failures, PHP relies on C's `setjmp`/`longjmp` mechanism to abruptly unwind execution back to top-level request boundaries. If a memory exhaustion (OOM) occurs mid-parse, this non-local jump bypasses intermediate destructors and object initialization routines. Furthermore, in high-performance application servers like PHP-FPM, worker processes persist across thousands of requests to avoid process-creation overhead. When a fatal error leaves internal global pointers dangling, the worker does not clean its heap completely before accepting the next connection, allowing state corruption to bleed across isolated HTTP requests.

Approach
The security research community (spanning Calif.io, Wiz, OpenAI, and Quarkslab) identified dual exploitation paths in the Zend engine. For CVE-2026-109618, attackers engineer multipart POST payloads with calibrated nesting and field lengths to exhaust `memory_limit` during early parsing, turning the subsequent cleanup path into an execution primitive. For CVE-2026-109619, attackers weaponize `session.upload_progress` to plant dangling pointers in persistent worker memory, followed by heap feng-shui requests that groom the freed memory block into an executable callback. The PHP core team responded with an emergency +11,500-line patch across all active branches, rewriting parser error recovery routines and sanitizing worker lifecycle tear-downs.

Key Insight
1. The Fragility of Non-Local Exception Handling (`longjmp`): Standard structured error handling (such as RAII or Rust error propagation) guarantees deterministic cleanup during stack unwinding. Using C `longjmp()` for runtime memory errors inherently risks leaving partially initialized internal structures in inconsistent states, turning standard memory limits into weaponized attack surfaces.
2. The Persistent Worker Re-entrancy Hazard: High-throughput execution pools (PHP-FPM, FastCGI) violate process isolation by sharing physical memory across sequential user requests. If an uncaught fatal condition fails to trigger immediate process termination, lingering heap artifacts transform request-scoped memory errors into cross-request remote code execution.
3. Inadequacy of Perimeter WAF Defense: Edge firewalls inspecting HTTP payloads can easily be bypassed by manipulating multipart boundary encodings, chunked transfer encoding, and nested MIME wrappers. When vulnerabilities exist in the core parsing engine itself, network-level filtering cannot reliably substitute for patching the runtime binary.

Result / Impact
Mitigated an industry-wide remote code execution crisis across millions of internet-facing servers, forcing emergency patching across global cloud runtimes and underscoring the severe operational fragility of unmanaged, legacy PHP deployments.

6. Why It Matters

• Security & DevSecOps / Threat Intelligence: Represents one of the most severe web infrastructure vulnerabilities of the decade, bypassing all application-layer authentication and impacting millions of production servers running default configurations.
• Systems Programming & Memory Safety: Demonstrates why memory safety initiatives (such as CISA and EU cyber resilience mandates) increasingly target foundational C/C++ runtimes where error-handling edge cases in parsing engines yield catastrophic arbitrary execution.
• Platform Engineering & Web Server Architecture: Highlights the structural trade-off between process isolation and raw execution performance; running persistent worker pools (PHP-FPM, FrankenPHP) without request-boundary process recycling creates persistent state pollution vectors.
• IT Risk / Governance & Vulnerability Disclosure: Serves as a textbook study in coordinated vulnerability disclosure (CVD) failure: premature leaks broke embargo windows, forcing maintainers and cloud providers to scramble emergency defenses while adversaries actively weaponized public Proof-of-Concepts.

7. Critical Analysis

• Patch Availability Asymmetry and Enterprise Debt: Because the PHP Foundation ceased supporting PHP 8.1 and earlier, millions of legacy enterprise and public-sector workloads have no official binary patch path. Expecting small enterprise IT teams to manually backport and compile C source patches guarantees that a vast population of internet-facing servers will remain permanently vulnerable to automated botnet scanning.
• Operational Penalties of Workaround Mitigations: Recommended stopgap mitigations—such as setting `pm.max_requests=1` (destroying worker processes after each request) or blocking multipart requests—introduce severe performance degradation, eliminating the concurrency benefits of PHP-FPM and breaking file upload capabilities in web applications.
• Post-Exploitation Memory Leak Exposure: Beyond remote code execution, CVE-2026-109618 allows an attacker to leak process memory without spawning child shells. Because web worker memory frequently holds database passwords, decrypted TLS session keys, and active user session tokens, stealthy data exfiltration can occur without leaving common indicators of compromise (like `whoami` process launches).

8. Connections

• React2Shell (CVE-2025-55182) & Text4Shell (CVE-2022-42889): Follows the historical lineage of foundational runtime parsing vulnerabilities that execute arbitrary commands before application business logic is reached.
• C `setjmp`/`longjmp` Unwinding Hazards: Directly connects to classic systems programming vulnerabilities where non-local jumps bypass cleanup routines, similar to signal-handler race conditions and C++ exception leaks in mixed-language codebases.
• PHP-FPM Process Pooling Architecture: Mirrors FastCGI and multi-threaded worker architectures (e.g., Apache Prefork, Node.js cluster mode) where process lifetime management directly impacts vulnerability blast radiuses.

9. Keywords

• PHP2Shell
• CVE-2026-109618
• CVE-2026-109619
• Zend Engine
• PHP-FPM
• Unauthenticated RCE
• Out-of-Memory Corruption
• Coordinated Vulnerability Disclosure

10. TL;DR

• Emergency patches were released for PHP2Shell (CVE-2026-109618 & CVE-2026-109619), two critical CVSS 9.2 unauthenticated RCE flaws in the core PHP runtime under active attack.
• Exploitation abuses `longjmp()` stack unwinding during multipart OOM errors and lingering state in persistent PHP-FPM workers, executing code before application auth runs.
• Full mitigation requires updating PHP runtimes (8.2.35+, 8.3.36+, 8.4.27+, 8.5.12+); interim defenses include disabling upload progress and recycling worker processes after every request.
