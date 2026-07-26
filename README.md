# Hung-Jen (Daniel) Tu — Resume

<sub>杜弘仁 · Đặng Hoàng Nhân · published as Hung-Jen Tu</sub>

[![Build and Publish Resume PDF](https://github.com/danghoangnhan/resume/actions/workflows/build-resume.yml/badge.svg)](https://github.com/danghoangnhan/resume/actions/workflows/build-resume.yml)

LaTeX source for my resume. Every push to `main` rebuilds the PDF and republishes it, so
these links always serve the current version:

- **Read it online** — <https://danghoangnhan.github.io/resume/>
- **Direct PDF** — <https://danghoangnhan.github.io/resume/resume.pdf>
- **Versioned download** — <https://github.com/danghoangnhan/resume/releases/latest/download/resume.pdf>

<p align="center">
  <a href="https://danghoangnhan.github.io/resume/resume.pdf">
    <img src="https://danghoangnhan.github.io/resume/preview.png" alt="First page of the resume" width="600" />
  </a>
</p>

The preview above is rendered by CI from the PDF it just built, so it always matches what the
links serve.

## Layout

```
resume.tex          driver — document class, PDF metadata, section order
resumestyle.sty     packages, margins, colours, and every \resume* macro
sections/           one file per section; this is where the content lives
  header.tex          name, title, contact block, QR code
  education.tex       skills.tex        experience.tex   projects.tex
  opensource.tex      publications.tex  certificates.tex languages.tex
imgs/               contact icons (PDF) and the QR code
.github/pages/      index.html wrapper for the published site
```

To change what the resume *says*, edit a file in `sections/`. To change how it *looks*, edit
`resumestyle.sty`. To add or reorder sections, edit `resume.tex`.

## Building locally

There is no need to install TeX — this uses the same image CI does:

```sh
docker run --rm -v "$PWD:/w" -w /w ghcr.io/xu-cheng/texlive-full:latest \
  latexmk -pdf -file-line-error -halt-on-error -interaction=nonstopmode resume.tex
```

On Windows Git Bash, prefix the command with `MSYS_NO_PATHCONV=1` and give the volume an absolute
Windows path, or the shell rewrites `/w` into `W:/` and Docker rejects it.

The build must exit 0. `latexmk` fails the CI step on any LaTeX error even when it still
manages to emit a PDF, so a produced `resume.pdf` alone is not proof the build passed.

Exit 0 is also not proof the *layout* survived. CI additionally asserts the PDF is exactly two
pages and that `resume.log` contains no Overfull/Underfull boxes — check both locally too:

```sh
pdfinfo resume.pdf | grep '^Pages:'
grep -c 'Overfull\|Underfull' resume.log   # must be 0
```

## Writing sections

A few conventions worth knowing, all enforced by macros in `resumestyle.sty`:

| Macro | Use |
|---|---|
| `\resumeSection{Title}` | Section heading. Already coloured — don't wrap it in `\textcolor`. |
| `\resumeSubheading{org}{location}{role}{dates}` | Job or degree entry. Exactly **four** arguments. |
| `\resumeItem{label}{body}` | A labelled bullet. |
| `\resumeSubItem{label}{body}` | As `\resumeItem`, with trailing space trimmed. Used by `projects.tex` and `opensource.tex`. |
| `\resumeItemNoBullet{label}{body}` | Labelled line with no bullet, hung under a `\resumeSubheading`. Used by `education.tex`. |
| `\resumeLink{url}{text}` | **Every** link goes through this. See the third trap below. |
| `\resumeItemListStart` / `...End` | Bullets belonging to a `\resumeSubheading` (`education.tex` hangs `\resumeItemNoBullet` lines directly instead). |
| `\resumeNestedListStart` / `...End` | Sub-bullets **inside** a `\resumeItem` body. |

Three traps, the first two of which have broken this build before:

- **Use `\resumeNestedListStart`, never a bare `\begin{itemize}`, inside a macro argument.**
  Opening a list outside a brace group and closing it inside one mis-nests the environment.
- **`&` inside a `tabular`/`tabular*` is a column separator — do not escape it.** Only escape
  `\&` when you literally want an ampersand in running text. A blanket escape pass once turned
  every job entry into a single left-aligned line reading `Employer & Location`, and because it
  still compiles cleanly, CI will not catch it for you.
- **Never write a bare `\href`.** `hyperref` is loaded with `[hidelinks]`, so an unwrapped link
  prints in body black, indistinguishable from ordinary text — which is exactly what had happened
  to the header's portfolio and ORCID links. Use `\resumeLink{url}{text}`.

## Publishing

`.github/workflows/build-resume.yml` compiles on every push to `main` and on every pull
request. Pull requests build and upload the PDF as an artifact but publish nothing; only `main`
deploys to Pages and updates the rolling `latest` release.

GitHub Pages must be set to **Settings → Pages → Source: GitHub Actions** for the deploy job to
succeed.

## Credits

Built on the [academic-resume-latex](https://github.com/rancheng/academic-resume-latex) template
by [rancheng](https://github.com/rancheng), itself adapted from Indu Dwivedi and Sourabh Bajaj.

Contact icons come from [FreeIcons.io](https://freeicons.io), recoloured by editing the SVG
`fill` and converted with [CloudConvert](https://cloudconvert.com/svg-to-pdf). The QR code was
generated at [the-qrcode-generator.com](https://www.the-qrcode-generator.com/) and points to
<https://github.com/danghoangnhan> — if you regenerate it, keep that target or update this line.

## License

Non-commercial use only — see [LICENSE](LICENSE) for the full terms.
