GitHub supports several keywords that automatically link and manage issues from commits or Pull Requests.

# Keywords That Close Issues

These keywords automatically close the issue when the PR is merged into the default branch.

## Supported Closing Keywords

```text id="0x7dz6"
close
closes
closed

fix
fixes
fixed

resolve
resolves
resolved
```

---

# Examples

## In Commit Message

```bash id="5qv3n6"
git commit -m "Added login validation fixes #12"
```

---

## In Pull Request Description

```text id="2h6j5f"
This PR fixes #12
```

When merged:

```text id="slv8nx"
Issue #12 → Automatically Closed
```

---

# Multiple Issues

You can close multiple issues together.

```text id="g7q25v"
Fixes #12
Closes #15
Resolves #18
```

---

# Referencing Without Closing

If you only want to mention an issue but NOT close it:

## Just Use `#`

```text id="jlql3q"
Worked on #12
Related to #15
```

This creates a link only.

Issue remains open.

---

# Other Useful Keywords

## Duplicate Issues

```text id="jccr3t"
Duplicate of #10
```

Marks issue as duplicate.

---

## Related Keywords (No Auto Close)

These are common conventions (not automatic):

```text id="msk1b4"
Related to #12
Part of #14
See #18
Depends on #20
```

Useful for documentation and tracking.

---

# Cross Repository Issues

You can close issues in another repository also.

```text id="a84oqn"
Fixes username/repo#45
```

Example:

```text id="ll20wu"
Fixes raman/project1#12
```

---

# Important Condition

Auto-close works only when:

* PR is merged into:

  * `main`
  * default branch
  * or configured target branch
* Issue and PR are in connected repositories

---

# Real Example Workflow

## Commit

```bash id="r8c0sp"
git commit -m "Implemented payment API fixes #25"
```

---

## Push

```bash id="f1jlwm"
git push origin feature/payment
```

---

## PR Description

```text id="z2onit"
This PR resolves #25
```

After merge:

```text id="6z9s6u"
Issue #25 automatically closed
```

---

# Best Practice

Inside PR description use:

```text id="vl07o5"
Fixes #issue-number
```

instead of commit messages because:

* cleaner history
* easier tracking
* works better with squash merge
* industry standard practice
