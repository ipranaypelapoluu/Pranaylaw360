# Pranay Law 360 — TG LAWCET 2026 Predictor Deployment

This package separates the public predictor from the private candidate database.

## Architecture

- **GitHub Pages**: public `public/index.html` website.
- **Vercel Function**: `api/save-prediction.js` receives prediction submissions.
- **Private GitHub repository**: stores `data/candidates.xlsx`.
- The GitHub token is a **server-side Vercel environment variable** and is never placed in the public HTML.

## 1. Create two GitHub repositories

### Public website repository

Create a public repository, e.g.

`Pranay-Law-360-TGLAWCET-Predictor`

Push the contents of `public/` to the repository root as `index.html` and `pranay-law-360-logo.png`.

Enable GitHub Pages using **GitHub Actions**.

### Private data repository

Create a separate **private** repository, e.g.

`Pranay-Law-360-Private-Data`

The API will create/update:

`data/candidates.xlsx`

The workbook has three sheets:

- `Candidates` — one current profile per mobile number.
- `PredictionHistory` — every successful prediction event.
- `Preferences` — the college preference rows generated for every prediction.

Do not publish this repository or place the workbook in the public website repository.

## 2. Create a GitHub token

Create a fine-grained GitHub Personal Access Token for the private data repository with only the repository **Contents: Read and Write** permission.

Do not put the token in `index.html`, localStorage, a browser environment variable, or a URL.

## 3. Deploy the API to Vercel

From this folder:

```bash
npm install
npx vercel login
npx vercel
```

In Vercel Project Settings → Environment Variables add:

```text
GITHUB_TOKEN=...
GITHUB_OWNER=your-github-username
GITHUB_REPO=Pranay-Law-360-Private-Data
GITHUB_BRANCH=main
GITHUB_WORKBOOK_PATH=data/candidates.xlsx
ALLOWED_ORIGIN=https://your-username.github.io
```

Deploy to production:

```bash
npx vercel --prod
```

The endpoint will be:

`https://YOUR-VERCEL-PROJECT.vercel.app/api/save-prediction`

## 4. Configure the public HTML

Open `public/index.html` and set the API URL near the configuration block:

```javascript
window.PRANAY_API_URL = 'https://YOUR-VERCEL-PROJECT.vercel.app/api/save-prediction';
```

Then publish the public repository through GitHub Pages.

## 5. Data privacy behavior

The public user can submit a prediction, but cannot see the private Excel workbook. The browser never receives the GitHub token. The API receives the candidate data, validates the mandatory 10-digit mobile number and consent, then updates the private workbook through the GitHub API.

When the same mobile number predicts again, the `Candidates` sheet is updated and its `Prediction Count` increases. A new row is added to `PredictionHistory` and the new preference list is appended to `Preferences`.

## 6. Important operational notes

- Keep the data repository private.
- Use a token with the minimum repository permission needed.
- Do not collect Aadhaar, passwords, bank information, or other sensitive data.
- Because this system stores mobile numbers, publish a clear privacy notice/consent statement appropriate for your use case.
- The workbook is rewritten on every successful submission. For a very large database, migrate the backend to a real database rather than continuously rewriting an Excel file.
- The API includes a short retry for GitHub 409 conflicts so concurrent submissions are less likely to overwrite each other.
- The prediction dataset embedded in the HTML should be treated as the source configured by the channel operator; it is not an official allotment guarantee.
