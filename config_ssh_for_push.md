# Push

1. Create SSH Keys
```bash
ssh-keygen -t ed25519 -C "comment"
```

2. Copy public-key to Root user in gitlab:

3. Test ssh:

```bash
ssh -T git@git.rahbia.iamsre.ir -p 1234
```

4. Pepare repository:
   
```bash
git config --global user.email "email@gmail.com"
git config --global user.name "milad"
cd /Project/Path
git init
git add .
git commit -m "comment"
git remote add origin "ssh://git@git.rahbia.iamsre.ir:1234/iamsre/blue-green.git"
git push --set-upstream origin master
```
