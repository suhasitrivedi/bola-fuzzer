 ## BOLA Fuzzer

Black-box tool that detects Broken Object Level Authorization (BOLA) in REST APIs. Given an OpenAPI 3.0 spec and two same-privilege accounts, it reports which endpoints let User B read, modify or delete User A's data.

Implements the paper "Automated Detection of Broken Object Level Authorization (BOLA) in REST APIs Using Specification-Guided Dual-Identity Dynamic Fuzzing" by Suhasi Trivedi, Kaushal Shah (PDEU, India) and Harshal Sanghvi (Florida Atlantic University, USA).

Research prototype. Results are preliminary (two self-built reference apps).

## How it works

For each route with an ID in its path (e.g. /notes/{id}) that has a POST creator, the tool tests GET, PUT/PATCH and DELETE:

User A creates a resource and the ID is captured.
Controls: B acts on B's own resource, B acts on a non-existent ID, an unauthenticated client sends a GET.
User B performs the operation on A's resource.
User A re-reads it to check whether the data really changed.
The oracle returns a verdict: confirmed_bola, suspected_bola, not_vulnerable, public_resource or inconclusive.
All created resources are deleted and logged in reports/cleanup_registry.json.

Exit code: 0 = no findings, 2 = confirmed or suspected BOLA, 1 = setup error.

# Install

Python 3.9+ (tested on 3.12 and 3.14).

# bash
pip install -r requirements.txt


The fuzzer registers the two test users itself. Use two terminals.

Vulnerable app (expect 3 confirmed):

# bash
python3 sample_target/vulnerable_app.py          # terminal 1, port 5000
python3 -m bola_fuzzer.cli --config config.example.yaml --report reports/vuln_report.json   # terminal 2

Hardened app (expect 0 findings): stop the first server, then

# bash
python3 sample_target/hardened_app.py            # terminal 1, port 5001
python3 -m bola_fuzzer.cli --config config_hardened.yaml --report reports/hardened_report.json   # terminal 2

Target	Tests	Confirmed	Suspected	False positives	True negatives
Hardened app	3	0	0	0	3
Vulnerable app	3	3	0	0	0
Use on your own API

The tool creates, modifies and deletes data.

Copy config.example.yaml and set openapi_spec, base_url, identity_a and identity_b, then run python3 -m bola_fuzzer.cli --config your.yaml.

Config keys per identity: login_url, login_payload, token_field, token (pre-obtained token, never refreshed), register_url (optional, creates the account first), whoami_url. Top level: rate_limit_delay_s (pause before each request). Login tokens are refreshed once on a 401.

# Limitations
Only object IDs in the URL path are covered (not query, body or headers).
ID and field matching uses name and type heuristics, so check the evidence for each finding.
crAPI/VAmPI evaluation and the AuthProbe/Autorize comparison are not done yet.
Layout

bola_fuzzer/ (spec_parser, identity, engine, oracle, cleanup, report, cli), sample_target/ (vulnerable and hardened apps plus spec), config.example.yaml, config_hardened.yaml.