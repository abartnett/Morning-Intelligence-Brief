# Topic Relevance Criteria

Used in Step 4 of `workflows/morning-brief.md` to classify articles into topic buckets.

For each article, read the headline and the first 2–3 sentences of the body or snippet. Match against the criteria below. An article may belong to multiple buckets. Mark articles matching no bucket as `NOISE` and discard them.

**Cross-topic boost:** If an article sits at the intersection of two or more topics, it is a strong keep and should appear in all matching sections.

---

## GEOPOLITICS — Geopolitics & Defense

### Keep if the article covers:
- Military operations: ground offensives, airstrikes, naval maneuvers, troop deployments, withdrawals
- Defense procurement: major weapons contracts, arms sales, capability acquisitions (jets, missiles, drones, ships)
- Alliance activity: NATO decisions, AUKUS/Quad/BRICS summits, bilateral security agreements, basing rights
- Sanctions with a security dimension: export controls on military technology, secondary sanctions, designation of state actors
- Diplomatic crises: ambassador expulsions, breakdown of negotiations, ultimatums, UN Security Council vetoes
- Intelligence and espionage: attribution of cyberattacks to state actors, spy scandals, surveillance programs
- Active conflict zones: Ukraine, Gaza/West Bank, Lebanon, Taiwan Strait, Korean Peninsula, Sahel, Sudan, Myanmar, Yemen, Somalia
- Nuclear and WMD programs: Iran enrichment levels, North Korea missile tests, Russian nuclear posture, NPT developments
- Hybrid warfare: cyberattacks on critical infrastructure (power grids, water systems, financial networks) attributed to state actors; information operations attributed to state actors
- Space and emerging domains: military satellites, anti-satellite weapons, AI in defense, autonomous weapons policy

### Discard (NOISE) if:
- Domestic crime with no foreign policy or intelligence dimension
- Sports, entertainment, celebrity
- Local electoral results with no geopolitical implication (e.g., a US mayoral race)
- Historical retrospectives with no current policy hook
- Natural disasters unless they trigger a security response or foreign intervention

---

## ENERGY — Energy & Resources

### Keep if the article covers:
- Oil and gas production decisions: OPEC+ quota changes, major producer output shifts (Saudi Arabia, Russia, UAE, US shale)
- LNG markets: new export terminals, long-term supply contracts, spot price moves with identified drivers
- Pipeline geopolitics: new agreements, sabotage, sanctions on pipelines, transit disputes
- Critical minerals: lithium, cobalt, nickel, copper, rare earth elements — supply chain shifts, new discoveries, export controls, resource nationalism
- Energy sanctions: Russian oil price cap enforcement, Iranian oil export restrictions, Venezuelan energy sanctions
- Refinery and infrastructure: major capacity changes, plant closures with market impact
- Power grid security: attacks on electricity infrastructure, major outages with strategic cause
- Energy transition policy with market scale: US IRA implementation, EU energy policy changes, major sovereign green investment programs
- Commodity price moves: oil, gas, or critical mineral price changes >3% in a single session with an identified geopolitical or supply driver (not just daily market noise)
- Nuclear energy: new plant approvals, major reactor deals between countries, SMR contracts of strategic significance

### Discard (NOISE) if:
- Minor utility company earnings with no macro signal
- Weather-driven demand spikes without structural implication
- EV consumer reviews or individual product launches
- Renewable energy project announcements below $500M with no policy significance
- Commodity price moves with no identified driver beyond normal market fluctuation

---

## MARKETS — Markets & Macro

