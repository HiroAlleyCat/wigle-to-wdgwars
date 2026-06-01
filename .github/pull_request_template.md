<!--
Reviewer's verification checklist. Fill out each box before merging.
Reasoning: today's pattern was shipping fixes that surfaced new issues
the moment a user touched the change. Slow-down at merge time prevents
the next round of that.
-->

## Summary

<!-- 1-3 sentences describing the change and its motivation. -->

## What changed (user-facing)

<!-- What an end user notices, if anything. New flag? New default?
"Internal only" is a valid answer for refactors. -->

## Verification

- [ ] `python -c "import wigle_to_wdgwars"` succeeds in a fresh venv.
- [ ] `python wigle_to_wdgwars.py --help` runs clean.
- [ ] If the change touches the upload path: live-tested with a sacrificial key, OR with `--dry-run`.
- [ ] If the change touches install instructions in README: ran them verbatim on a fresh clone (no leftover venv).
- [ ] CHANGELOG.md has an entry for this change.
- [ ] `__version__` in `wigle_to_wdgwars.py` is bumped if user-visible behavior changed.
- [ ] No `Co-Authored-By: Claude` trailer in any commit (per public-repo convention).
- [ ] No `zhn*` hostnames, real names, or lab-internal references in code/commits/README.

## Notes for reviewer

<!-- Anything that needs context, links to related PRs, links to user
reports that motivated this, etc. -->
