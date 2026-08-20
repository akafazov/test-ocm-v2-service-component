# Test OCM v2 Service Component Direct Publish

This is a test repo to verify the fix for OCM v2 service component resolution.

## Problem

OCM v2's `add component-versions` resolves all `componentReferences` during graph discovery. When publishing to a local CTF, external component references cannot be resolved (they don't exist in the local archive).

## Solution

Publish directly to `ghcr.io/platform-mesh` instead of a local CTF. This allows OCM to resolve external component references from the remote registry where they actually live.

## Test

Run the workflow via GitHub Actions:

```
gh workflow run test-service-component.yml -f chartVersion=0.1.0 -f appVersion=v0.1.0
```

The workflow will:
1. Create a test component with a reference to an existing external component
2. Publish directly to `ghcr.io/platform-mesh` using OCM v2
3. Verify OCM can resolve the external reference during graph discovery

**Expected result:** Success (no "component version not found" errors)

## Difference from helm-charts ocm-service-component.yaml

**Before (broken):**
```yaml
ocm_ctf=ctf::transport.ctf
./ocm add component-versions --repository "$ocm_ctf" --constructor "$constructor_file"
# Then transfer to ghcr.io
```

**After (fixed):**
```yaml
./ocm add component-versions --repository "ghcr.io/platform-mesh" --constructor "$constructor_file"
# Direct publish, no transfer needed
```

This matches the pattern already used in `ocm-aggregator.yaml` which works fine.
