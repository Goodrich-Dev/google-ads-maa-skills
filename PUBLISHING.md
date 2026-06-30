# Publishing this repo to GitHub

Goal: put this folder on GitHub as a **public** repo named
`google-ads-maa-skills` so others can install it. (Public is required for the
one-line plugin install to work.)

## Recommended: GitHub CLI — one-time setup, then one command

1. Install GitHub CLI: <https://cli.github.com> (on a Mac: `brew install gh`).
2. Sign in: `gh auth login` (choose GitHub.com → HTTPS → log in with a browser).
3. From inside this folder, run:

   ```
   gh repo create google-ads-maa-skills --public --source=. --remote=origin --push \
     --description "Claude skills for Google Ads MAA reporting for local service businesses"
   ```

   That creates the repo and pushes everything. Done.

## Alternative: plain git + a personal access token

1. Create an empty repo at <https://github.com/new> named
   `google-ads-maa-skills`, set **Public**, and do NOT add a README, license, or
   .gitignore (this repo already has them).
2. Create a token: GitHub → Settings → Developer settings → Personal access
   tokens → Fine-grained tokens → grant Contents read/write on the new repo.
3. In Terminal, from inside this folder:

   ```
   git init && git add -A && git commit -m "Initial release: Google Ads MAA skills v1.0.0"
   git branch -M main
   git remote add origin https://github.com/<your-username>/google-ads-maa-skills.git
   git push -u origin main
   ```

   Paste the token as the password when prompted.

## Last resort: the web uploader (no terminal)

1. Create the repo at <https://github.com/new> (Public).
2. On the new repo page, click **uploading an existing file**, then drag in the
   CONTENTS of this folder.
3. **Important caveat:** the browser uploader often skips the hidden
   `.claude-plugin` folder, which the plugin REQUIRES. After uploading, confirm
   that `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` are in
   the repo. If they're missing, add them manually (Add file → Create new file →
   type `.claude-plugin/plugin.json` and paste the contents).

## After it's live: how people install it

Share this URL: `https://github.com/<your-username>/google-ads-maa-skills`

```
/plugin marketplace add <your-username>/google-ads-maa-skills
/plugin install google-ads-maa-skills
```

Or they can copy the folders under `skills/` into their own `.claude/skills/`.

## Making updates later

Edit files, then:

```
git add -A && git commit -m "Describe the change" && git push
```
