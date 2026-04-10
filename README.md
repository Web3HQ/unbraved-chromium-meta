# Unbraved Chromium Meta

A privacy-focused, fully customizable Chromium derivative inspired by **Ungoogled Chromium**, but out of Brave Browser. This project removes Brave-specific proprietary features (Sync servers, accounts, AI, wallet infrastructure, rewards) and replaces them with extensible, configurable alternatives that put **you in control**.

## About Unbraved Chromium Meta

**Unbraved Chromium** is not just a fork of Brave Browser—it's a reimagining. We take the excellent privacy features and rendering engine from Chromium, strip away Brave's proprietary infrastructure, and create a foundation for truly **customizable privacy**.

Unlike Brave Browser (which integrates its own sync servers, account systems, and cloud services), Unbraved Chromium gives you:

- ✅ **Core Privacy Features** — Ad blocking, tracker blocking, fingerprinting protection, HTTPS upgrade
- ✅ **No Proprietary Lock-In** — No Brave accounts, sync servers, or forced integrations
- ✅ **Customizable Services** — Plug in your own sync (Nextcloud, WebDAV), AI (Ollama, local), wallets (any RPC)
- ✅ **Full Transparency** — Know exactly what's running and where your data goes
- ✅ **Community-Driven** — Built by and for privacy-conscious developers

## Key Removals & Replacements

| Feature | Brave | Unbraved | |
|---------|-------|----------|---|
| **Sync** | Brave Sync servers | Configurable (Nextcloud, WebDAV, custom) | ✓ Customizable |
| **Accounts** | Brave Accounts required | Optional local auth or custom provider | ✓ Your choice |
| **AI Assistant** | Brave Leo (cloud-based) | Disabled by default; plug in Ollama/local LLM | ✓ Extensible |
| **Crypto Wallet** | Built-in Brave Wallet | Disabled by default; custom RPC endpoints | ✓ Optional |
| **Rewards** | Brave Rewards/BAT token | Removed entirely (incompatible) | ✗ Not available |
| **News Feed** | Brave News | Disabled—use RSS readers instead | ✗ Removed |
| **VPN** | Brave VPN subscription | Removed—use standalone VPN tools | ✗ Removed |
| **Shields** | "Brave Shields" branding | **Rebranded as "Unbraved Shields"**—keeps all privacy tech | ✓ Enhanced |

## Quick Start

This repository contains issue tracking, release notes, and documentation. **Source code** is at [unbraved-chromium](https://github.com/Web3HQ/unbraved-chromium) (sister repository).

### Resources

- 📖 **[Research & Design Document](./unbrave-studyresearchby-claudehaiku45.md)** — Complete analysis of feature removal, architecture, implementation guide
- 📋 **Issues** — Report bugs, request features (once repo is public)
- 📦 **Releases** — Binary downloads and build instructions
- 🛠️ **[Build Documentation](./docs/BUILD.md)** — How to compile from source
- 🔒 **[Security Policy](./docs/SECURITY.md)** — Responsible disclosure guidelines

## Core Features Kept

Our philosophy mirrors **Ungoogled Chromium**: remove Google/Brave tracking and lock-in, keep everything useful.

### Privacy & Security (From Core Chromium)
- **Unbraved Shields** — Procedural cosmetic filtering, tracker blocking, fingerprinting protection
- **HTTPS Upgrade** — Force HTTPS where available
- **Script Blocking** — Control third-party scripts
- **Cookie Management** — Separate 1st-party and 3rd-party cookies
- **Global Privacy Control (GPC)** — Privacy preference signal
- **Sandbox & Exploit Mitigations** — All Chromium security features

### User Control
- **Vertical tabs** — Organize tabs your way
- **Split view** — Side-by-side tab viewing
- **Sidebar** — Customizable extensions/tools
- **Theme Options** — Light, dark, auto, custom
- **Settings Search** — Find settings fast
- **All Chromium extensions** — Full WebExtensions support

### NOT Included
- ❌ Brave Sync & accounts
- ❌ Brave Leo AI (remove or provide extensibility for alternatives)
- ❌ Brave Wallet (or custom RPC configuration only)
- ❌ Brave Rewards
- ❌ Brave News Feed
- ❌ Brave VPN
- ✓ But **everything is configurable** via flags and config files

## Design Philosophy

**Unbraved Chromium Meta** follows these principles:

1. **Privacy First** — No proprietary tracking, sync servers, or forced cloud integrations
2. **User Control** — You decide what services to use and where
3. **Transparency** — All code is open, all configurations are documented
4. **Modularity** — Services can be disabled, replaced, or extended
5. **Sustainability** — Regular Chromium updates, community-maintained

## Building from Source

See [BUILD.md](./docs/BUILD.md) for platform-specific instructions (Windows, macOS, Linux, Android, iOS).

## Configuration

### Service Providers

Configure custom backends for sync, AI, wallets:

```json
// ~/.config/unbraved/services.json
{
  "sync": {
    "provider": "custom",
    "backend": "http://your-nextcloud-server/remote.php/dav/files/"
  },
  "ai": {
    "provider": "ollama",
    "endpoint": "http://localhost:11434"
  },
  "wallet_rpc": {
    "ethereum": "http://your-eth-node:8545",
    "solana": "http://your-solana-node:8899"
  }
}
```

**[Full Configuration Guide](./docs/CONFIGURATION.md)** — Service providers, flags, environment variables

## Contributing

We welcome contributions! See [CONTRIBUTING.md](./CONTRIBUTING.md) for:
- How to report issues
- Code style guidelines
- Build instructions for developers
- Pull request process

## Security

Found a security vulnerability? **Please don't open a public issue.** Instead:

- Email: [your-security-contact@example.com](mailto:security@example.com)
- Follow: [Security Policy](./docs/SECURITY.md)

We take security seriously and will respond promptly.

## License

**Unbraved Chromium** is licensed under the **Mozilla Public License 2.0 (MPL-2.0)**, same as Brave Browser. See [LICENSE](./LICENSE) for details.

Chromium itself is under the BSD license, which is compatible.

## Related Projects

- **[Ungoogled Chromium](https://github.com/ungoogled-software/ungoogled-chromium)** — Our inspiration; removes Google integrations
- **[Librewolf](https://librewolf.net/)** — Privacy-focused Firefox derivative
- **[Tor Browser](https://www.torproject.org/)** — Privacy and anonymity
- **[Nextcloud](https://nextcloud.com/)** — Self-hosted sync & files (potential Sync replacement)
- **[Ollama](https://ollama.ai/)** — Local LLM runner (potential AI replacement)

## Acknowledgments

- **Brave Software** — Excellent privacy features and codebase as foundation
- **Chromium Project** — World-class browser engine
- **Ungoogled Chromium** — Philosophy and patterns for feature removal
- **Community** — Your contributions and feedback drive improvement

## Questions?

- 💬 **Discussions** — General questions and ideas
- 🐛 **Issues** — Bug reports and feature requests
- 📧 **Email** — [your-contact@example.com](mailto:contact@example.com)

---

**Status**: Research Phase Complete | **Roadmap**: [ROADMAP.md](./ROADMAP.md) | **Latest Release**: [Releases](./releases)

*Last Updated: April 10, 2026*
