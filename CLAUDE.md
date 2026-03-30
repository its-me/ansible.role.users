# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Linting

The CI pipeline runs ansible-lint via a GitLab component:
```bash
ansible-lint
```

The `.ansible-lint` config skips the `role-name` rule only.

## Role Architecture

This is an Ansible role for managing system users, groups, SSH keys, and sudo access on Linux hosts.

### Variable Merging System

The role merges user/group lists from four scopes (lowest to highest priority):

| Scope | Variable | Description |
|-------|----------|-------------|
| Global | `users_all` / `users_groups_all` | Applied to all hosts |
| Environment | `users_env` / `users_groups_env` | Environment-specific overrides |
| Host | `users_host` / `users_groups_host` | Host-specific overrides |
| Role call | `users` / `users_groups` | Highest priority (used in `dependencies:`) |

Merging uses Jinja2 `combine` with `recursive=True` keyed on `user`/`group` name — so lower-scope entries are deep-merged into, not replaced by, higher-scope entries.

The merged results land in `_users_envs` and `_users_groups_envs`, then split into `users_add`/`users_groups_add` (after excluding `users__removed`/`users_groups__removed`).

### Task Flow (`tasks/main.yml`)

1. **Sudo setup** — detects sudo version, ensures `/etc/sudoers.d` uses the correct `@includedir` (≥1.9.1) or `#includedir` (<1.9.1) directive
2. **Add groups** — calls `group_with_gid.yml` or `group_without_gid.yml` depending on whether `gid` is set
3. **Add users** — calls `user.yml` per user, which:
   - Sets user facts (comment, shell, home path)
   - Creates primary group
   - Calls `user_with_uid.yml` or `user_without_uid.yml`
   - Adds SSH keys (exclusive mode — removes unlisted keys)
   - Writes `/etc/sudoers.d/95-{username}` if user is in `users__admins`
4. **Remove users/groups** — removes home dirs, deletes sudoers files, then validates removed users are absent from all sudoers files (fails if found)

### Key Behaviors

- Passwords are hashed with SHA512 via the `password_hash` filter; `update_password` defaults to `on_create`
- SSH keys use `exclusive: true` — any key not listed will be removed
- Sudo grant is passwordless (`NOPASSWD: ALL`) via `/etc/sudoers.d/95-{username}`
- The role depends on the `apt` role to install `sudo` (configurable via `users_packages`)
