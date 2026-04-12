# Operability

Only raise operability findings when a real production failure mode is made harder to detect, diagnose, or recover from. Do not flag the absence of observability on trivial or internal-only code.

## What to look for
- New failure modes with no way for an operator to detect them — no log, metric, trace, or alert
- Risky changes landing without a feature flag, kill switch, or safe rollout path
- Rollout ordering hazards: schema, config, or API changes incompatible with the previous version during deploy
- Log lines and error messages missing the context needed to diagnose a real incident
- Config, secret, or infrastructure requirements introduced without a matching deploy or migration plan
- Behavior changes that silently invalidate existing dashboards, SLOs, or alerts

## Questions to ask
- When this breaks in production, what signal will tell an operator that it broke?
- Do errors at the failure point carry enough context — ids, inputs, state — to act on?
- Can this change be turned off or rolled back without another code deploy?
- Is the previous version of the system still compatible with the new data, config, or API during rollout?
- Are new external calls, timeouts, and retries visible to monitoring?

## Common patterns to flag
- Swallowed or vaguely-logged errors on newly added external calls
- New background jobs, consumers, or schedulers with no health signal
- Unbounded retries or backoff without metrics or circuit breakers
- Schema migrations that assume writers and readers are on the same version
- Feature rollouts gated only by deploy, with no flag or staged rollout
- Changes that require coordinated config or infrastructure updates but document neither

## Context to gather
- Check how nearby code exposes logs, metrics, and traces
- Look for existing feature-flag, config, or rollout patterns in the repository
- Check deploy documentation for migration or rollout expectations
- Name the specific failure mode and the signal gap when raising a concern
