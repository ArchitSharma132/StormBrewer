# Passive Web Vulnerability Scanner

A lightweight, read-only security checker for websites you are authorized to test. It inspects HTTP responses, TLS configuration, and page content to surface common misconfigurations and weaknesses, then generates a report organized around the OWASP Top 10:2025 risk categories.

## Legal and Ethical Notice

Only run this tool against systems you own or have explicit written permission to test. Unauthorized scanning of third-party systems may be illegal depending on your jurisdiction, even when the checks performed are non-destructive. The tool will prompt for confirmation of authorization before scanning unless the `-y` flag is passed.

## What It Checks

Findings are grouped into the following categories:

- **A01 Broken Access Control** - admin/management interface exposure, risky HTTP methods, CORS misconfiguration, open redirects
- **A02 Security Misconfiguration** - missing security headers, CSP quality, directory listing, banner disclosure, missing security.txt, GraphQL introspection, API spec exposure
- **A03 Software Supply Chain Failures** - exposed dependency manifests (package.json, composer.lock, etc.), outdated frontend library versions
- **A04 Cryptographic Failures** - TLS protocol and certificate checks, active TLS 1.0/1.1 downgrade attempts, mixed content
- **A05 Injection (passive indicator only)** - reflected-parameter detection using a benign canary string, not an executable payload
- **A07 Authentication Failures** - cookie security flags, login forms submitted over HTTP, password autocomplete settings
- **A08 Software/Data Integrity Failures** - missing Subresource Integrity (SRI) on third-party scripts and stylesheets
- **A10 Mishandling of Exceptional Conditions** - verbose error pages and stack trace leakage
- **Email and DNS** - SPF and DMARC record presence

## What It Does Not Do

This is a hygiene and recon scanner, not a penetration testing tool. It deliberately does not:

- Send SQL, command, XXE, SSTI, or other injection payloads
- Attempt authentication bypass, brute forcing, or credential guessing
- Perform denial-of-service or load testing
- Test business logic (race conditions, price manipulation, workflow bypass)

These require controlled, explicitly scoped testing and are better handled by a manual review or a dedicated penetration test.

## Requirements

- Python 3.8+
- `requests`
- `beautifulsoup4`
- `dnspython`

Install dependencies:

```
pip install requests beautifulsoup4 dnspython
```

## Usage

Run a safe local self-test with no internet access required. This starts a deliberately insecure demo app on 127.0.0.1, scans it, and shuts it down automatically:

```
python3 vuln_scanner.py --demo
```

Scan a target you are authorized to test:

```
python3 vuln_scanner.py https://example.com
```

Skip the authorization confirmation prompt (useful for automation):

```
python3 vuln_scanner.py https://example.com -y
```

Generate JSON and HTML reports:

```
python3 vuln_scanner.py https://example.com --json report.json --html report.html
```

Skip sensitive-path and dependency-manifest checks:

```
python3 vuln_scanner.py https://example.com --skip-paths
```

Skip parameter-based probes (reflected-input check, open-redirect check, TLS downgrade attempts) for the lightest possible scan:

```
python3 vuln_scanner.py https://example.com --skip-probes
```

Other available flags:

- `--timeout SEC` - per-request timeout in seconds (default 10)
- `--no-color` - disable colored console output

## Output

The console report groups findings by category and severity (critical, high, medium, low, info) and includes a hygiene score out of 100. JSON and HTML reports can be generated with `--json` and `--html` for sharing or archiving.

## Limitations

- This tool performs passive and light-touch checks only. A clean report does not mean a target is free of vulnerabilities, only that these specific checks did not detect an issue.
- Some findings (for example, exposed sensitive paths) can be false positives on sites that return HTTP 200 for all unknown paths. The tool compares against a baseline request to reduce this, but manual verification is still recommended.
- The outdated-library detection uses a small hardcoded list of version thresholds and is not a substitute for a real CVE database or software composition analysis tool.
- DNS-based checks (SPF/DMARC) are skipped for local or private hostnames.
