# qa-robots-test-site

Static fixture site for manually QA-testing a web crawler's `robots.txt` compliance. Hosted via
Netlify (continuous deploy from this repo's `main` branch) at:

**https://robots-qa.netlify.app/**

## Pages

- `/` — homepage, links to both pages below (exercises link-following discovery)
- `/public/about.html` — always allowed
- `/private/secret.html` — disallowed by the default `robots.txt`
- `/sitemap.xml` — lists all three pages (exercises sitemap-derived discovery)

Both common crawler discovery methods (sitemap-derived and link-following) reach
`/private/secret.html`, so a scan needs to exclude it via *either* method to prove robots.txt
compliance actually works, not just one.

## How to use for QA

1. Point the crawler/scanner under test at this site (as a full URL or domain, depending on how
   the tool being tested models sites).
2. Run a scan/crawl with robots.txt compliance enabled. Confirm `/private/secret.html` never
   appears among crawled pages, while `/` and `/public/about.html` appear normally.
3. Run the same scan/crawl with robots.txt compliance disabled (if the tool under test supports
   toggling it) as a regression check — `/private/secret.html` should now be crawled normally.

The live `robots.txt` at the repo root is the "basic disallow" scenario and should be left as the
default between test runs — other scenarios need a different `robots.txt` and are handled by
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
# ... run the scan/crawl, confirm ~0 pages crawled ...

# Malformed scenario: robots.txt isn't valid robots.txt content at all.
cp scenarios/robots-malformed.txt robots.txt
git commit -am "test: malformed scenario" && git push
# ... run the scan/crawl, confirm it proceeds as if there were no restrictions (no crash) ...

# Missing scenario: no robots.txt at all (404).
git rm robots.txt
git commit -am "test: missing robots.txt scenario" && git push
# ... run the scan/crawl, confirm it behaves exactly as with robots.txt compliance disabled ...

# Always restore the default afterward:
cp scenarios/robots-default.txt robots.txt
git add robots.txt
git commit -am "restore default robots.txt"
git push
```

Netlify redeploys within roughly a minute of a push — give it a moment before re-scanning.

## Why a real hosted site, not a local fixture

A crawler running server-side (rather than from a developer's own machine) typically needs to make
a real outbound HTTP fetch to a real, internet-reachable origin — it can't reach `localhost` or
anything only reachable from one machine's local network. That's what this site is for.
