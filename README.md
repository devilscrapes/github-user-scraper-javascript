<p align="center"><a href="https://apify.com/DevilScrapes/github-user-scraper"><img src="assets/banner.svg" alt="GitHub User Scraper — Profiles, Repos &amp; Orgs — JavaScript example" width="100%"></a></p>

# GitHub User Scraper for JavaScript and Node.js: account and org lookups for your backend

<p align="center">
<a href="https://apify.com/DevilScrapes/github-user-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="JavaScript" src="https://img.shields.io/badge/Node.js-20%2B-339933?logo=nodedotjs&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/github-user-scraper-javascript/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/github-user-scraper-javascript/actions/workflows/ci.yml/badge.svg"></a>
</p>

Use this repository to call the **[GitHub User Scraper](https://apify.com/DevilScrapes/github-user-scraper)** from Node.js, using a single ES module and the `apify-client` package that Apify maintains. Send a list of logins or profile links and await an array of objects, one per account, carrying follower and repository totals, profile links, timestamps and, optionally, a nested array of recently pushed public repositories. Apify runs the lookups and deals with retries, so your Express handler, worker or scheduled function never has to manage GitHub's request budget itself. Rows end up in `results.json` and `results.csv`, or stay in memory when you import the call into your own code.

> **Actor:** [GitHub User Scraper — Profiles, Repos & Orgs](https://apify.com/DevilScrapes/github-user-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in Python:** [github-user-scraper-python](https://github.com/devilscrapes/github-user-scraper-python)

## 🔥 What you get

- **People and organizations in one schema**: both kinds of account come back with identical keys, and `type` says `User` or `Organization` so you can split them later.
- **Paste logins or links**: entries in `usernames` may be bare slugs or full profile URLs; the host part is removed before lookup.
- **Counts as integers**: `followers`, `following`, `public_repos` and `public_gists` are numbers, ready to sort, sum or chart.
- **Recent repositories on request**: set `includeRepos` and each row gains a `repos` list of the account's most recently updated public repositories, each with name, description, language, stars, forks, fork and archived flags and last push time. `maxReposPerUser` caps it at up to 100.
- **Bring a token if you need volume**: without one, GitHub allows 60 requests an hour; an optional personal access token lifts that to 5,000.
- **Back-off built in**: on throttling or server errors the Actor waits and retries, and it reports a clear status message instead of finishing with a silently empty dataset.
- **Only what is public**: `email` is filled only when the account owner chose to publish one, and private repositories or memberships are never touched.

## 🎯 Use cases

### Show live GitHub stats in a product page

Refresh the numbers for your own organization on a timer, cache the row and render follower and repository totals in a footer or an about page. The page stays current without the browser calling GitHub.

### Enrich sign-ups that include a GitHub handle

When a user connects or types a GitHub handle, queue a run for it and store `public_repos`, `followers` and `created_at` next to the account. A support or sales tool can then tell a brand-new account from a long-standing maintainer.

### Post a weekly webhook digest

Run the Actor on an Apify schedule, diff the rows against last week's copy and send the deltas to Slack, Discord or any webhook endpoint. DevRel teams get follower and repository changes for themselves and peers without opening a browser.

### Feed an internal developer directory

For an employee directory or an open-source program office, keep each team account's `blog`, `location` and recent `repos` in your database and serve them through your own API, keyed by `login`.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/github-user-scraper-javascript.git
cd github-user-scraper-javascript
npm install
export APIFY_TOKEN=your_token_here
node index.js                       # uses input.json
node index.js my-input.json --out data --max-charge 2
```

The script starts the [GitHub User Scraper — Profiles, Repos & Orgs](https://apify.com/DevilScrapes/github-user-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Here is the contract in four statements. `call()` resolves once the run has stopped; `listItems()` then returns the dataset rows as plain JavaScript objects.

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_APIFY_TOKEN' });
const run = await client.actor('DevilScrapes/github-user-scraper').call({"usernames": ["apify", "microsoft"], "includeRepos": true, "maxReposPerUser": 5, "proxyConfiguration": {"useApifyProxy": false}});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items);
```

## 📥 Input

`index.js` loads `input.json` and passes it along unchanged. The sample checks two organizations and keeps five recent repositories each, which comes back as two objects in a short run. Only `usernames` is mandatory. In a service, build the input per request and inject `githubToken` from an environment secret instead of writing it into a file.

```json
{
  "usernames": [
    "apify",
    "microsoft"
  ],
  "includeRepos": true,
  "maxReposPerUser": 5,
  "proxyConfiguration": {
    "useApifyProxy": false
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `usernames` | array | yes | `["torvalds", "apify"]` | — |
| `githubToken` | string | no | — | — |
| `includeRepos` | boolean | no | `false` | — |
| `maxReposPerUser` | integer | no | `30` | — |
| `concurrency` | integer | no | `6` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": false}` | — |

## 📤 Output

Expect one object per account. `login`, `type` and `html_url` are always set; other profile properties may be null. Counts are numbers, timestamps are ISO strings, and `repos` is either null or an array of objects with fields such as `full_name`, `language`, `stargazers_count` and `pushed_at`.

| Field | Type | Always present |
|---|---|---|
| `login` | string | yes |
| `type` | string | yes |
| `name` | string or null | no |
| `company` | string or null | no |
| `blog` | string or null | no |
| `location` | string or null | no |
| `email` | string or null | no |
| `bio` | string or null | no |
| `twitter_username` | string or null | no |
| `public_repos` | integer | no |
| `public_gists` | integer | no |
| `followers` | integer | no |
| `following` | integer | no |
| `html_url` | string | yes |
| `avatar_url` | string | no |
| `hireable` | boolean or null | no |
| `created_at` | string | no |
| `updated_at` | string | no |
| `repos` | array or null | no |
| `scraped_at` | string | no |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Result emitted | $0.003 |

Prices as configured on 2026-11-03. The [Actor page](https://apify.com/DevilScrapes/github-user-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in JavaScript

This script compares today's `results.json` with the copy saved last time, reports follower and repository changes per account plus the latest repository push, and then stores the current file for the next comparison. It uses only Node.js built-ins.

```javascript
import { readFile, writeFile } from 'node:fs/promises';

const current = JSON.parse(await readFile('results.json', 'utf8'));
let previous = [];
try {
    previous = JSON.parse(await readFile('previous.json', 'utf8'));
} catch {
    // First run: nothing to compare against yet.
}

const before = new Map(previous.map((acc) => [acc.login, acc]));
const report = current.map((acc) => {
    const old = before.get(acc.login);
    const pushes = (acc.repos ?? []).map((repo) => repo.pushed_at).filter(Boolean).sort();
    return {
        login: acc.login,
        type: acc.type,
        followers: acc.followers,
        followerChange: old ? acc.followers - old.followers : null,
        repoChange: old ? acc.public_repos - old.public_repos : null,
        lastPush: pushes.at(-1) ?? null,
    };
});

console.table(report);
await writeFile('previous.json', JSON.stringify(current, null, 2));
```

## ❓ FAQ

### How many accounts can one request handle?

That depends on GitHub's allowance rather than the Actor. Anonymous access gets 60 requests an hour per address; a token in `githubToken` raises it to 5,000. Each account is one request, and `includeRepos` adds a second.

### Can my API look up a handle on demand?

Yes. Start a run with a one-item `usernames` array from your route handler and await it, or, if your platform has short timeouts, use `start()` and pick up the dataset from an Apify webhook when the run succeeds.

### What does `concurrency` control?

How many lookups the Actor keeps in flight at once, from 1 to 32 with 6 as the default. Lower it if you share a token with other jobs and want to leave headroom in its hourly budget.

### Are private repositories or org members included?

No. Rows contain public profile data only, and the `repos` array lists public repositories. Private membership and private code need scopes this read-only lookup does not use.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/github-user-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [GitHub User Scraper — Profiles, Repos & Orgs on Apify Store](https://apify.com/DevilScrapes/github-user-scraper)
- [Python version of this example](https://github.com/devilscrapes/github-user-scraper-python)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [GitHub Organization Scraper](https://apify.com/DevilScrapes/github-org-scraper) — organization metadata plus its public repository list, including member counts.
- [GitHub Repo Scraper](https://apify.com/DevilScrapes/github-repo-scraper) — deep metadata for individual repositories, such as topics, license and latest release.
- [GitHub Trending Scraper](https://apify.com/DevilScrapes/github-trending-scraper) — what is climbing GitHub's trending page today.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/github-user-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with GitHub or Microsoft.
