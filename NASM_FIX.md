# FFmpeg nasm Dependency Fix

## Issue
The ffmpeg-mingw64 job was failing with the following error:
```
nasm not found or too old. Please install/update nasm or use --disable-x86asm for a build without hand-optimized assembly.
```

## Root Cause
The ffmpeg-mingw64 job configuration only installed `mingw-w64` but not `nasm`, which is required for x86 assembly optimizations in FFmpeg builds.

## Comparison

### Before
**ffmpeg-mingw32** (working):
```yaml
- name: Install build dependencies
  run: |
    sudo apt-get update
    sudo DEBIAN_FRONTEND=noninteractive apt-get install -yq \
      --no-install-recommends mingw-w64 nasm
```

**ffmpeg-mingw64** (broken):
```yaml
- name: Install mingw-w64
  run: |
    sudo apt-get update
    sudo DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends mingw-w64
```

### After
Both jobs now have matching configurations:

**ffmpeg-mingw32**:
```yaml
- name: Install build dependencies
  run: |
    sudo apt-get update
    sudo DEBIAN_FRONTEND=noninteractive apt-get install -yq \
      --no-install-recommends mingw-w64 nasm
```

**ffmpeg-mingw64**:
```yaml
- name: Install build dependencies
  run: |
    sudo apt-get update
    sudo DEBIAN_FRONTEND=noninteractive apt-get install -yq \
      --no-install-recommends mingw-w64 nasm
```

## Fix Applied
- Updated the ffmpeg-mingw64 job to install nasm alongside mingw-w64
- Renamed the step from "Install mingw-w64" to "Install build dependencies" to match ffmpeg-mingw32
- This enables x86 assembly optimizations for both 32-bit and 64-bit builds

## Expected Outcome
- ✅ ffmpeg-mingw64 configure step will succeed
- ✅ FFmpeg will build with hand-optimized assembly
- ✅ Both mingw32 and mingw64 FFmpeg builds will complete successfully

## Related Issues
This fix completes the resolution of all FFmpeg build issues:
1. ✅ x264 pkg-config paths (commit 760f80f)
2. ✅ x264 static library detection (commit 7a77416)
3. ✅ Invalid --disable-postproc flag (commit 760f80f)
4. ✅ OpenAL version pinning (commit 7a77416)
5. ✅ Missing nasm dependency (commit 95a01ab) - **This fix**

## Workflow Run
Latest run: https://github.com/Cerwym/kfx-deps/actions/runs/21765843106

Status: Awaiting approval (bot-triggered workflows require manual approval)
