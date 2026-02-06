# FFmpeg x264 pkg-config Detection Fix

## Latest Issue (Resolved)
After previous fixes, FFmpeg was still failing with:
```
ERROR: x264 not found using pkg-config
```

## Root Cause Analysis

### Problem 1: Incorrect PKG_CONFIG Variable
The previous fix set `PKG_CONFIG="pkg-config --static"` which was incorrect because:
- The `PKG_CONFIG` environment variable should point to the pkg-config executable binary
- It should NOT include command-line arguments
- When set to `"pkg-config --static"`, the system tries to execute a file literally named "pkg-config --static"

### Problem 2: PKG_CONFIG_PATH vs PKG_CONFIG_LIBDIR
Using `PKG_CONFIG_PATH` in cross-compilation scenarios can cause issues:
- `PKG_CONFIG_PATH` **adds** directories to the default search path
- This means pkg-config still searches system directories first
- In cross-compilation, this can cause it to find native system libraries instead of cross-compiled ones

## Solution

### Changed Environment Variable
```yaml
# Before (incorrect for cross-compilation)
PKG_CONFIG_PATH=${{github.workspace}}/x264-install/lib/pkgconfig PKG_CONFIG="pkg-config --static"

# After (correct)
PKG_CONFIG_LIBDIR=${{github.workspace}}/x264-install/lib/pkgconfig
```

### Why PKG_CONFIG_LIBDIR?
- `PKG_CONFIG_LIBDIR` **replaces** the default search path entirely
- pkg-config will ONLY look in the specified directory
- Perfect for cross-compilation: ensures only cross-compiled libraries are found
- Prevents accidental linkage against system libraries

## Technical Details

### Environment Variables for pkg-config:
1. **PKG_CONFIG**: Path to the pkg-config executable (e.g., `/usr/bin/pkg-config` or `i686-w64-mingw32-pkg-config`)
2. **PKG_CONFIG_PATH**: Adds directories to search path (supplements defaults)
3. **PKG_CONFIG_LIBDIR**: Replaces the default search path (for cross-compilation)

### Cross-Compilation Best Practices:
- Always use `PKG_CONFIG_LIBDIR` when cross-compiling
- Never include arguments in the `PKG_CONFIG` variable
- Ensure .pc files in the LIBDIR directory have correct paths

## Expected Outcome
- ✅ pkg-config exclusively searches our x264-install directory
- ✅ No interference from system x264 packages
- ✅ FFmpeg configure successfully finds x264
- ✅ Correct cross-compiled x264 library is used

## References
- pkg-config Guide: https://people.freedesktop.org/~dbn/pkg-config-guide.html
- Cross-compilation with pkg-config: Common practice is to set PKG_CONFIG_LIBDIR to the sysroot's pkgconfig directory

