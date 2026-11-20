# Stage 2: PS5 system-library compatibility audit

Audit target: `CaptainXack/AnyPS5` branch `local-fixes`, 2026-11-20.

This is a source-backed triage, not a claim of full PS5 API coverage. See `docs/dev/TechnicalDebt.md` and the library-specific files in `docs/dev/debt/`. A successful build or synthetic ELF launch does not establish game compatibility.

| Priority | Area | Confirmed limitation | Source | Next verification |
| --- | --- | --- | --- | --- |
| P1 | libkernel pthread cancellation | File/socket I/O cancellation, sleep interruption, exit/detach races and condition-signal conservation are unverified. | `docs/dev/TechnicalDebt.md`, Functional | Add deterministic host-side cancellation and race tests before changing semantics. |
| P1 | libkernel mutex protocols | Priority inheritance/protection and priority ceiling are stored or accepted, but not applied to mutex ownership/scheduling. | `docs/dev/debt/libkernel.md`, Silent stubs; `core/libs/prx/libkernel/Pthread/Posix/Mutex.cpp` | Test protocol attribute validation and document host scheduling limitations. Do not silently claim console-equivalent priority inheritance. |
| P1 | libkernel read/write locks | `pthread_rwlockattr_settype_np` stores types 1 and 2, but rwlock initialisation ignores the attribute. | `docs/dev/debt/libkernel.md`, Silent stubs; `core/libs/prx/libkernel/Pthread/Posix/Rwlock.cpp` | Add regression tests for attr storage and lock behaviour; implement policy only with grounded semantics. |
| P2 | libkernel internal memory | `sceKernelInternalMemoryGetAvailableSize` does not model an internal allocation pool; returns failure values without filling a valid output. | `docs/dev/debt/libkernel.md`, Silent stubs; `core/libs/prx/libkernel/DirectMemory/Export.cpp` | Test error paths and nonmutation of output; requires evidence for successful pool semantics. |
| P2 | libkernel direct-memory aliasing | `sceKernelEnableDmemAliasing` returns success without changing mapping behaviour; aliasing is always allowed by the host implementation. | `docs/dev/debt/libkernel.md`, Silent stubs; `core/libs/prx/libkernel/DirectMemory/Export.cpp` | Test repeated mappings and distinguish host behaviour from unverified PS5 rules. |
| P2 | libkernel notifications | `sceKernelSendNotificationRequest` writes a message to stderr, not a native PS5 notification device/UI. | `docs/dev/debt/libkernel.md`, Unknown function info; `core/libs/prx/libkernel/System/src/Notification.cpp` | Verify supported request validation and failure codes; avoid claiming a console UI. |
| P3 | Missing/unknown signatures | Some AudioIn, NpSessionSignaling, Font, Http2, Net, Ssl, SystemService, Share, NpTrophy2, NpEntitlementAccess, WebBrowserDialog and other exports have assumed or unknown signatures. | `docs/dev/TechnicalDebt.md`, Unknown function info | Inventory unresolved exports and match against legally available API definitions or test observations before implementing. |

## Implementation order

1. Inventory current automated tests for mutex and rwlock attributes, cancellation, and direct memory.
2. Add regression tests for **known** behaviour, including documented errors and output-buffer preservation.
3. Fix any discrepancies those tests expose. Do not turn throwing stubs into fabricated successes.
4. Run focused Windows tests and full CI on `local-fixes`.
5. Only then expand to console-dependent functions with verified signatures and semantics.

## Scope and caution

- No proprietary console firmware, decrypted game files or keys are required for this audit.
- The project's library completion percentages cover known exports only.
- Unknown console behaviour must remain explicitly marked as unverified.
- No changes to upstream or the fork's `main` branch are authorised by this audit.
