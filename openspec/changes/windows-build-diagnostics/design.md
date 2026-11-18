## Approach

Use the pinned CLI's fingerprint and the exact symbol from the failing link.
For revisions with a release-specific names header, compare the selected body
with the definition and object in the failed build. Older wizard releases
can supply a raw executable hash.

## Validation

Compare the three names with the reference and PAL release maps. Check the
PAL fingerprint and expected name expansion, exercise the PowerShell search
and verify that the guide's links resolve.
