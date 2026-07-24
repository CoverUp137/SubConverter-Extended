# Ruleset options

SubConverter-Extended supports optional ruleset behavior flags without changing
the traditional three-field INI layout.

## `no-resolve`

For a native Clash/Mihomo IP-CIDR rule provider, append `|no-resolve` to the
update interval:

```ini
; SubConverter-Extended extension:
; append |no-resolve after interval for clash-ipcidr
; traditional subconverter preserves URL and interval but ignores this option
ruleset=🎯 全球直连,clash-ipcidr:https://example.com/cn-ip.yaml,28800|no-resolve
```

SubConverter-Extended then emits the option on the provider reference:

```yaml
rules:
  - RULE-SET,cn-ip,🎯 全球直连,no-resolve
```

The provider remains a normal `behavior: ipcidr` provider. Its URL, path,
interval, and remote `payload:` are not modified.

Do not add a fourth comma-separated field:

```ini
# Wrong: breaks traditional subconverter parsing
ruleset=🎯 全球直连,clash-ipcidr:https://example.com/cn-ip.yaml,28800,no-resolve
```

The pipe suffix is intentionally compatible with traditional subconverter:
its `std::atoi`-style interval parser reads `28800|no-resolve` as `28800`,
preserves the URL, and safely ignores the extension.

The option is case-insensitive. Empty and duplicate options are ignored.
Unknown options produce a warning and are ignored.

## TOML and YAML

TOML rulesets use an options array:

```toml
[[rulesets]]
group = "🎯 全球直连"
type = "clash-ipcidr"
ruleset = "https://example.com/cn-ip.yaml"
interval = 28800
options = ["no-resolve"]
```

YAML rulesets use the equivalent sequence:

```yaml
rulesets:
  rulesets:
    - group: 🎯 全球直连
      ruleset: clash-ipcidr:https://example.com/cn-ip.yaml
      interval: 28800
      options:
        - no-resolve
```

All three formats normalize to the same internal options model.

## Scope and safe fallback

`no-resolve` currently applies only to non-Script Clash/Mihomo output for
`clash-ipcidr` rulesets:

- Direct provider mode emits one `no-resolve` on the `RULE-SET` reference.
- Expanded mode emits one `no-resolve` on each `IP-CIDR` and `IP-CIDR6` rule.
- Existing `no-resolve` suffixes are not duplicated.
- `clash-domain`, `clash-classic`, Clash Script mode, and non-Clash targets
  safely ignore the option and continue conversion with a warning.

Configurations that do not declare any options retain their previous output.
