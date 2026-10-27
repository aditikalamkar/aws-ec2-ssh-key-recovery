# SSH Cheat Sheet — EC2 Key Management

## Generate a key pair (local machine)

```powershell
# Windows PowerShell
ssh-keygen -t ed25519 -C "my-laptop" -f $env:USERPROFILE\.ssh\ec2-key
```

```bash
# Linux / macOS
ssh-keygen -t ed25519 -C "my-laptop" -f ~/.ssh/ec2-key
```

## View the public key (to copy to the server)

```powershell
Get-Content $env:USERPROFILE\.ssh\ec2-key.pub
```

```bash
cat ~/.ssh/ec2-key.pub
```

## Add a public key on the server

```bash
echo "ssh-ed25519 AAAA... my-laptop" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## List / remove authorized keys on the server

```bash
cat -n ~/.ssh/authorized_keys          # list with line numbers
sed -i '2d' ~/.ssh/authorized_keys     # delete line 2 (an old key)
```

## Fix private key permissions

```bash
chmod 400 ~/.ssh/ec2-key               # Linux / macOS
```

```powershell
# Windows — "UNPROTECTED PRIVATE KEY FILE" error fix
icacls $env:USERPROFILE\.ssh\ec2-key /inheritance:r
icacls $env:USERPROFILE\.ssh\ec2-key /grant:r "$($env:USERNAME):(R)"
```

## Connect

```bash
ssh -i ~/.ssh/ec2-key ubuntu@<public-ip>      # Ubuntu AMI
ssh -i ~/.ssh/ec2-key ec2-user@<public-ip>    # Amazon Linux AMI
ssh -v -i ~/.ssh/ec2-key ubuntu@<public-ip>   # verbose — for debugging
```

## Optional: SSH config shortcut

`~/.ssh/config` (Windows: `C:\Users\<you>\.ssh\config`)

```
Host my-ec2
    HostName <public-ip>
    User ubuntu
    IdentityFile ~/.ssh/ec2-key
```

Then just: `ssh my-ec2`

## Where am I?

| Prompt | You are on |
|---|---|
| `PS C:\Users\...>` | Local Windows (PowerShell) |
| `C:\Users\...>` | Local Windows (cmd) |
| `ubuntu@ip-172-...:~$` | The EC2 server |

## Common errors

| Error | Likely cause |
|---|---|
| `Permission denied (publickey)` | Wrong key, wrong user (`ubuntu` vs `ec2-user`), or public key not in `authorized_keys` |
| `UNPROTECTED PRIVATE KEY FILE` | Private key permissions too open |
| `Identity file ... not accessible` | Path doesn't exist on *this* machine — check which machine you're on |
| `Connection timed out` | Security group doesn't allow port 22 from your IP, or wrong IP |
