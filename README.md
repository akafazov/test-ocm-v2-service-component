# Test OCM v2 Service Component Direct Publish

This is a test repo to verify the fix for OCM v2 service component resolution as implemented in [platform-mesh/helm-charts#2419](https://github.com/platform-mesh/helm-charts/pull/2419).

## Problem

OCM v2's `add component-versions` resolves all `componentReferences` during graph discovery. When publishing to a local CTF, external component references cannot be resolved (they don't exist in the local archive).

## Solution

Publish directly to a remote registry (e.g., `ghcr.io`) instead of a local CTF. This allows OCM to resolve external component references from the remote registry where they actually live.

## Test

Run the workflow via GitHub Actions:

```
gh workflow run test-service-component.yml -f chartVersion=0.1.0
```

The workflow will:
1. Create a test chart component and publish it to `ghcr.io/akafazov`
2. Create a test service component with a reference to the chart component
3. Publish directly to `ghcr.io/akafazov` using OCM v2
4. Verify both components were created successfully with references resolved

**Expected result:** Success (both components published and verified)

## How It Works

The test creates two components in the same org:
- `github.com/akafazov/test-service-component` - a service component
- `github.com/akafazov/test-chart-component` - a chart component (referenced by the service component)

Both are published directly to the remote registry, allowing OCM to resolve the reference during graph discovery.

## Difference from Previous Approach

**Before (broken):**
```yaml
ocm_ctf=ctf::transport.ctf
./ocm add component-versions --repository "$ocm_ctf" --constructor "$constructor_file"
# Then transfer to ghcr.io (separate step)
```

**After (fixed):**
```yaml
./ocm add component-versions --repository "ghcr.io/akafazov" --constructor "$constructor_file"
# Direct publish, no transfer needed
```

This matches the pattern in [platform-mesh/helm-charts PR #2419](https://github.com/platform-mesh/helm-charts/pull/2419).
