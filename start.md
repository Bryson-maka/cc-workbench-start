# Start a new cc-workbench Mac

You are Claude Code on a new Apple Silicon Mac. The person who installed and signed you in gave
you this file's link because they want this Mac set up as a cc-workbench machine. This file
takes you as far as a private workbench checkout on this Mac. That checkout's
`docs/BRINGUP.md` owns every step after it.

## Rules for every step

- Use this file's exact text. If a web tool gave you a summary of it, fetch the raw link with
  `curl -fsSL LINK -o "$TMPDIR/cc-workbench-start.md"` and read that file. Take commands only
  from the exact text.
- Look before you install. Reuse every tool, sign-in and folder that already works, and
  install only what is missing, through the vendor's own installer.
- The person handles every browser sign-in, password, admin prompt, macOS dialog and account
  choice. Never read, print, store or copy a password, token or private key, and never ask for
  one in the conversation. Public keys are exchanged on purpose later in setup.
- When a step needs the Mac's admin password, give the person the exact command to run in the
  Terminal app, then check the result yourself. Your shell tool has no terminal for a password
  prompt. Never set up passwordless sudo.
- Do not change FileVault, login, lock-screen or privacy settings, or another user's files.
- Tell the person plainly what you found, what you installed, what they must do next and what
  you skipped.

## 1. Look at this Mac

```bash
sw_vers; uname -m; id -un; echo "$SHELL"
xcode-select -p
command -v git python3 brew gh
ls -d /opt/homebrew/bin/brew /opt/homebrew/bin/gh ~/Projects ~/Projects/cc-workbench 2>/dev/null
```

`uname -m` must print `arm64`; otherwise stop and say this guide covers Apple Silicon Macs
only. Note the permission mode you are running in as you observe it; do not change it.

## 2. Command Line Tools

If `xcode-select -p` printed a folder, reuse it. Otherwise run `xcode-select --install`: macOS
shows a dialog. Ask the person to choose Install and accept the license, then wait until
`xcode-select -p` prints a folder. These tools provide Git and Python 3. Running `git` or
`python3` before they are installed can open the same dialog.

## 3. Homebrew

