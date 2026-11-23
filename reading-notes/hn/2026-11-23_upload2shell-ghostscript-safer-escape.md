1. Title
How We Hacked the Pipes — Upload2Shell

2. Source
Author / Organization: Paulos Yibelo / pwn.ai
Link: https://pwn.ai/upload2shell/research.html
Date: November 21, 2026
HN discussion: https://news.ycombinator.com/item?id=50592097

3. One-line Summary
A use-after-free in Ghostscript's Type 1 font loader (`CVE-2026-125396`) turns a recovered `.forceput` store into a native write-what-where primitive that disables `-dSAFER` from inside the interpreter, exposing remote code execution across apps whose image or document pipelines silently delegate `.png` or `.pdf` uploads to Ghostscript.

4. Key Points
- **Implicit Ghostscript reachability**: Controlled tests on Ubuntu 24.04 showed that ordinary pixel-decoding or thumbnail calls in Pillow `13.0.0` (`Image.load()`, `crop()`, `thumbnail()`), `imageio`/`scikit-image`, PHP `Imagick::readImage()`, system `libvips` (`magickload`), and Qt `KImageFormats` inspect magic bytes rather than filenames—routing PostScript payloads disguised as `.png` or `.jpg` directly into Ghostscript, whereas PHP GD, OpenCV, Sharp, Jimp, Java `ImageIO`, and Go's standard image decoders did not.
- **Root cause in `.putgstringcopy`**: Found by GPT 5.6 Sol with Paulos Yibelo in `Resource/Init/gs_fonts.ps`, the font-loading helper used privileged `.forceput` after `dup length string copy` without validating operand types; substituting a local array for the filename string triggered a `typecheck` error whose document-defined `errordict /typecheck` handler popped five stack items and resumed execution at `.forceput`.
- **Use-after-free via local/global VM mismatch**: Because `.forceput` in `psi/zgeneric.c` temporarily treats `systemdict` as local during the store, the exploit places a local string into surviving global `systemdict`; a subsequent `save`/`restore` frees the local string while leaving a dangling reference in `systemdict`.
- **ASLR-independent native memory control**: Reusing the freed 2,048-byte string slot with marker-filled integer arrays (`0x12345678`) reveals an overlapping 16-byte Ghostscript `ref` without guessing heap addresses; forging a temporary string view over `currentdevice` lets the document walk pointers to `gs_lib_ctx_core_t + 156` (`path_control_active`) and zero it out, disabling `-dSAFER` path checks and `LockFilePermissions` to run `%pipe%` shell commands.
- **Remote attack chains despite "local" CVSS vector**: Although `CVE-2026-125396` is rated CVSS 4.0 `8.5` with a local attack vector, the researchers demonstrated remote execution across unauthenticated Debian 13 CUPS IPP print queues (`uid=7(lp)`), Roundcube 1.7.2 message thumbnailing (`www-data`), WordPress 7.0.2 public upload plugins (`Forminator`, `Essential Addons`), MediaWiki, DokuWiki, Mayan EDMS, and `/proc/version` reads on Google Docs/Chat/Drive/Gmail pipelines.
- **Fixes and uneven downstream patching**: Artifex patched the operand check and began moving `.putgstringcopy` into C without `.forceput` in Ghostscript `10.08.0` (September 8, 2026), and WordPress shipped `CVE-2026-65640` mitigations in `7.0.4`/`6.9.7`; however, as of November 12, 2026, Debian 12 (Bookworm), Amazon Linux 2/2023/2027 Preview, and several SUSE/openSUSE lines still showed vulnerable or pending packages.

5. Deep Dive (Structured Understanding)
**Problem**
Web applications, mail clients, and print servers routinely accept user-supplied images and documents, assuming image libraries like Pillow or ImageMagick only decode passive raster formats. In reality, PostScript is a Turing-complete language, and those libraries often delegate disguised `.png` or `.jpg` payloads to Ghostscript—relying on Ghostscript's internal `-dSAFER` sandbox to prevent file reads or command execution.

**Approach**
The researcher combined two techniques:
1. **Pipeline tracing**: Tested 29 image APIs to map which library calls (`load()`, `readImage()`, `thumbnail()`) inspect file headers and hand PostScript bytes to Ghostscript even under `.png` extensions.
2. **Interpreter exploitation**: Exploited missing type validation in `gs_fonts.ps`'s `.putgstringcopy` via a custom `errordict /typecheck` handler to hijack `.forceput`, planted a local object inside global `systemdict`, freed it with `restore`, reclaimed the chunk with typed `ref` arrays, and walked live C struct pointers from `currentdevice` to clear `path_control_active` in `gs_lib_ctx_core_t`.

