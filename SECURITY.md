# Security policy

## Supported scope

Security fixes ship on the default branch (**`master`**). Consume this reusable workflow by pinning a commit SHA or tag rather than blindly tracking **`master`** when supply-chain control matters.

## Reporting a vulnerability

Email **Garret Patten** at **<garret.patten@proton.me>** with:

- Brief description of impact and suspected component (workflow job, scanner config, action pin, or documentation).
- Whether you believe it is remotely exploitable and any proof-of-concept you can safely share.

You should receive acknowledgement of receipt; substantive updates align with remediation progress. If a finding is declined, reasoning will be given.

### Out of scope without prior agreement

- Social engineering against maintainers or users.
- Findings in consumer repositories that call this workflow (report those to the consuming project first).
- Theoretical attacks without a plausible path through this repository’s workflow definitions or release artifacts.
