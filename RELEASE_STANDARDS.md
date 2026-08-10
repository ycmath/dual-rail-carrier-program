# Release Standards of the Dual-Rail Carrier Program

Every release in the series passes the following checklist before it is
published. The checklist exists so that each repository is independently
auditable and so that the series stays internally consistent.

## 1. Manuscript standard

- Full rigorous proofs — no proof sketches. A main-text theorem may
  derive from appendix results, but every cited result is proved in
  full somewhere in the paper.
- Certificate-level computational premises (finite closures, exhaustive
  saturations) are allowed, but must be (a) stated as explicit Facts,
  separate from theorems, (b) shipped as replayable artifacts with
  frozen outputs, and (c) independently reproduced at release time.
- Scope boundaries stated in print: what is proved, for which
  parameters, and what is open. No silent overclaims; corrections of
  earlier statements are recorded as visible remarks.
- References verified against primary sources (DOIs where they exist)
  before inclusion.
- Author contact and provenance section present; the division of labor
  between the author's mathematics and AI-assisted verification is
  stated explicitly.

## 2. Machine-verification standard

- Lean 4, core only: no mathlib, no `native_decide`, no `sorry`, no
  `axiom` declarations. Toolchain pinned via `lean-toolchain`.
- Kernel axiom profile at most `[propext, Quot.sound]` for every
  theorem; the shipped `axcheck.log` records per-theorem
  `#print axioms` output of a clean build.
- Known pitfalls (avoid): `omega` closing a conjunction goal directly
  pulls in `Classical.choice` — split with `constructor <;> omega`;
  closing `0 = n+1`-style length contradictions with `simp at` pulls in
  `Classical.choice` — use `Nat.noConfusion`.
- Independent replay: every finite/enumerable claim of the paper is
  re-checked by a standalone script (stdlib only), with the frozen
  output committed alongside.

## 3. Repository layout

```
paper/          paper.md + paper.tex + paper.pdf
lean/           the Lean development (self-contained lake project)
verification/   replay scripts + frozen outputs (+ certificates and an
                independent rebuild, when certificate premises exist)
README.md       result statement, contents, verification status,
                companion links, provenance, licenses
CITATION.cff    citation metadata (see §5)
LICENSE         Apache-2.0 (code, Lean, scripts, certificates)
LICENSE-text    CC BY 4.0 (text)
```

## 4. PDF quality gate

- Compile with **three** pdflatex passes; the log must contain zero
  "undefined references" / "Rerun to get" warnings.
- Extract the text of the final PDF (e.g. with pypdf) and assert zero
  occurrences of "??" before committing. Never commit a PDF produced by
  a single aux-less pass.
- LaTeX sources are ASCII-clean (Unicode goes in the markdown edition;
  the TeX edition uses macros).

## 5. Citation metadata (Zenodo-compatible)

- `CITATION.cff` uses **cff-version 1.1.0** with a **single SPDX
  license id** — Zenodo's citation loader rejects cff 1.2.0 extras and
  license lists/AND-expressions ("Citation metadata load failed").
- After the DOI is minted, record the **concept DOI** (resolves to the
  latest version) in `CITATION.cff` (`doi:`) and at the top of the
  README.

## 6. Publication flow

1. Stage the complete repository locally; run a leak scan (no internal
   working-document names, no internal path references, no internal
   status vocabulary; historical tag strings inside byte-bound
   certificates are kept verbatim and disclosed in the README).
2. Push to GitHub (public) and create release `v1.0.0`.
3. Enable the repository in the Zenodo GitHub integration, then publish
   an archival release (`v1.0.1`) to trigger the DOI mint.
4. Record the concept DOI in the repository (commit, no new release —
   a new release would create a new Zenodo version).
5. Update cross-references in companion repositories (and recompile
   their PDFs under §4) so the citation graph stays closed at DOI
   level.

## 7. Cross-reference discipline

- Papers cite companions by title + concept DOI + repository URL.
- When a release discharges a citation IOU in an earlier paper (a fact
  previously attributed to "a companion line"), the earlier paper is
  patched to cite the released DOI, and the patch is synced to Zenodo
  with a release.