If `/opt/homebrew/bin/brew` exists, reuse it. Otherwise ask the person to run this command from
[Homebrew's installation page](https://docs.brew.sh/Installation) in the Terminal app. It asks
for their Mac password and confirms before it changes anything.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

When they say it finished, run `/opt/homebrew/bin/brew --version`. The installer's "Next steps"
names a shell line for `~/.zprofile`. Add it only when no `brew shellenv` line is there yet;
never replace the file:

```bash
grep -qs 'brew shellenv' ~/.zprofile || echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
```

If `$SHELL` is not zsh, follow the installer's own next-steps text for that shell instead. Your
own shell does not reread the profile: use absolute `/opt/homebrew/bin/...` paths for the rest
of this conversation.

## 4. An existing checkout

The checkout lives at `~/Projects/cc-workbench`. If that folder does not exist, go to step 5.

If it exists, inspect it and keep it. Never reset, clean, check out, delete or replace it, and
never change its remote or the way it reaches GitHub.

```bash
git -C ~/Projects/cc-workbench remote get-url origin
git -C ~/Projects/cc-workbench status --short --branch
git -C ~/Projects/cc-workbench rev-parse HEAD
git -C ~/Projects/cc-workbench ls-remote origin HEAD
```

Continue when `origin` names `github.com/Bryson-maka/cc-workbench` (the HTTPS or SSH form) and
the status starts `## main...origin/main` with no tracked changes. Otherwise report exactly what
you see and stop there; the person decides what happens to that folder.

`ls-remote origin` tests the access this checkout already uses, over its own transport, and
prints the published source's current commit. When it prints a commit, that access works: keep
it and go to step 7 without any GitHub sign-in. When it fails, go to step 5 for access only, and
keep this checkout.

The status line cannot show whether the checkout is current: it compares with the checkout's
last fetch, so a checkout behind the published source still reads `## main...origin/main`. Step
7 compares `rev-parse HEAD` with the commit `ls-remote` printed instead.

## 5. Access to the private source

This step runs only for a new checkout, or when the existing checkout's access failed. The
source is the private repository `https://github.com/Bryson-maka/cc-workbench`.

Reuse a working `gh`. Otherwise run `/opt/homebrew/bin/brew install gh`. Below, `gh` means the
path `command -v gh` printed, or `/opt/homebrew/bin/gh`. Check the access that already exists
before any sign-in:

```bash
gh auth status --hostname github.com
git ls-remote https://github.com/Bryson-maka/cc-workbench.git HEAD
```

- `git ls-remote` printed a commit: access works. Keep it as it is. Do not sign in again, log
  out, refresh or switch accounts, even when `gh auth status` shows a narrow token.
- gh is signed in but Git was refused: Git may not be using gh's sign-in. Tell the person that
  `gh auth setup-git --hostname github.com` makes Git use gh's sign-in for github.com. With
  their yes, run it and check again with `git ls-remote`.
- No account is signed in, or the signed-in account cannot see the repository: explain that the
  workbench is private and GitHub must be told which account may read it. Offer both native
  ways and let the person choose:
  - **Browser sign-in.** The person runs
    `gh auth login --hostname github.com --git-protocol https --web` in the Terminal app, picks
    the account in the browser and enters the one-time code gh shows. Say before they start
    that this asks GitHub for broad access (gh's help names the `repo`, `read:org` and `gist`
    scopes); `repo` covers every repository that account can reach, not only this one. The
    browser page lists what is requested.
  - **A narrower credential.** The person creates a fine-grained token limited to this
    repository on github.com, then runs `gh auth login` in the Terminal app and pastes it at
    gh's own prompt. You never see it.

  When they say it finished, check again with `gh auth status --hostname github.com` and
  `git ls-remote`. Run `gh auth setup-git --hostname github.com` only if Git is still refused.

An existing checkout whose own `origin` uses SSH keeps it: when HTTPS works but its SSH access
does not, report that difference to the person instead of changing the remote. Then go to step
7 for an existing checkout, or step 6 for a new one.

## 6. A new checkout

```bash
mkdir -p ~/Projects
git clone https://github.com/Bryson-maka/cc-workbench.git ~/Projects/cc-workbench
git -C ~/Projects/cc-workbench status --short --branch
```

The status must read `## main...origin/main` and nothing else.

## 7. Continue from the checkout

Record the commit `git -C ~/Projects/cc-workbench rev-parse HEAD` prints: it is the source this
setup used.

The checkout is current when it is the new clone from step 6, or when its `rev-parse HEAD`
equals the commit `git ls-remote` printed in step 4 or 5.

If it is current and `~/Projects/cc-workbench/docs/BRINGUP.md` exists, read that file in full
and follow it from its first step. It uses the checkout's own `cc-machine` skill, and its
current text outranks this file. Tell the person in one line that the workbench source is on
this Mac at the recorded commit and that you are continuing with its setup guide.

Otherwise the checkout differs from the published source and needs its normal update first:
its own `docs/BRINGUP.md` may be older or missing. Tell the person in one line that updating
fast-forwards the checkout from its own upstream and keeps any local work; with their yes:

- When `~/.config/cc-workbench/installation.json` records this checkout as its
  `source_checkout`, run its cc-machine source update, which refuses local changes, divergence
  and unpublished commits:
  `python3 -B ~/Projects/cc-workbench/.agents/skills/cc-machine/scripts/machine.py update-source --json`.
  The guide you continue with installs the update.
- Otherwise the workbench is not installed from it yet: run
  `git -C ~/Projects/cc-workbench pull --ff-only`.

Relay any refusal and stop; never reset, stash, rebase or merge to get past it. After the
update, run `rev-parse HEAD` and the `ls-remote` from step 4 or 5 again. When they match, record
the new commit and continue with `docs/BRINGUP.md` as above. When they still differ, report both
commits and stop there; the person decides what happens to that folder.
