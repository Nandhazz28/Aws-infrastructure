# Linux Users and Groups

Evidence: [`linux-users-groups-acl.png`](../screenshots/ec2-linux/linux-users-groups-acl.png), [`linux-acl-install-and-user-creation.png`](../screenshots/ec2-linux/linux-acl-install-and-user-creation.png). Context: [documentation/05](../documentation/05-linux-administration.md).

## Verified state

Output of `getent passwd devops tester ubuntu` and `getent group cloudteam` (the `devops` comment field contained a real name and was redacted in the published screenshot):

| User | UID | Primary GID | Home | Shell |
|---|---|---|---|---|
| `ubuntu` | 1000 | 1000 | `/home/ubuntu` | `/bin/bash` |
| `devops` | 1001 | 1001 | `/home/devops` | `/bin/bash` |
| `tester` | 1003 | 1003 | `/home/tester` | `/bin/bash` |

| Group | GID | Members |
|---|---|---|
| `cloudteam` | 1002 | `devops`, `ubuntu` |

`tester` is deliberately **not** a member of `cloudteam`. Its access comes only from an ACL entry (see [permissions-and-acls.md](permissions-and-acls.md)).

The screenshot shows `sudo adduser tester` selecting UID/GID from the 1000-59999 range and creating group `tester` (1003).

## Design intent

- `ubuntu` is the default cloud-image administrator account.
- `devops` represents a named administrator who works with the shared project files.
- `tester` represents a restricted account with read-only needs.
- `cloudteam` is the shared group for people who may modify project files.

## Illustrative commands

The creation commands for `devops` and `cloudteam` were not captured. Equivalent commands, for reference only:

```bash
sudo adduser devops
sudo addgroup cloudteam
sudo usermod -aG cloudteam devops
sudo usermod -aG cloudteam ubuntu
sudo adduser tester            # captured in the screenshot
```

## Good practice

- Use neutral, non-personal values in account comment fields when documenting publicly.
- Prefer SSH key authentication; avoid shared passwords.
- Review `sudo` rights separately; membership of `cloudteam` grants no `sudo` access.
- Remove accounts that are no longer needed (`sudo deluser --remove-home <user>`).
