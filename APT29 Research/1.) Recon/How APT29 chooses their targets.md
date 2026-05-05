### Intro

APT29 has a particular interest in think tanks, NGOs, government contractors, and universities. That is because SVR's mission is political intelligence — they want to know what Western policy advisors are thinking before it becomes policy. Think tanks feed directly into government decision-making. APT29 is attributed to Russia's Foreign Intelligence Service (SVR) by multiple Western intelligence agencies including CISA, NSA, and the UK NCSC.. Mainly I will focus on the human-focused part, not the port scanning of services in the organization of interest.

### How they choose the target and how do they tailor individual phishing emails

They want to target employees in these organizations, they are their leverage into the internal systems of the organization. As we will see later in the other section for **Initial Access**, according to Volexity's 2016 PowerDuke report, the spear-phishing emails were designed to appear sent from real individuals at well-known think tanks. But how do they do this? How they gather all that information about an individual?

First, they have to figure out which person will receive the spear-phishing email, when they have that answered they start doing OSINT on this particular person. They will use LinkedIn - there is enough information to understand a target's role, responsibilities, institutional affiliations, and professional interests, Instagram any other social media platform in order to gather enough intelligence about what that specific person likes and does so that they can tailor the email for them. Also it's very common for websites - for example university websites, think tanks to reveal bios, research focus areas, recent publications, upcoming events, conference appearances and off-topic activities they are interested in. For example if a target recently published a paper or spoke at a conference, that's the ideal lure topic — it's something they're actively engaged with, they'll recognize related content as legitimate, and they may even expect follow-up communications about it. This way adversaries craft their phishing emails credible enough in order to deceive the target it is something legit.

Second, they may try to compromise websites that targets usually visit to steal their credentials, also credential information can be exposed to adversaries via leaks to online or other accessible data sets like code repos, dbs.

### Real Scenario

The 2016 campaigns hit immediately post-election. The lure content was election-related because that was the most compelling topic for policy researchers at that exact moment. APT29 monitored the news cycle and timed their lure content to it. That's how they managed to get so many people to click on their phishing emails.

The research process APT29 used is **identical to legitimate professional research**. There's no hacking involved in the recon phase — it's all public information aggregated and analyzed. This is why it's so hard to defend against at the individual level.

### OSINT hygiene for organizations

- Staff bio pages that reveal too much about individual research focus
- Conference attendance publicly announced before the event
- Detailed LinkedIn profiles for individuals in sensitive roles
- Public calendar listings for institutional events

Each of these is a data source an attacker uses. Reducing unnecessary public exposure of staff research interests and institutional activities directly raises the cost of targeted lure construction.

### MITRE Mappings
**T1589: Gather Victim Identity Information .001: Credentials**
**T1595: Active Scanning .002: Vulnerability Scanning**
**T1598: Phishing for Information**
