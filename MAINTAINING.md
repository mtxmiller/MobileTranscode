# MobileTranscode Plugin Maintenance Guide

This guide explains how to maintain and update the MobileTranscode plugin.

## Table of Contents

1. [Submitting to Official LMS Repository](#submitting-to-official-lms-repository)
2. [Handling Pull Requests & Updates](#handling-pull-requests--updates)
3. [Release Checklist](#release-checklist)

---

## Submitting to Official LMS Repository

### Overview

The LMS plugin repository automatically collects plugins from individual author repositories and merges them into one central XML file. Users can then discover and install your plugin directly from LMS without manually adding repository URLs.

### Steps to Submit

#### 1. Fork the LMS Plugin Repository

```bash
# Via GitHub web interface
Go to: https://github.com/LMS-Community/lms-plugin-repository
Click "Fork" button
```

Or via command line:
```bash
gh repo fork LMS-Community/lms-plugin-repository --clone
cd lms-plugin-repository
```

#### 2. Add Your Repository URL to include.json

Edit `include.json` and add your repo.xml URL:

```json
{
  "repositories": [
    "https://existing-repo.com/repo.xml",
    "https://raw.githubusercontent.com/mtxmiller/MobileTranscode/main/repo.xml"
  ]
}
```

**Important**: Add your URL to the end of the array.

#### 3. Commit and Push

```bash
git add include.json
git commit -m "Add MobileTranscode plugin repository

Mobile Transcode Rules plugin provides optimized custom-convert.conf
rules for mobile Squeezebox clients (LyrPlay, xTune, Squeezelite).
"
git push origin main
```

#### 4. Create Pull Request

```bash
gh pr create --title "Add MobileTranscode plugin" --body "Add MobileTranscode plugin repository to official LMS plugin list.

**Plugin**: Mobile Transcode Rules
**Repository**: https://github.com/mtxmiller/MobileTranscode
**Purpose**: Optimized transcode rules for mobile clients

This plugin provides custom-convert.conf rules for mobile Squeezebox apps with proper FT flag usage to prevent timeout errors.
"
```

Or via GitHub web interface:
1. Go to your fork
2. Click "Pull requests" → "New pull request"
3. Fill in title and description
4. Submit

#### 5. Wait for Review

- The LMS Community maintainers will review your PR
- They may ask questions or request changes
- Once approved, a GitHub Action automatically merges plugin repositories
- Your plugin will appear in the official LMS plugin list within a few hours

### After Acceptance

Once your PR is merged:
- Your plugin appears in LMS Settings → Plugins → Third Party automatically
- Users no longer need to manually add your repo URL
- Updates to your repo.xml are picked up automatically by the build system

---

## Handling Pull Requests & Updates

### Scenario: Someone Adds Their App's Rules

When another developer submits a PR to add their mobile app's transcode rules to `custom-convert.conf`, follow these steps:

### Step 1: Review the Pull Request

**Check:**
- [ ] Rules follow the template format
- [ ] Model name is unique and descriptive
- [ ] Quality settings are reasonable
- [ ] Rules are properly commented
- [ ] README is updated with app name
- [ ] Developer has tested the rules

**Example review comment:**
```
Thanks for adding [AppName] rules!

✅ Rules look good
✅ Following template format
✅ README updated

Please confirm:
- Have you tested these rules with your app?
- What platform is this optimized for (Android/iOS/other)?
```

### Step 2: Merge the Pull Request

Once approved:
```bash
# Via GitHub CLI
gh pr merge <PR-NUMBER> --squash

# Or via GitHub web interface
Click "Squash and merge"
```

### Step 3: Update Version Number

Edit `install.xml` and bump the version:

```xml
<!-- Before -->
<version>1.0.0</version>

<!-- After -->
<version>1.1.0</version>
```

**Version numbering:**
- **Major (X.0.0)**: Breaking changes, major restructuring
- **Minor (1.X.0)**: New features (like adding a new app's rules)
- **Patch (1.0.X)**: Bug fixes, documentation updates

### Step 4: Commit Version Bump

```bash
git add install.xml
git commit -m "Bump version to 1.1.0

Added [AppName] transcode rules via PR #XX
"
git push origin main
```

### Step 5: Create New Release Zip

**IMPORTANT:** Only include these files in the zip, DO NOT include `repo.xml` or `.git`:

```bash
# From parent directory (Documents-Mac-Mini)
cd /Users/ericmiller/Documents/Documents-Mac-Mini

# Create zip with only the plugin files (NOT repo.xml)
zip -r MobileTranscode-1.1.0.zip \
  MobileTranscode/Plugin.pm \
  MobileTranscode/install.xml \
  MobileTranscode/strings.txt \
  MobileTranscode/custom-convert.conf \
  MobileTranscode/README.md

# Verify zip was created
ls -lh MobileTranscode-1.1.0.zip
```

**Files included in zip:**
- `Plugin.pm` - Main plugin code
- `install.xml` - Plugin metadata
- `strings.txt` - Localization strings
- `custom-convert.conf` - Transcode rules
- `README.md` - Documentation

**Files EXCLUDED from zip:**
- `repo.xml` - (only used on GitHub, not in releases)
- `.git/` - (git history, not needed in release)
- `CLAUDE.md`, `MAINTAINING.md` - (development docs only)

### Step 6: Calculate New SHA1

```bash
shasum -a 1 MobileTranscode-1.1.0.zip
```

**Example output:**
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0  MobileTranscode-1.1.0.zip
```

**Copy this SHA1** - you'll need it in the next step.

### Step 7: Update repo.xml

Edit `MobileTranscode/repo.xml`:

```xml
<?xml version="1.0"?>
<extensions>
  <details>
    <title lang="EN">Mobile Transcode Rules Repository</title>
  </details>
  <plugins>
    <!-- UPDATE these three fields: -->
    <plugin name="MobileTranscode" version="1.1.0" minTarget="7.9" maxTarget="9.*">
      <title lang="EN">Mobile Transcode Rules</title>
      <desc lang="EN">Optimized custom-convert.conf rules for mobile Squeezebox clients (LyrPlay, xTune, Squeezelite). Uses FT flag for fast pipeline launch to prevent timeout errors.</desc>

      <!-- UPDATE URL to new version -->
      <url>https://github.com/mtxmiller/MobileTranscode/releases/download/v1.1.0/MobileTranscode-1.1.0.zip</url>

      <!-- UPDATE SHA1 from Step 6 -->
      <sha>a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0</sha>

      <link>https://github.com/mtxmiller/MobileTranscode</link>
      <creator>Eric Miller (mtxmiller)</creator>
      <email>ericlmiller@gmail.com</email>
    </plugin>
  </plugins>
</extensions>
```

**Changes to make:**
1. `version="1.1.0"` (line 7)
2. `<url>` - update version in download URL
3. `<sha>` - update with new SHA1 from Step 6

### Step 8: Commit repo.xml Update

```bash
cd MobileTranscode

git add repo.xml
git commit -m "Update repo.xml for v1.1.0 release

New SHA1: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0
Added [AppName] rules
"
git push origin main
```

### Step 9: Create GitHub Release

```bash
# Tag the release
git tag v1.1.0
git push origin v1.1.0

# Create GitHub release with new zip
gh release create v1.1.0 ../MobileTranscode-1.1.0.zip \
  --title "v1.1.0 - Add [AppName] Rules" \
  --notes "## What's New

- Added transcode rules for [AppName] (Platform)
- Updated by @username via PR #XX

## Currently Supported Apps

- LyrPlay (iOS)
- [AppName] (Platform)

## Installation

Install via LMS Settings → Plugins → Third Party.

Or add repository manually:
\`https://raw.githubusercontent.com/mtxmiller/MobileTranscode/main/repo.xml\`

## Files Changed

- \`custom-convert.conf\`: Added [AppName] rules
- \`README.md\`: Updated supported apps list
- \`install.xml\`: Version bump to 1.1.0
- \`repo.xml\`: Updated SHA1 and release URL
"
```

### Step 10: Verify Everything

**Check these items:**

- [ ] New release appears at: https://github.com/mtxmiller/MobileTranscode/releases
- [ ] Zip file is attached to release
- [ ] repo.xml on main branch has new version and SHA1
- [ ] In LMS, plugin updates automatically (may take a few minutes)

### Step 11: Test the Update

**In LMS:**
1. Go to Settings → Plugins
2. Find "Mobile Transcode Rules"
3. If it shows an update available, click "Update"
4. Restart LMS
5. Verify new rules are present in the custom-convert.conf

---

## Release Checklist

Use this checklist for every release:

### Pre-Release

- [ ] All PRs merged and tested
- [ ] Version bumped in `install.xml`
- [ ] CHANGELOG.md updated (if you create one)
- [ ] README.md updated with new apps/features

### Release Process

- [ ] Commit version bumps to `install.xml` and `repo.xml`
- [ ] Create zip with **only** these files: `Plugin.pm`, `install.xml`, `strings.txt`, `custom-convert.conf`, `README.md`
- [ ] Calculate SHA1: `shasum -a 1 MobileTranscode-X.Y.Z.zip`
- [ ] Update `repo.xml` with new version, URL, and SHA1
- [ ] Commit repo.xml changes
- [ ] Create git tag: `git tag vX.Y.Z`
- [ ] Push tag: `git push origin vX.Y.Z`
- [ ] Create GitHub release with zip attached: `gh release create vX.Y.Z ../MobileTranscode-X.Y.Z.zip --title "vX.Y.Z - ..." --notes "..."`
- [ ] Write release notes with "What's Fixed/New" and "Installation" sections

### Post-Release

- [ ] Verify release appears on GitHub
- [ ] Verify zip is downloadable
- [ ] Test update in LMS
- [ ] Announce in forums (if significant update)

---

## Quick Reference Commands

### Check Current Version
```bash
grep -o 'version="[^"]*"' install.xml
```

### Create Release Zip (Correct Files Only)
```bash
VERSION="1.2.1"
cd /Users/ericmiller/Documents/Documents-Mac-Mini

zip -r MobileTranscode-${VERSION}.zip \
  MobileTranscode/Plugin.pm \
  MobileTranscode/install.xml \
  MobileTranscode/strings.txt \
  MobileTranscode/custom-convert.conf \
  MobileTranscode/README.md
```

### Calculate and Display SHA1
```bash
VERSION="1.2.1"
cd /Users/ericmiller/Documents/Documents-Mac-Mini
SHA1=$(shasum -a 1 MobileTranscode-${VERSION}.zip | awk '{print $1}')
echo "SHA1: $SHA1"
# Copy this SHA1 to repo.xml
```

### Full Release Pipeline
```bash
VERSION="1.2.1"
cd /Users/ericmiller/Documents/Documents-Mac-Mini/MobileTranscode

# 1. Commit version changes
git add install.xml repo.xml
git commit -m "Bump version to ${VERSION}"

# 2. Create zip with correct files only
cd ..
zip -r MobileTranscode-${VERSION}.zip \
  MobileTranscode/Plugin.pm \
  MobileTranscode/install.xml \
  MobileTranscode/strings.txt \
  MobileTranscode/custom-convert.conf \
  MobileTranscode/README.md

# 3. Get SHA1
SHA1=$(shasum -a 1 MobileTranscode-${VERSION}.zip | awk '{print $1}')

# 4. Update repo.xml manually with new SHA1 and version

# 5. Commit repo.xml, tag, and push
cd MobileTranscode
git add repo.xml
git commit -m "Update repo.xml for v${VERSION} release

SHA1: $SHA1"
git tag v${VERSION}
git push origin main v${VERSION}

# 6. Create GitHub release
gh release create v${VERSION} ../MobileTranscode-${VERSION}.zip \
  --title "v${VERSION} - Fix/Feature description" \
  --notes "## What's Fixed/New
- Description here
"
```

---

## Troubleshooting

### "Plugin won't update in LMS"

- Check that repo.xml has the new version number
- Verify the SHA1 matches the zip file exactly
- Wait 5-10 minutes (LMS caches plugin data)
- Try Settings → Plugins → Rescan

### "SHA1 mismatch error"

- Recalculate SHA1: `shasum -a 1 MobileTranscode-X.Y.Z.zip`
- Verify you're using SHA1 (not SHA256)
- Make sure you're hashing the correct zip file
- Update repo.xml with exact SHA1 from command output

### "Can't create release"

- Ensure git tag exists: `git tag -l`
- Ensure tag is pushed: `git push origin vX.Y.Z`
- Check GitHub release permissions

---

## Getting Help

- **LMS Forums**: https://forums.lyrion.org
- **Plugin Repository Issues**: https://github.com/LMS-Community/lms-plugin-repository/issues
- **This Plugin Issues**: https://github.com/mtxmiller/MobileTranscode/issues

---

**Last Updated**: October 2025
**Current Version**: 1.0.0
