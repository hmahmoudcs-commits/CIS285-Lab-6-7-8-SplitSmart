# SplitSmart — Architecture Review (CIS 285 Lab 7)

**Reviewed by:** ChatGPT  
**Architecture A:** Spec Kit layered design (`architecture.md`)  
**Architecture B:** Codex hexagonal design (`alternative-architecture.md`)

| | A: Layered | B: Hexagonal |
|---|---|---|
| Simplicity | Easier | More complex |
| Maintainability | Good | Better-defined interfaces |
| Testing | Separated logic | Easier to isolate business rules |
| Web/mobile support | Logical client boundaries | Explicit adapters and shared core |

**Decision: Select B.** Its ports and adapters better isolate expense splitting, approvals, and balances from user interfaces and storage. **Trade-off:** More design complexity. Keep A's simpler domain model in the final design.

**Important:** Both are proposed designs; the repository review found no application source code to verify an implementation.
