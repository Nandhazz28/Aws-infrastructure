# Linux Permissions and ACLs

Evidence: [`linux-users-groups-acl.png`](../screenshots/ec2-linux/linux-users-groups-acl.png). Context: [documentation/05](../documentation/05-linux-administration.md).

## Verified state of `/opt/cloud-project`

```text
$ ls -ld /opt/cloud-project
drwxrws---+ 2 root cloudteam 4096 Oct 31 15:08 /opt/cloud-project

$ getfacl /opt/cloud-project
# file: opt/cloud-project
# owner: root
# group: cloudteam
# flags: -s-
user::rwx
user:tester:r-x
group::rwx
mask::rwx
other::---
```

(Transcribed from the screenshot. `getfacl` prints a notice that it is removing the leading `/` from the absolute path; that is normal.)

## Reading the output

| Element | Meaning |
|---|---|
| `d` | Directory |
| `rwx` (owner) | `root` has full access |
| `rws` (group) | Group `cloudteam` has read, write, execute, and the **setgid** bit is set. With ACLs present this field shows the mask |
| `---` (other) | No access for anyone else |
| `+` | An extended ACL is present |
| `flags: -s-` | setgid set; setuid and sticky not set |
| `user:tester:r-x` | `tester` can list the directory and enter it, but cannot create, delete or rename files |
| `mask::rwx` | Upper limit on named-user and group entries; it does not reduce anything here |

The octal mode is `2770` (setgid plus `rwxrwx---`), inferred from the symbolic output.

## Why setgid

New files and subdirectories created inside inherit group `cloudteam` instead of the creator's primary group. Collaborators therefore do not have to fix ownership manually.

## Illustrative commands (the setup commands were not captured)

```bash
sudo apt install acl                              # captured in the screenshot
sudo mkdir -p /opt/cloud-project
sudo chown root:cloudteam /opt/cloud-project
sudo chmod 2770 /opt/cloud-project
sudo setfacl -m u:tester:rx /opt/cloud-project
getfacl /opt/cloud-project
```

## Suggested verification (not yet captured)

```bash
sudo -u tester ls /opt/cloud-project              # expected: succeeds
sudo -u tester touch /opt/cloud-project/x         # expected: Permission denied
sudo -u devops touch /opt/cloud-project/y         # expected: succeeds, file group is cloudteam
```

## Notes

- Add a default ACL (`setfacl -d -m u:tester:rx ...`) if `tester` must also read files created later; the current evidence shows only an access ACL, so new files may not grant `tester` access.
- Changing mode bits with `chmod` on a directory with an ACL changes the mask. Check `getfacl` afterwards.
- Avoid `chmod 777`. It is almost never the correct fix.
