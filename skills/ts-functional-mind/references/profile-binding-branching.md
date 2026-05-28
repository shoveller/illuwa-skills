# Profile/binding branching pattern

Session learning: when documenting or implementing TypeScript config/profile dispatch, prefer explicit registry maps plus guard clauses over `switch`, ternary chains, `else if`, or IIFEs. This matched the user's preferred functional style for Cloudflare sandbox/MCP examples.

## Pattern

Use a narrow union for request/profile names, a narrow union for binding names, and a `Record` checked with `satisfies` so adding a profile forces type errors at the registry boundary.

```ts
type SandboxProfile = "opencode" | "dind";

type SandboxBindingName = "OPENCODE_SANDBOX" | "DIND_SANDBOX";

type SandboxBindings = Pick<Env, SandboxBindingName>;

const sandboxBindingNameByProfile = {
  opencode: "OPENCODE_SANDBOX",
  dind: "DIND_SANDBOX",
} satisfies Record<SandboxProfile, SandboxBindingName>;

const resolveSandboxBindingName = (profile: SandboxProfile): SandboxBindingName => {
  return sandboxBindingNameByProfile[profile];
};

const resolveSandboxBinding = (
  bindings: SandboxBindings,
  profile: SandboxProfile,
) => {
  const bindingName = resolveSandboxBindingName(profile);
  const binding = bindings[bindingName];

  if (binding) {
    return binding;
  }

  throw new Error(`Sandbox binding is not configured: ${bindingName}`);
};
```

For capability aliases, keep the capability predicate small and use early return guard clauses rather than parsing command strings or hiding decisions in expressions.

```ts
type SandboxCapability = "git" | "node" | "docker" | "opencode";

type SandboxProfileRequest = {
  capabilities: SandboxCapability[];
};

const includesCapability = (
  capabilities: readonly SandboxCapability[],
  capability: SandboxCapability,
): boolean => {
  return capabilities.includes(capability);
};

const resolveProfileFromCapabilities = (
  request: SandboxProfileRequest,
): SandboxProfile => {
  if (includesCapability(request.capabilities, "docker")) {
    return "dind";
  }

  return "opencode";
};
```

## Why

- Profile-to-binding conversion stays a pure lookup table.
- Runtime lookup failure is handled with an explicit guard clause.
- New profiles fail typecheck at `SandboxProfile`, `SandboxBindingName`, and the registry map.
- Capability aliases are explicit and testable.
- Avoids brittle behavior like inferring a sandbox profile by parsing `run_command` strings.
