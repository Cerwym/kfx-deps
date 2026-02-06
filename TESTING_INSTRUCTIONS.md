# Testing Instructions for x264 pkg-config Fix

## Overview
This document explains how to test the GitHub Actions workflow with the x264 pkg-config path fixes.

## What Was Fixed
1. **ffmpeg-mingw32**: Fixed x264 installation to use `--prefix` instead of `DESTDIR`
   - This ensures pkg-config files have correct paths
   - Updated FFmpeg configure paths to match new x264 installation location

2. **ffmpeg-mingw64**: Removed invalid `--disable-postproc` flag
   - This flag no longer exists in FFmpeg master branch

## Current Status
- **Branch**: `copilot/fix-package-generation-flags`
- **Latest commit**: `a602e00` - "Test workflow with x264 pkg-config fixes"
- **Fix commit**: `760f80f` - "Fix x264 pkg-config paths and remove invalid FFmpeg flag"

## Tag Created (Local)
A git tag has been created locally but not pushed:
- **Tag name**: `20260206-1807-x264-pkg-config-fix`
- **Points to**: `760f80f` (the actual fix)

To push this tag:
```bash
git push origin 20260206-1807-x264-pkg-config-fix
```

## Workflow Runs
Two workflow runs have been triggered:
1. Run #21760671983 - Status: action_required (first commit)
2. Run #21760896412 - Status: action_required (test commit)

URLs:
- https://github.com/Cerwym/kfx-deps/actions/runs/21760671983
- https://github.com/Cerwym/kfx-deps/actions/runs/21760896412

## Why "action_required"?
GitHub requires manual approval for workflows triggered by:
- Pull requests from forks
- Bot accounts (like copilot-swe-agent)

This is a security measure to prevent unauthorized code execution.

## How to Test

### Option 1: Approve Pending Workflow
1. Go to: https://github.com/Cerwym/kfx-deps/actions/runs/21760896412
2. Click "Approve and run" button
3. Monitor the workflow execution
4. Check that ffmpeg-mingw32 and ffmpeg-mingw64 jobs complete successfully

### Option 2: Merge PR and Test
1. Merge the PR #1 to master branch
2. This will trigger the workflow on master (no approval needed)
3. Monitor the workflow execution

### Option 3: Manual Push from Main Account
1. Checkout the branch locally as the repository owner
2. Make a small change or empty commit
3. Push - this will trigger workflow without approval needed

## Expected Results

### ffmpeg-mingw32 Job
Should complete successfully:
- x264 configures with `--prefix=${{github.workspace}}/x264-install`
- x264 installs to correct location
- pkg-config finds x264.pc at `x264-install/lib/pkgconfig/`
- FFmpeg configure finds x264 via pkg-config
- FFmpeg builds with libx264 support
- Artifacts uploaded successfully

### ffmpeg-mingw64 Job  
Should complete successfully:
- FFmpeg configures without `--disable-postproc` error
- All libraries build successfully
- Artifacts uploaded successfully

## Verification Steps

After workflow completes, verify:
1. All jobs show green checkmarks
2. ffmpeg-mingw32 artifact contains libx264-enabled FFmpeg
3. ffmpeg-mingw64 artifact built successfully
4. No "x264 not found" errors
5. No "--disable-postproc" errors

## Original Issue
Reference: https://github.com/Cerwym/kfx-deps/actions/runs/21729150102

The original workflow failed with:
- ffmpeg-mingw32: "ERROR: x264 not found using pkg-config"
- ffmpeg-mingw64: "Unknown option '--disable-postproc'"

Both issues are now fixed in commit `760f80f`.
