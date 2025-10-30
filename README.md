# VIEW OF ERD FILE CREATED USING AGENT
![alt text](image.png)



# ERD Repository (bigER + VS Code)

This repository is for students to submit Entity‑Relationship Diagrams (ERDs) using the bigER VS Code extension.

## Prerequisites
- VS Code installed
- Git installed and a GitHub account
- Java 11+ runtime (JDK recommended)
- VS Code extension: BIGER Modeling Tool (`BIGModelingTools.erdiagram`)

### Install Java quickly
- macOS (Homebrew): `brew install --cask temurin17`
- Windows (winget): `winget install --id EclipseAdoptium.Temurin.17.JDK -e`
- Verify: `java -version` (shows 11+)

## Submission (No PRs/Merges)
You will submit your ERD via a GitHub Issue — no forks or pull requests required.

1) Create your ERD in VS Code
- Ensure bigER extension is installed and Java works: `java -version` shows 11+.
- Create a file at the repo root, for example: `Firstname_Lastname.erd`.
- Use Command Palette → “New Sample ER Model” to start, or write your own.
- Open the diagram from the editor toolbar or press Ctrl/⌘+O.

2) Submit via GitHub Issue
- Go to: https://github.com/BADM554/ERD/issues → “New issue”.
- Choose “ERD Submission” template (if available) or open a blank issue.
- Title: `Submission: Firstname Lastname`.
- Attach your `.erd` file by dragging it into the issue body.
- Provide any requested details (name, section, email) and click “Submit new issue”.

3) Updates/Resubmissions
- If you need to resubmit, open a new issue with the updated `.erd` attached.

## Optional: Git one‑time setup (only if you plan to use Git locally)
Run once in a VS Code terminal:
```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

## Notes
- You do not need to fork or open a pull request for submissions.
- Place your `.erd` file(s) at the repository root when working locally.
- Keep filenames consistent with the convention `Firstname_Lastname.erd` unless instructed otherwise.

## Troubleshooting
- bigER inactive or no diagram: ensure the file ends with `.erd` and the extension is enabled.
- Java not found: install JDK 11+ (see above) and restart VS Code.
