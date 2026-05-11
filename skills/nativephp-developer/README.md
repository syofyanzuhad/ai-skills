# nativephp-development

> AI agent skill for building native desktop and mobile apps with [NativePHP](https://nativephp.com) and Laravel.

Works with **Claude Code**, **Cursor**, **Windsurf**, **GitHub Copilot**, and any agent that supports the [Agent Skills](https://agentskills.io) format.

---

## What This Skill Does

Teaches your AI agent how to work with NativePHP, covering:

- **Mobile** (iOS & Android) — installation, routing, offline-first SQLite, dev/build commands
- **Desktop** (Electron) — windows, menus, system notifications, clipboard
- **Native API plugins** — Camera, SecureStorage, Biometrics, Geolocation, Push Notifications, Scanner, Network
- **EDGE native UI components** — TopBar, BottomNav, SideNav (v3+)
- **Events** — AppForegrounded, DeepLinkReceived, PushNotificationReceived, etc.
- **Authentication** — token-based with SecureStorage (mobile pattern)
- **Deployment** — App Store, Google Play, Bifrost cloud builds, desktop distributables
- **Anti-patterns** — common mistakes to avoid

## Installation

### Via Laravel Boost

```bash
php artisan boost:add-skill <your-github-username>/nativephp-development
```

### Via Claude Code

```bash
claude install-skill https://github.com/<your-github-username>/nativephp-skill/tree/main/nativephp-development
```

### Manual

Copy the `nativephp-development/` folder into your project at `.ai/skills/nativephp-development/`.

## Repo Structure

```
nativephp-skill/
├── README.md                        ← You are here
└── nativephp-development/
    └── SKILL.md                     ← The skill definition
```

## References

- [NativePHP Mobile Docs](https://nativephp.com/docs/mobile/3/getting-started/introduction)
- [NativePHP Desktop Docs](https://nativephp.com/docs/desktop/2/getting-started/introduction)
- [Bifrost Cloud Builds](https://bifrost.nativephp.com)
- [NativePHP Plugins](https://nativephp.com/plugins)

## License

MIT
