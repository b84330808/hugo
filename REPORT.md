# Hugo PR report: `isset` with a negative index

## Branch

- Fork branch: `b84330808/hugo:claude/determined-faraday-36csur`
- Commit: `f2e6c83d9d8a347f649d3b19e53e91aab2a2c6f8`
- Base: `gohugoio/hugo` `master` at `a1668da`
- Author: `b84330808 <b84330808@gmail.com>`, with trailer `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`
- Diff: 1 line changed in `tpl/collections/collections.go` and 2 table cases added in `tpl/collections/collections_test.go`

## PR title

```
tpl/collections: Fix isset with negative index
```

## PR body

```
`isset` returned `true` for any negative index into a slice or array, because
it only checked `av.Len() > k`:

    {{ isset (slice 1 2 3) -1 }} → true (want false)

The documentation says `isset` "Reports whether a specific key or index exists
in the given map or slice", and `index (slice 1 2 3) -1` is an error. This
change adds the missing `k >= 0` check and two table cases to `TestIsSet`.

I used Claude Code to write this change.
```

## Fail before (fix reverted, new test cases kept)

`go test ./tpl/collections/ -run '^TestIsSet$' -count=1`

```
--- FAIL: TestIsSet (0.01s)
    collections_test.go:473: 
        error:
          values are not equal
        comment:
          [4] {[1 2 3 5] -1 false false}
        got:
          bool(true)
        want:
          bool(false)
        stack:
          /home/user/hugo/tpl/collections/collections_test.go:473
            c.Assert(result, qt.Equals, test.expect, errMsg)
        
FAIL
FAIL	github.com/gohugoio/hugo/tpl/collections	0.053s
FAIL
```

## Pass after

`go test ./tpl/collections/ -run '^TestIsSet$' -count=1 -v`

```
=== RUN   TestIsSet
=== PAUSE TestIsSet
=== CONT  TestIsSet
WARN  calling IsSet with unsupported type "struct" (time.Time) for key Day will always return false.
WARN  calling IsSet with unsupported type "invalid" (<nil>) for key nil will always return false.
--- PASS: TestIsSet (0.00s)
PASS
ok  	github.com/gohugoio/hugo/tpl/collections	0.039s
```

## Other checks

- `gofmt -l tpl/collections/`: no output
- `go vet ./tpl/collections/`: clean
- `./check.sh ./tpl/collections/...`: all checks passed
- `./check.sh` (all packages): gofmt OK. Two tests fail because they need the network, and they fail the same way on clean `upstream/master`: `TestGetRemoteHead` (`resources/resource_factories/create`, `lookup gohugo.io ... no such host`) and `TestVimeoShortcode` (`tpl/tplimpl`, where `vimeo_simple` calls `resources.GetRemote` on vimeo.com). Every other package passes. This includes the packages that `-failfast` skipped, which were re-run by hand.

## Duplicate search (gohugoio/hugo, all states)

No open or closed issue or PR covers negative indexes in `isset`.

PR searches, all states:

- `repo:gohugoio/hugo isset in:title`, 8 results, all closed, none about index bounds:
  - gohugoio/hugo#15061 tpl/collections: Include key in IsSet unsupported-type warning
  - gohugoio/hugo#14931 tpl/collections: Include template path in IsSet unsupported type warning
  - gohugoio/hugo#8668 Make isset warning message more actionable
  - gohugoio/hugo#5278 tpl: Cast IsSet key to int for indexed types
  - gohugoio/hugo#4580 add a check in IsSet function which panics if array is passed and key is not int
  - gohugoio/hugo#3752 calling isSet on array with key type string no longer panics but retu…
  - gohugoio/hugo#3480 tpl/collections: Log an error on unsupported types in IsSet
  - gohugoio/hugo#3093 tplimpl: return an error on unsupported type in isSet
- `repo:gohugoio/hugo IsSet`, 14 results: the 8 above plus unrelated PRs (gohugoio/hugo#13644, gohugoio/hugo#2686, gohugoio/hugo#2687, gohugoio/hugo#3412, gohugoio/hugo#8305, gohugoio/hugo#1734)
- `repo:gohugoio/hugo IsSet negative`: 0 results
- `repo:gohugoio/hugo isset slice index`: 0 results
- `repo:gohugoio/hugo negative index`: gohugoio/hugo#14154, gohugoio/hugo#11688 and gohugoio/hugo#10177, all unrelated
- `repo:gohugoio/hugo is:open tpl/collections`: only gohugoio/hugo#15501 (merge of maps), which is unrelated

Issue searches, all states:

- "isset negative index slice returns true": gohugoio/hugo#9627 (Position slice bounds), which is unrelated
- "isset function slice index out of range negative key": gohugoio/hugo#9627, which is unrelated
- "collections.IsSet wrong result": 0 results
- "isset returns true for index that does not exist": 0 results
- "isset": gohugoio/hugo#14580 (getJSON), which is unrelated

`git log -S'av.Len() > k' upstream/master -- tpl/` shows no fix on master. The clone is shallow, with 60 commits.

## Policy check

The upstream `CONTRIBUTING.md` still allows AI-assisted PRs from non-maintainers when the scope is narrow, such as a bug fix, and the AI use is disclosed. Upstream changed `CONTRIBUTING.md` since the fork's master, but only to remove the LibSASS and `extended` build notes. `AGENTS.md` and `SECURITY.md` are unchanged.

Because this is a trivial fix, it has no linked issue. Per `CONTRIBUTING.md`, a non-trivial change needs one.
