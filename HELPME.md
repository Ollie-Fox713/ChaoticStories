# How to Contribute (Beginner's Guide)

This guide walks you through contributing audio, art, translations, or stories to the Chaotic project — even if you've never used Git before.

---

## What You'll Need

- A **GitHub account** (free) — [sign up here](https://github.com/signup)
- **Git** installed on your computer
- A **text editor** (any will do — Notepad, VS Code, etc.)
- Your files ready to contribute (audio, art, translations, etc.)

---

## Step 1: Install Git

### Windows

1. Download Git from [git-scm.com/download/win](https://git-scm.com/download/win)
2. Run the installer — the default options are fine for everything
3. When it finishes, you'll have a program called **Git Bash** — this is your terminal

### Mac

1. Open **Terminal** (search for it in Spotlight, or find it in Applications → Utilities)
2. Type `git --version` and press Enter
3. If Git isn't installed, macOS will prompt you to install it — follow the prompts

### Linux

Open your terminal and run:
```bash
sudo apt install git      # Ubuntu/Debian
sudo dnf install git      # Fedora
```

### Verify it works

Open your terminal (Git Bash on Windows, Terminal on Mac/Linux) and type:
```bash
git --version
```
You should see something like `git version 2.43.0`. If you do, you're good.

---

## Step 2: Set Up Git (One-Time)

Tell Git who you are (use the email linked to your GitHub account):

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

---

## Step 3: Fork the Repository

A "fork" is your own copy of the project on GitHub that you can freely edit.

1. Go to the ChaoticStories repository page on GitHub
2. Click the **Fork** button (top-right corner)
3. This creates a copy under your GitHub account

---

## Step 4: Clone Your Fork

"Cloning" downloads your fork to your computer so you can work on it locally.

1. On your fork's GitHub page, click the green **Code** button
2. Copy the URL (it looks like `https://github.com/YOUR-USERNAME/ChaoticStories.git`)
3. Open your terminal and run:

```bash
git clone https://github.com/YOUR-USERNAME/ChaoticStories.git
```

4. Move into the folder:

```bash
cd ChaoticStories
```

---

## Step 5: Create a Branch

Branches keep your changes separate from the main project until they're ready. Always create a new branch for your work.

```bash
git checkout -b my-contribution
```

Replace `my-contribution` with a short description like `add-battle-music` or `translate-pt-stories`.

---

## Step 6: Add Your Files

Now just put your files in the right place. Here's where things go:

### Audio files

| What | Where to put it |
|------|----------------|
| Music tracks | `audio/music/` |
| Sound effects | `audio/sfx/` |
| Voice lines | `audio/voices/{creature_name}/` |

**Audio format:** `.webm` with Opus codec is preferred. If you only have `.mp3` or `.wav`, that's okay too.

**Music example:** Drop your track into `audio/music/`:
```
audio/music/my_new_battle_theme.webm
```

**Voice line example:** Create a folder for the creature, then add your clips:
```
audio/voices/chaor/encounter.webm
audio/voices/chaor/attack.webm
```

### Art files

| What | Where to put it |
|------|----------------|
| Card art (creatures, attacks, etc.) | `art/cards/{type}/{CardName}.png` |
| UI images | `art/ui_images/` |
| Avatar layers | `art/avatar/{layer}/` |

### Translations

| What | Where to put it |
|------|----------------|
| UI strings | `i18n/{locale}.json` (e.g., `pt.json`) |
| Card text | `cards/{locale}/cards.json` |
| Stories | `stories/{locale}/` |

---

## Step 7: Stage and Commit Your Changes

Once your files are in the right folders:

1. **See what changed** (optional, just to check):
   ```bash
   git status
   ```
   You should see your new/modified files listed in red.

2. **Stage everything** (tells Git "I want to include these"):
   ```bash
   git add .
   ```

3. **Commit** (saves a snapshot with a message describing what you did):
   ```bash
   git commit -m "Add battle music track"
   ```

   Write a short, clear message. Examples:
   - `"Add Chaor voice lines"`
   - `"Translate Portuguese UI strings"`
   - `"Add Kiru City card art"`

---

## Step 8: Push to GitHub

Upload your branch to your fork on GitHub:

```bash
git push origin my-contribution
```

(Use whatever branch name you chose in Step 5.)

---

## Step 9: Create a Pull Request (PR)

A Pull Request asks the project maintainers to review and merge your changes.

1. Go to your fork on GitHub — you should see a yellow banner saying your branch was recently pushed, with a **Compare & pull request** button. Click it.
   - If you don't see the banner, click the **Pull requests** tab, then **New pull request**
2. Make sure it says:
   - **base repository:** the main ChaoticStories repo
   - **base:** `main`
   - **head repository:** your fork
   - **compare:** your branch name
3. Write a title and description:
   - **Title:** Short summary, e.g., "Add battle music track"
   - **Description:** What you added, any notes (e.g., "Recorded at 128kbps Opus, 2:30 loop")
4. Click **Create pull request**

That's it! The maintainers will review your contribution and merge it if everything looks good. They might leave comments if anything needs adjusting.

---

## Updating Your Fork Later

If you want to contribute again later, update your fork first to get the latest changes:

```bash
cd ChaoticStories
git checkout main
git pull upstream main
```

If you haven't set up "upstream" yet:
```bash
git remote add upstream https://github.com/ORIGINAL-OWNER/ChaoticStories.git
git pull upstream main
```

Then create a new branch and repeat from Step 5.

---

## Common Problems

### "Permission denied" when pushing

You probably need to authenticate with GitHub. The easiest way:

1. Go to [github.com/settings/tokens](https://github.com/settings/tokens)
2. Click **Generate new token (classic)**
3. Give it `repo` permissions, generate it, and copy the token
4. When Git asks for your password during `git push`, paste the token instead

Or install [GitHub CLI](https://cli.github.com/) and run `gh auth login` for a smoother experience.

### "Your branch is behind"

Your fork is out of date. Follow the "Updating Your Fork Later" section above.

### "Merge conflict"

This means someone else changed the same file. Don't panic — ask for help in the project's issues or Discord, and a maintainer can help you resolve it.

### I messed something up

If you haven't pushed yet, you can undo your last commit:
```bash
git reset HEAD~1
```

If things are really tangled, you can always delete your local copy and re-clone:
```bash
cd ..
rm -rf ChaoticStories
git clone https://github.com/YOUR-USERNAME/ChaoticStories.git
```

---

## Quick Reference

| Action | Command |
|--------|---------|
| Check what changed | `git status` |
| Stage all files | `git add .` |
| Commit with message | `git commit -m "your message"` |
| Push to GitHub | `git push origin branch-name` |
| Create a new branch | `git checkout -b branch-name` |
| Switch to main | `git checkout main` |
| Pull latest changes | `git pull upstream main` |
