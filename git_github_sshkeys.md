
setting up git & github using ssh keys ubuntu guide
GitHub SSH: Two Profiles + Short Aliases (Step-by-Step)
This guide uses generic placeholders (no personal info). Replace:

personal@example.com / community@example.com
personal-user / community-user
Aliases p and c (change to whatever you like)
0) Prereqs
bash
sudo apt update && sudo apt install git openssh-client -y
(Optional) Clean slate
rm -f ~/.ssh/id_ed25519*    # only if you want to delete old keys
mkdir -p ~/.ssh && chmod 700 ~/.ssh
Create two SSH keys (one per GitHub account)
Personal key
ssh-keygen -t ed25519 -C "personal@example.com" -f ~/.ssh/id_ed25519_personal

Community key
ssh-keygen -t ed25519 -C "community@example.com" -f ~/.ssh/id_ed25519_community
Press Enter to accept the path; passphrase is optional (adds security).
Start ssh-agent & load keys
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_personal
ssh-add ~/.ssh/id_ed25519_community
ssh-add -l   # verify both keys are listed
Add the public keys to GitHub
Show/copy each public key

cat ~/.ssh/id_ed25519_personal.pub
cat ~/.ssh/id_ed25519_community.pub
On each GitHub account: Settings → SSH and GPG keys → New SSH key → paste → Save.
5) Create short SSH aliases (so clone commands are tiny)

Edit config:

nano ~/.ssh/config
Paste:

# Personal GitHub (alias: p)
Host p
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
  IdentitiesOnly yes

# Community GitHub (alias: c)
Host c
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_community
  IdentitiesOnly yes
You can rename p/c to anything (e.g., hersh / potu).
IdentitiesOnly yes forces SSH to use the specified key (avoids “wrong key” issues).
Test both profiles
ssh -T p   # expect: "Hi personal-user! You've successfully authenticated..."
ssh -T c   # expect: "Hi community-user! You've successfully authenticated..."
Clone using tiny commands
Personal account

git clone p:personal-user/repo-name.git
Community account

git clone c:community-user/repo-name.git
Tip: Respect exact capitalization of user/repo (GitHub treats it as case-sensitive in URLs).
Set commit identity per repo (so commits show the right name)
Inside a personal repo:

git config user.name  "Personal Name"
git config user.email "personal@example.com"
Inside a community repo:

git config user.name  "Community Name"
git config user.email "community@example.com"
(Set once per repo. Skip --global so identities stay separate.)

Optional – auto-identity by folder

~/.gitconfig
[user]
email = personal@example.com
name = Personal Name

[includeIf "gitdir:~/code/community/"]
path = ~/.gitconfig-community

~/.gitconfig-community
[user]
email = community@example.com
name = Community Name

    Put community repos under ~/code/community/ and Git will auto-use the community identity.

9) Daily workflow (quick reference)

git status
git pull            # always pull before starting work
git add .           # or: git add <file>
git commit -m "Message"
git push            # uses the alias saved in this repo’s remote
Check which remote a repo uses:

git remote -v
Troubleshooting
“Permission denied (publickey)”

ssh-add -l                  # see if keys are loaded
eval "$(ssh-agent -s)"      # start agent
ssh-add ~/.ssh/id_ed25519_personal
ssh -T p                    # test personal alias
“Repository moved” or wrong path/case

git remote -v
git remote set-url origin p:personal-user/repo-name.git   # or c:community-user/...
OR use full SSH:
git remote set-url origin git@github.com:personal-user/repo-name.git

Use verbose SSH to debug

ssh -vT p
Safety notes

Never share files without .pub — those are your private keys.

Keep ~/.ssh at 700 and key files at 600 permissions.

Consider passphrases + an agent for better security.

Quick alias rename (optional)

If you prefer different aliases:

Change 'Host p' to 'Host hersh' and 'Host c' to 'Host potu' in ~/.ssh/config
git remote set-url origin hersh:personal-user/repo-name.git
git remote set-url origin potu:community-user/repo-name.git

You’re set! From now on, cloning is as short as:

git clone p:personal-user/repo.git
git clone c:community-user/repo.git
