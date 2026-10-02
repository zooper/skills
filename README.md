# OpenCode Skills

Reusable OpenCode skills.

## Network & Infrastructure Review

Production-focused code review for Terraform, AWS, GCP, Kubernetes/GKE, BGP, IPv4/IPv6, Java, Go, SRE, observability, security, MTU/PMTUD, and failure modes.

### OpenCode V2 HTTP catalog

Add this source to `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": [
    "https://raw.githubusercontent.com/zooper/skills/main/"
  ]
}
```

Then load `network-infra-review` explicitly, or let OpenCode discover it when relevant.

### Direct/raw skill

The conventional skill definition is also available at:

`network-infra-review/SKILL.md`

For a local install, place that directory under `.opencode/skills/` or `~/.config/opencode/skills/`.

## License

MIT
