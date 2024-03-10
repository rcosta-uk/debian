# Debian commands

## User

Create a new user with the adduser command that creates the account, a group, and the home directory:
```bash
adduser <name>
```

Add user to ```sudo``` for this, use the usermod command:
```bash
usermod -aG sudo <name>
```
