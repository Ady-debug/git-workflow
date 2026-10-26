# Using the repo

Follow the steps below instructions to set up conventional commit checks locally and on pull request.

<a href="https://typicode.github.io/husky/">Husky</a> is used for automatic linting and to commit to the repo so it can be shared across all users.

More information on convential commits can be found <a href="https://www.conventionalcommits.org/en/v1.0.0/#summary">here</a>.

## Step 1: Setup a package.json file

Initialise NPM from the repo root:

```bash
npm init -y
```

Edit the `package.json` file to mark as tooling only:

```json
{
  "name": "SSIS-repo-tooling",
  "private": true,
  "version": "1.0.0"
}
```

## Step 2: Install the tools

```bash
npm install --save-dev husky @commitlint/cli @commitlint/config-convential
```

## Step 3: Initialise Husky

```bash
npx husky init
```
This step adds `"prepare": "husky"` to the `package.json` file (to ensure the setup is automatic for anyone cloning the repo and running npm install) and create a `.husky/` folder with a sample pre-commit hook which should be deleted as not required (run the below command or delete manually)

```bash
rm .husky/pre-commit
```
## Step 4: Add the commit-msg hook

```bash
echo 'npx --no -- commitlint --edit $1' > .husky/commit-msg
```
Git runs this hook before a commit message is saved. `$1` is the path top the temp file holding the message, and commitlint reads it and exits if it doesn't conform - cancelling the commit

## Step 5:

Create `commitlint.config.js` in the repo root

```js
module.exports = {
  extends: [`@commitlint/config-convential`],
  // The below is optional to either if additional specified scopes are not used
  rules: {
    'scope-enum': [1, "always", ['exampleScope1'],['exampleScope2'],['exampleScope3']],
  },
};
```

## Step 6: Commit your changes

Add `node_modules/` to a `.gitignore` file before committing the files created, ensure the convention is followed, e.g.:

```bash
git add .
git commit -m "chore(ci): add commitlint and husky for conventional commits"
```

## Step 7: Test the changes so far

The below should fail

```bash
git commit --allow-empty -m "updated project readme"
```

The below should pass with no warnings

```bash
git commit --allow-empty -m "chore(docs): updated project readme"
```

## Step 8: Setup the GitHub Actions workflow

Create a `.github/workfloes/commitlint.yml` file with the following content:

```yaml
name: Conventional Commits

on:
  pull_request:
    types:[opened, edited, synchronize, reopened]

permissions:
  contents: read

jobs:
  commitlint:
    name: Lint commit messages
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # needs full history to see every commit in the PR

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - run: npm ci

      -name: Lint every commit in the PR # can be dropped if using squash merge
        run: >
          npx commitlint
            --from ${{ github.event.pull_request.base.sha }}
            --to ${{ github.event.pull_request.base.sha }}
            --verbose

pr-title:
  name: Lint pull request title
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - uses: actions/setup-node@v4
      with:
        node-version: 20
        cache: npm

    - run: npm ci

    - name: Lint the pull request title
      env:
        PR_TITLE: ${{ github.event.pull_request.title }}
      run: echo "$PR_TITLE | npx commitlint --verbose"
```

## Step 9: Test the PR title linting

Open a test Pull Request to confirm the check runs

## Step 10: Setup branch protection rulesets (only enforced on GitHub paid accounts)

In your GitHub account, go to **Settings -> Branches(or Rulesets)** and setup a rule for `main` to require **status checks** for **lint commit messages** and **lint pull request** title.

If you only squash merge you can require just the title check and drop the `Lint every commit in the PR` section from step 8. You can also set the default squash message to "Pull request title" under **Settings -> General -> Pull Requests**

## Step 10: Ready to share with users

Following the above setup, any users who clone the repo will have the correct setup for conventional commits locally, as well as on pull request. Ensure all users run `npm install` to ensure the hook fires for them.
