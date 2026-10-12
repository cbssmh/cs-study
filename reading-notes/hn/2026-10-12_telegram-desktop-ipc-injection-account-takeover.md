1. Title

Telegram Desktop: One-Click Account Takeover via IPC Injection

2. Source

Author / Organization: BeakSec
Link: https://beaksec.github.io/posts/telegram-desktop-one-click-account-takeover/
Date: 2026-10-03 (Updated 2026-10-07; CVE-2026-107181 assigned 2026-10-07)

3. One-line Summary

A delimiter injection vulnerability in Telegram Desktop’s single-instance local IPC socket combined with an internal release-publishing URI scheme (`interpret:`) and auto-downloaded chat attachments allowed an attacker to achieve arbitrary local file exfiltration and full account takeover via a single clicked link.

4. Key Points

• Single-Instance IPC Serialization Defect: When launched with a custom URI, Telegram Desktop connects to an already-running instance over a local socket, serializing URLs as semicolon-delimited commands (`OPEN:<url>;`) without escaping semicolons present in incoming query strings.
• IPC Command Injection: Appending an unescaped semicolon in a clicked `tg://` link enables an attacker to inject arbitrary IPC instructions, splitting a single URL into multiple distinct commands executed by the primary running process.
• Abuse of Unregistered `interpret:` Scheme: The injected command invokes Telegram’s internal `interpret:` URI scheme—a legacy tool originally designed for release builds that reads an instruction file from disk and uploads specified files to a Telegram chat without user confirmation or authentication checks.
• Deterministic File Placement via Default Auto-Download: Telegram Desktop’s default settings automatically download group chat attachments up to 8 MiB to a predictable path (`%APPDATA%\Telegram Desktop\..\..\..\Downloads\Telegram Desktop\`), allowing an attacker to place malicious instruction files on the victim's disk silently.
• Path Traversal for User-Agnostic Execution: The `interpret:` handler resolves relative paths against Telegram's working directory (`%APPDATA%\Telegram Desktop`), allowing attackers to traverse directories (`../../../Downloads/...`) and locate payload files without needing to know the victim's Windows username.
• Cryptographic Key Unwrapping to Session Stealing: By chaining stacked `OPEN:` commands, an attacker exfiltrates the victim's `key_datas`, `D877F783D5D3EF8Cs` (encrypted MTProto authorization), and map index; because Telegram sets no local passcode by default, the Key Encryption Key (KEK) is derived from an empty password and cleartext salt, enabling immediate decryption of the Data Encryption Key (DEK) and complete account takeover.
• Weaponized Delivery via External Redirection: Because in-app `tg://` links are handled in-process, the attacker delivers an ordinary HTTPS link in chat that redirects via HTTP 302 to the crafted `tg://` payload, forcing the OS to spawn a second instance and trigger the vulnerable local socket IPC handshake.
• Patch and Security Hardening: Telegram Desktop 7.2.9 addressed CVE-2026-107181 (CVSS 8.1) by removing the `interpret:` helper, percent-encoding command separators over the IPC socket, dropping local file paths when non-local URLs are present, and disallowing `CMD:`/`CTRL:` records on connections containing `OPEN:`.

5. Deep Dive (Structured Understanding)

Problem
Desktop operating systems register custom protocol handlers (such as `tg://`) to launch registered applications when clicked. When an application enforces a single-instance model, subsequent process launches must forward invoked URI parameters to the primary running instance via Inter-Process Communication (IPC). If the IPC mechanism flattens structured URL objects into unstructured text streams using simple delimiter splitting without robust escaping, boundary confusion arises. Furthermore, internal debugging or automation protocols embedded in native clients often assume that local execution implies trusted origin, creating severe privilege escalation pathways if reachable through IPC.

Approach
BeakSec analyzed the boundary crossing between the secondary launched process and the primary listening process in Telegram Desktop. The secondary process serialized incoming URLs by string concatenation (`"OPEN:" + url + ";"`) and transmitted them over a local socket. The receiving server process split incoming data at every semicolon. The researcher crafted an injection payload that embedded semicolons into the query parameters of a `tg://` link, appending multiple `OPEN:interpret:<path>` directives. To provide the required instruction files, the researcher exploited Telegram’s default behavior of automatically downloading files under 8 MiB in group chats, using directory traversal relative to `%APPDATA%` to construct deterministic paths.

Key Insight
1. The "Weird Machine" of IPC Serialization: Semicolon splitting over raw byte streams demonstrates the classic vulnerability pattern where structured data loses type boundaries across processes; unescaped input effectively becomes executable bytecode for the receiving process.
2. The Danger of Dormant Internal Schemes: The `interpret:` protocol scheme was built as a maintenance convenience to automate internal software releases by reading instructions and sending files to channels. Exposing an administrative action that reads arbitrary local files and uploads them to a network endpoint without confirmation turns any URI injection into full-blown arbitrary file exfiltration.
3. Key Wrapping Degradation with Default Empty Credentials: Two-tier encryption (DEK protected by KEK) provides zero cryptographic protection at rest when the KDF password is default-empty. Because `key_datas` stores the salt in cleartext and the passcode is an empty string, the DEK can be instantly recovered once the salt file is exfiltrated, rendering session storage encryption cosmetic.

Result / Impact
Demonstrated a full zero-interaction (post-click) account takeover and arbitrary file exfiltration exploit on default installations of Telegram Desktop across Windows. Prompted the silent removal of the `interpret:` feature and rigorous escaping of IPC record separators in version 7.2.9 (CVE-2026-107181).

6. Why It Matters

• Security / Application Security: Highlights the severe risks of single-instance IPC implementations and custom URI protocol schemes in desktop client software, an often-overlooked attack surface compared to web applications.
• DevSecOps & Secure Architecture: Demonstrates why internal administrative and release automation scripts must never be compiled into production consumer binaries, enforcing strict separation of build/maintenance tooling from user-facing runtimes.
• Reliability / Platform Engineering: Emphasizes that desktop client sandboxing (e.g., AppContainer, Flatpak, bubblewrap) and least-privilege file access are critical safeguards to prevent messaging applications from having unrestricted read access to user files.
• IT Risk / Governance: Underscores the danger of insecure default configurations (automatic file downloads and absent local passcodes) in widespread enterprise communication tools, where a single compromised link can compromise sensitive enterprise credentials and SSH keys.

7. Critical Analysis

• Silent Patching and Transparency Deficit: Telegram patched this high-severity vulnerability (commit `db3405699f`) under the vague commit message "Remove legacy interpret path helper" and attributed 7.2.9 solely to a "rendering fix," omitting any public security advisory. This lack of transparency denies enterprise administrators the awareness needed to audit historical exposure or enforce immediate client updates.
• OS-Level Sandboxing Blind Spot: As reflected in community discussions, the exploit succeeds because traditional desktop operating systems (Windows and Linux) grant native applications unrestricted read access to the user's entire home directory. Mobile operating systems (iOS/Android) prevent this class of attack by strictly isolating application storage within isolated sandbox containers.
• Browser Protocol Prompt Friction: In practice, web browsers often display a confirmation prompt (e.g., "Open Telegram Desktop?") before executing a custom protocol handler redirect. While users frequently click through these prompts routinely, it does represent a slight operational hurdle that prevents pure, invisible zero-click execution from a web visit.

8. Connections

• Delimiter Injection (CWE-77 / CWE-88): Shares foundational mechanics with classic SQL injection, HTTP header injection (CRLF), and shell command injection, where in-band control characters subvert structural parsing.
• Desktop IPC Attack Surfaces (Slack, Discord, Zoom): Parallels historical RCE and file-stealing vulnerabilities in Electron and native desktop apps exploiting custom protocol handlers (e.g., `discord://`, `zoommtg://`) and local WebSocket/named pipe IPC.
• "Weird Machines" & LangSec (Language-Theoretic Security): Exemplifies the LangSec principle that ad-hoc, unformalized parsers operating on unescaped input streams transform message parsers into vulnerable virtual machines.

9. Keywords

• Telegram Desktop
• IPC Injection
• CVE-2026-107181
• Custom Protocol Handler
• Account Takeover
• Arbitrary File Read
• Key Encryption Key (KEK)
• Serialization Security

10. TL;DR

• A delimiter injection flaw in Telegram Desktop’s local IPC socket allowed a single clicked link to split into multiple commands and invoke the hidden internal `interpret:` scheme.
• By chaining automatically downloaded group attachments and path traversal, attackers could read arbitrary local files and exfiltrate them to an external chat without user prompts.
• Stealing cleartext salt files and session data enabled full decryption of MTProto authorization keys, leading to complete account takeover on default installations prior to version 7.2.9.