### Keep if the article covers:
- Central bank decisions and forward guidance: Fed, ECB, Bank of Japan, PBoC, Bank of England, Bank of Canada — rate decisions, meeting minutes, governor statements on policy trajectory
- Major economic data releases from G7 or systemically important EM economies: GDP (quarterly or revision), CPI/PCE inflation, employment (NFP, unemployment rate), retail sales, PMI surveys
- Currency crises or significant FX moves: single-day moves >2% for major pairs (EUR/USD, USD/JPY, GBP/USD, USD/CNY, USD/EM), or ongoing currency defense operations
- Sovereign debt stress: credit rating changes (Moody's, S&P, Fitch), debt ceiling events, yield curve moves >20bps in a session, sovereign bond auctions that fail or price wide
- Trade policy with market impact: new tariffs, retaliatory tariffs, export controls affecting major trade flows, WTO rulings
- Banking system stress: bank failures, emergency liquidity operations, bail-in events, systemic risk warnings from regulators
- Major equity index moves: S&P 500, Nasdaq, FTSE, DAX, Nikkei, CSI 300 moves >1.5% in a session with an identified macro cause
- IMF and World Bank: Article IV assessments for major economies, World Economic Outlook updates, structural adjustment program conditions, emergency lending
- Financial sanctions and de-dollarization: SWIFT exclusions, correspondent banking restrictions, significant bilateral currency swap agreements

### Discard (NOISE) if:
- Individual company earnings unless they reveal systemic stress (e.g., a major bank's results signaling broad credit deterioration)
- Cryptocurrency price movements unless tied to regulatory action or macro contagion risk
- Retail consumer sentiment surveys without a macro implication
- Minor M&A deals with no industry-level or regulatory significance
- Stock recommendations or analyst upgrades/downgrades on individual names

---

## EM — Emerging Markets

### Keep if the article covers:
- Political transitions in EM economies: elections, coups, leadership changes, constitutional crises — in any country outside the G7
- China: PBoC policy moves, PMI and trade data, fiscal stimulus, Politburo decisions on economic policy, Taiwan policy statements, BRI developments, tech regulation with market impact
- India: government economic policy, GDP/CPI data, RBI decisions, geopolitical positioning (US-India, India-China, Quad), major infrastructure or industrial policy
- Southeast Asia / ASEAN: supply chain relocation trends, ASEAN summits and agreements, bilateral security or trade deals involving Vietnam, Indonesia, Thailand, Malaysia, Philippines
- Latin America: commodity-driven macro shifts, political economy of Brazil/Mexico/Argentina/Chile/Colombia, IMF programs, resource nationalism, currency stress
- Middle East: Gulf state sovereign wealth fund activity (PIF, ADIA, QIA), Vision 2030 and equivalents, oil revenue allocation, regional security architecture
- Africa: Chinese investment and debt renegotiation, IMF programs, resource nationalism, political transitions in commodity-producing states (DRC, Nigeria, South Africa, Angola, Zambia)
- Any EM country with an active IMF program, in default, or undergoing debt restructuring
- EM sovereign bond markets: spreads, issuance, rating changes for countries outside G7

### Discard (NOISE) if:
- Tourism statistics or travel advisories without security implications
- Sports events, unless they reveal something about a country's political economy (e.g., a hosting decision tied to sovereign investment)
- Local weather disasters unless they cause macro supply-chain disruption or trigger food security concerns at a national scale
- EM company earnings with no sovereign or macro signal

---

## Borderline Cases — Decision Rules

**"Could be MARKETS or GEOPOLITICS" (e.g., sanctions announcement):**  
If the primary driver is a political/security decision by a government, classify as GEOPOLITICS. If the primary frame is market impact (bond yields, FX, equity reaction), also add MARKETS. Keep in both sections.

**"Could be ENERGY or MARKETS" (e.g., oil price move):**  
If the story explains a supply-side cause (OPEC, pipeline disruption, sanctions), it is primarily ENERGY. If the story is about investor reaction, derivatives positioning, or central bank implications, also add MARKETS.

**"Could be EM or GEOPOLITICS" (e.g., China-Taiwan military activity):**  
Keep in both. These are not mutually exclusive.

**Small country with no obvious bucket:**  
If the country is an IMF program country, an active conflict zone, or a significant commodity producer, default to EM. Otherwise discard.