**Key Insight**
An in-process interpreter sandbox like `-dSAFER` shares the same address space and C runtime state as the parser executing untrusted Turing-complete input. Any memory-corruption primitive inside the interpreter lets a document rewrite the sandbox's own enforcement flags (`path_control_active`)—and because outer image libraries dispatch by magic bytes before application-level extension checks or post-thumbnail validation, a "local" interpreter CVE immediately becomes remote code execution.

**Result / Impact**
The research produced a self-contained 104-line PostScript PoC (using Ghostscript's `%ram%` device with no external font file needed), prompted Artifex's `10.08.0` fix and WordPress's `CVE-2026-65640` update, and demonstrated why host OS package updates alone miss bundled or containerized Ghostscript copies.

6. Why It Matters
- **Security / DevSecOps**: Exposes the "hidden interpreter" supply-chain risk in routine file-upload, thumbnailing, and preview features—showing why trusting file extensions or relying on an interpreter's self-policed sandbox (`-dSAFER`) is insufficient without OS-level isolation.
- **Backend Engineering & Infrastructure**: Highlights how library choice (e.g., PHP GD or Sharp vs. Imagick/Pillow with Ghostscript installed) and order of operations (validating magic bytes *before* thumbnail generation, as in Roundcube) determine whether untrusted uploads reach a Turing-complete renderer.
- **Software-Industry Changes**: Notes that GPT 5.6 Sol assisted in discovering the `.putgstringcopy` flaw alongside human researchers, reflecting growing LLM involvement in auditing legacy C/PostScript codebases.

7. Critical Analysis
- **Overbroad "Upload2Shell" framing vs. single-dependency reality**: The research spans multiple distinct products and even separate undisclosed Ghostscript bugs (Dropbox and several redacted services had different root causes), while the primary `CVE-2026-125396` flaw is specifically a Ghostscript bug rather than an upload-framework vulnerability.
- **Environment-dependent exploitability**: Several listed chains require non-default or permissive configurations—such as Roundcube needing a permissive ImageMagick policy, MediaWiki/DokuWiki allowing anonymous uploads, or CUPS queues accepting unauthenticated network jobs—and many AI/web entries in the second table (Gradio, ComfyUI, InvokeAI) only proved Ghostscript *invocation* via Pillow rather than end-to-end RCE.
- **Incomplete distro-coverage table**: At retrieval, the article's closing distribution-package table truncated mid-sentence after Debian/Amazon/SUSE trackers, leaving RHEL/Fedora/Ubuntu exact package details partially unverified from this page alone.
- **HN community perspective**: Commenters criticized the "Upload2Shell" name as clickbait for a Ghostscript vulnerability (suggesting `GhostScript2Shell`) and found the broader disclosure timeline noisy; technically, practitioners emphasized that unsafe legacy file parsers like Ghostscript or ImageMagick must be wrapped in strict OS-level allowlist sandboxes (`seccomp` + `landlock` or OpenBSD `pledge`) rather than trusting `-dSAFER` or waiting on per-CVE patches.

8. Connections
- **ImageTragick (`CVE-2016-3714`) & Delegate Chains**: Directly extends the long-running class of ImageMagick/Pillow delegate vulnerabilities where content-sniffing routes disguised `.png`/`.jpg` uploads into external PostScript/SVG/MVG interpreters.
- **Artifex's Multi-Year `.forceput` Removal**: As Artifex maintainer Ken Sharp noted on bug 709684, `"it isn't easy to undo 30 years worth of assumptions in the code."`—echoing years of historical `-dSAFER` escapes rooted in PostScript-implemented helpers needing privileged dictionary access.
- **Kernel-Enforced vs. In-Interpreter Sandboxing**: Contrasts `-dSAFER` (which failed once native memory was corrupted inside `gs`) with kernel-enforced boundaries like Linux `seccomp-bpf` + `Landlock`, gVisor, or micro-VMs (e.g., Hyperlight/Firecracker), which hold even after arbitrary memory writes inside the renderer.

9. Keywords
Upload2Shell, CVE-2026-125396, Ghostscript, -dSAFER Bypass, Use-After-Free, ImageMagick, Pillow, File Upload Security, PostScript, OS Sandboxing

10. TL;DR
A use-after-free in Ghostscript's Type 1 font loader (`CVE-2026-125396`) lets a crafted PostScript file hijack `.forceput`, overwrite native `path_control_active` memory, and bypass `-dSAFER` to execute shell commands.
Because libraries like Pillow, `Imagick`, and `libvips` dispatch by file contents rather than extensions, `.png` or `.jpg` uploads silently reach Ghostscript across WordPress, Roundcube, CUPS, and AI web UIs.
Patching Ghostscript (`10.08.0`+) inside containers—not just hosts—validating file types before thumbnailing, and isolating file parsers with `seccomp`/`Landlock` are essential because in-interpreter sandboxes cannot survive memory corruption.
