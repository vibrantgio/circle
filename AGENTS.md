# AGENTS.md — circle

A circle drawn as four cubic Bézier segments: `CirclePath` returns the
`clip.PathSpec`, `FillCircle` paints it, and `Circle` wraps it as a
`layout.Widget`. The Bézier constant is Spencer Mortensen's
0.5519150244935105, not the 0.55228475 usually quoted.

**Layer.** Tier 0 of ADR-001's table — a leaf whose only dependency outside
the organization is Gio. Its root module imports nothing else in the
organization. That direction is measured rather than typed —
`scripts/check-layers.sh --edges` reports the graph and
`scripts/sync-agents.sh` renders these sentences from it — so correcting
them here changes nothing. The other direction is measured too and
deliberately not written down: the gate checks the graph both ways, but a
public API's consumers are unknowable, so this file says what its module
needs and never who needs it.

**Read the canonical guide before you write code against this module.** It is
the organization's one agent guide — the module inventory with current tags,
the application skeleton, the MVU loop and rx semantics, typography, and the
pitfalls that are not guessable. It lives exactly once, in `vibrantgio/workbench` —
the repository that showcases building applications with Vibrant Gio —
and this file links it rather than copying it:

    https://raw.githubusercontent.com/vibrantgio/workbench/master/llms.txt

**Module.** `github.com/vibrantgio/circle`, one module at the repository
root.

**Build and test.** From the repository root:

    go build ./... && go test ./...
