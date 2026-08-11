# qa-robots-test-site

Static fixture site for manually QA-testing accessFlow's scanner `robots.txt` compliance
(AC-2708 / [accessFlow#1364](https://github.com/acsbe/accessFlow/pull/1364)) on staging. Hosted via
GitHub Pages at:

**https://nehemiah-accessibe.github.io/qa-robots-test-site/**

## Pages

- `/` — homepage, links to both pages below (exercises live-crawl link-following discovery)
- `/public/about.html` — always allowed
- `/private/secret.html` — disallowed by the default `robots.txt`
- `/sitemap.xml` — lists all three pages (exercises sitemap-derived discovery)

Both discovery paths accessFlow's crawler uses (sitemap-derived and live-crawl link-following)
reach `/private/secret.html`, so a scan with the flag on needs to exclude it via *either* path to
prove the fix works, not just one.

## How to use for QA

1. In accessFlow staging, add this site as a website (`nehemiah-accessibe.github.io/qa-robots-test-site` as the
   domain, or the full path if the product requires a bare domain — adjust as needed for how
   accessFlow models sub-path sites).
2. Turn the `aflwScannerRobotsTxt` LaunchDarkly flag **on** for that website only.
3. Run a scan. `/private/secret.html` must not appear among scanned webpages;
   `/` and `/public/about.html` must appear normally.
4. Turn the flag **off** and re-scan the same website as a regression check — `/private/secret.html`
   should now scan normally (this is pre-existing, unchanged behavior).

The live `robots.txt` at the repo root is the "basic disallow" scenario and should be left as the
default between test runs — other QA scenarios need a different `robots.txt` and are handled by
temporarily swapping it (see below), not by adding more permanently-live sites, so keep this repo
in its default state when you're not actively mid-scenario.

### Testing the other scenarios (missing / full-disallow / malformed)

These need `robots.txt` itself to be different, so they're not simultaneously live. To run one:

```sh
git clone https://github.com/nehemiah-accessibe/qa-robots-test-site.git
cd qa-robots-test-site

# Full-disallow scenario: robots.txt blocks the entire site.
cp scenarios/robots-full-disallow.txt robots.txt
git commit -am "test: full-disallow scenario" && git push
# ... run the scan, confirm ~0 pages scanned and the
# "SiteUrlScan - run: robots.txt disallows crawling this site entirely" warning appears in logs ...

# Malformed scenario: robots.txt isn't valid robots.txt content at all.
cp scenarios/robots-malformed.txt robots.txt
git commit -am "test: malformed scenario" && git push
# ... run the scan, confirm it proceeds as if there were no restrictions (no crash) ...

# Missing scenario: no robots.txt at all (404).
git rm robots.txt
git commit -am "test: missing robots.txt scenario" && git push
# ... run the scan, confirm it behaves exactly as with the flag off ...

# Always restore the default afterward:
cp scenarios/robots-default.txt robots.txt
git add robots.txt
git commit -am "restore default robots.txt"
git push
```

GitHub Pages redeploys within roughly a minute of a push — give it a moment before re-scanning.

## Why a real hosted site, not a local fixture

accessFlow's staging scanner runs on GCP and needs to make a real outbound HTTP fetch to a real,
internet-reachable origin. The repo's own Jest fixtures
(`test/scanner/services/pages/robots.txt` in the main `accessFlow` repo) cover automated unit/
integration tests against a local server, but can't be reached by a real staging scan — that's what
this site is for.
