# PlantDocket Backend

Backend planning for a browser-based factory document control system. It covers revisions of drawings and policies, remarks, multi-step departmental approvals, audit history, and approved guest access.

Web repository: [plantdocket-web](https://github.com/QuantumBreakz/plantdocket-web).

## Planning documents

- [Master Documents](docs/README.md)
- [Architecture](Architecture.md)
- [MVP scope](docs/mvp/01-scope.md)
- [Repository scaffolds](docs/scaffolding/README.md)
- [Backend scaffold](docs/scaffolding/backend.md)
- [Future mobile scaffold](docs/scaffolding/mobile.md)
- [Feature specifications and verification](docs/delivery/05-feature-verification.md)

## Write-ups

Design notes from this project, with tested Go code:

- [Multi-step document approval in Go: keep the old revision effective until the last step approves](https://aliahmed.si/blog/document-approval-workflow-go/)
- [Email one-time codes for guest file sharing in Go: hashing, attempt limits, expiry and races](https://aliahmed.si/blog/email-otp-guest-file-sharing-go/)
- [HTTP Range requests in Go: resumable downloads, 206 and 416, If-Range, and capping multipart ranges](https://aliahmed.si/blog/http-range-requests-go/)
