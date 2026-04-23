# Personal Build Notes

- Security Onion initially showed Elasticsearch as pending, then fault, and the Alerts page stopped working.
- The issue turned out to be a red Elasticsearch cluster state caused by an unassigned primary shard.
- Rather than rebuilding the entire Security Onion instance, I identified the broken data stream and removed it to restore usability.
- After that fix, Hunt, Alerts, and related pages loaded again even though Grid still showed Elasticsearch as pending.
- During investigation, the source attribution did not line up perfectly with the older manually assigned Kali IP used earlier in the lab.
- The suspicious activity was observed through 192.168.30.1 during this phase, which reinforced that virtual lab networking, adapter changes, and translation behavior can affect how attacker traffic appears in telemetry.
- The strongest evidence path for this project came from repeated correlation across related alert families rather than from a single enriched host identity field.