# Global Internet Incidents

A selected, non-exhaustive catalog of major incidents affecting Internet
connectivity and widely used online services around the world. It includes
physical infrastructure failures, routing incidents, service outages, and
large-scale disruptions to Internet-dependent systems.

Dates below are UTC. Incidents are listed chronologically; impact descriptions
are concise summaries of the cited reports.

| Date | Incident | Affected scope | Type | Impact | Source |
|---|---|---|---|---|---|
| 2008-01-30 | Mediterranean submarine cable cuts | Middle East and South Asia | Physical infrastructure failure | Damage to multiple cables near Egypt and later in the Persian Gulf reduced international connectivity and disrupted Internet and phone traffic across the region. | [RIPE NCC analysis](https://www.ripe.net/analyse/archived-projects/mediterranean-fibre-cable-cut/) |
| 2008-02-24 | Pakistan Telecom YouTube route hijack | YouTube users worldwide | BGP route hijack | An unauthorized route announcement was propagated globally, redirecting some YouTube traffic to Pakistan and making the service unreachable for affected users. | [RIPE NCC case study](https://www.ripe.net/about-us/news/youtube-hijacking-a-ripe-ncc-ris-case-study/) |
| 2011-01-27 | Egyptian Internet disconnection | Egypt | Government-ordered network disruption | Many Egyptian networks disappeared from the global Internet during the country-wide disruption, although some networks and services remained reachable. | [RIPE Labs analysis](https://labs.ripe.net/author/csquarce/three-case-studies-on-the-egyptian-disconnection/) |
| 2016-10-21 | Dyn managed DNS DDoS attack | DNS-dependent websites and services worldwide | DDoS attack | Two attack waves against Dyn’s managed DNS infrastructure caused availability problems for customers and their users. | [Dyn incident analysis (archived)](https://postmortem.io/incidents/dyn--2016-10-21--managed-dns-ddos-mirai-attack/) |
| 2017-02-28 | Amazon S3 outage in US-EAST-1 | AWS customers and dependent websites and services | Cloud service outage | An incorrectly entered operational command removed more servers than intended, disrupting S3 and other AWS services in the Northern Virginia region. | [AWS summary](https://aws.amazon.com/message/41926/) |
| 2019-06-24 | Verizon BGP route leak | Cloudflare and other networks, from parts of the Internet | Routing incident | Incorrectly propagated routes made many websites and services unreachable from parts of the Internet for about two hours. | [Cloudflare analysis](https://blog.cloudflare.com/how-verizon-and-a-bgp-optimizer-knocked-large-parts-of-the-internet-offline-today/) |
| 2021-06-08 | Fastly CDN outage | Websites and services worldwide using Fastly | CDN software bug | A latent software bug triggered by a valid customer configuration change caused errors across 85% of Fastly’s network. | [Fastly incident summary](https://www.fastly.com/blog/summary-of-june-8-outage) |
| 2021-10-04 | Meta global outage | Facebook, Instagram, WhatsApp, and related services worldwide | Backbone configuration failure | A faulty configuration change disconnected Meta’s data centers and caused its services to become unreachable. | [Meta engineering postmortem](https://engineering.fb.com/2021/10/05/networking-traffic/outage-details/) |
| 2021-12-07 | AWS US-EAST-1 service event | AWS services and customers in Northern Virginia, with downstream effects | Cloud network congestion | A surge in internal connection activity overwhelmed network devices and caused elevated errors and latency across affected services. | [AWS event summary](https://aws.amazon.com/message/12721/) |
| 2022-07-08 | Rogers network outage | Rogers wireless and wireline customers across Canada | IP core routing failure | A configuration change led to a flood of routing information that overwhelmed core routers, interrupting Internet, phone, and other services. | [CRTC assessment](https://crtc.gc.ca/eng/publications/reports/xona2024.htm?t) |
| 2024-07-19 | CrowdStrike Channel File 291 incident | Windows systems and dependent organizations worldwide | Faulty software update | A faulty security-content update crashed Windows systems, disrupting services including air travel and healthcare. | [U.S. GAO summary](https://www.gao.gov/products/gao-24-107733) |

## Contributing

Add incidents that had significant, documented effects on Internet
connectivity or widely used online services. Keep entries chronological, use
UTC dates, describe the affected scope and impact factually, and link to a
primary source or a reliable technical analysis.
