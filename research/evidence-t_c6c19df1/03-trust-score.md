[![CoinGecko Knowledge Base](https://avatars.slack-edge.com/2025-04-08/8723447122498_27920a1063f64edb4313_88.png)![CoinGecko Knowledge Base](https://avatars.slack-edge.com/2025-04-08/8723447122498_27920a1063f64edb4313_88.png)](https://support.coingecko.com/)

[API Docs](https://docs.coingecko.com/) [Submit a request](https://contact.coingecko.com/forms/support-request-form)

Log In

[API Docs](https://docs.coingecko.com/) [Submit a request](https://contact.coingecko.com/forms/support-request-form)

# Trust Score Methodology

Last updated 13 days ago

## Purpose of the Trust Score

In traditional financial markets, tight spreads, thick orderbooks, and high trading volumes are often reliable indicators of deep liquidity. However, in the crypto markets, these are not always the case, as exchanges may artificially inflate their reported trading volume.

Since 2019, CoinGecko has developed and introduced Trust Score to users with a more reliable and multi-dimensional measure of an exchange's quality. The Trust Score methodology goes beyond reported volume to consider a comprehensive set of components including liquidity, cybersecurity, and more, to arrive at an overall score for an exchange.

Over time, CoinGecko’s Trust Score methodology has evolved together with developments in the industry, such as the demand for transparent proof-of-reserves, the advent of crypto exchange regulations, the growing institutional presence on orderbooks, and the proliferation of mobile trading. This latest methodology update takes into account all such aspects to arrive at a holistic view of an exchange.

## Score Composition

The Trust Score is computed from five components:

| **Component** | **Weight** |
| Liquidity | 50% |
| Cybersecurity | 20% |
| Regulation | 15% |
| Incident | 10% |
| Proof of Reserves | 5% |

The five component scores are combined using the weights shown above. However, the final Trust Score is not a simple weighted sum.

Instead, the combined score is graded on a curve consisting of the full population of scored exchanges, before being assigned a final Trust Score. This means an exchange's final Trust Score reflects its relative standing among peers, rather than a number on a fixed scale. Two exchanges with similar raw component scores will receive similar final scores; an exchange that leads across most components will score near the top of the distribution, not at a fixed ceiling.

Trust Scores are recalculated **weekly**. This cadence balances responsiveness to real changes against the ranking volatility that more frequent recalculation would create.

## Liquidity

Liquidity refers to how likely a crypto asset is to be traded at a price that reflects its market value, and is the most heavily weighted component of the Trust Score. The Liquidity score takes into account factors such as trading volume, order book depth, spread, and trading activity across a selected set of trading pairs on an exchange.

**Factors Influencing the Liquidity Score**

A set of top liquid coins is selected based on market cap as the parameters for scoring. Tickers from a set of trusted exchanges with high confidence of truthfulness is also selected as a reference benchmark set. The set of coins, as well as trusted exchanges, may change over time as part of our ongoing efforts to improve our assessment.

For each exchange, the liquidity score is primarily assessed based on the best ticker for each coin in the evaluation set. An aggregated score is calculated primarily from two factors:

1. **Volume consistency.** Whether the exchange's reported trading volume for these best tickers is consistent against the reference benchmark set.

2. **Order book depth.** Whether the exchange's order book depth for these best tickers falls within the range observed across the reference benchmark set.


Exchanges whose reported data is consistent with the reference benchmark set will score well. Exchanges whose data deviates significantly will be penalized for displaying abnormal patterns.

**Exchange-Level Quality Gate**

An exchange-level quality gate ensures that just maintaining a small number of clean trading pairs, while the rest of the tickers on the exchange exhibiting anomalous activity or poor liquidity is not sufficient to receive a high score. The gate considers the overall pattern across an exchange's evaluated pairs, not just its best-performing ones.

**Trading Pair Score**

Each individual trading pair on an exchange is also evaluated based on its specific liquidity, trading volume, order book depth, and trading activity. These individual pair scores feed into the exchange's overall Liquidity Score.

**Manual Review**

A human review process may also take place to evaluate liquidity in ways that the algorithm cannot automatically detect. This may include patterns that are technically within algorithmic thresholds but appear anomalous in context. Care is taken to minimize human intervention in determining the final liquidity score of any exchange.

## Regulation

The Regulation Score reflects the depth of regulatory oversight an exchange has voluntarily placed itself under for spot trading activity. It is a regulatory-posture signal — a measure of how much an exchange has chosen to implement regulatory controls and compliance requirements, while also subjecting itself to external supervision — not a full compliance audit or legal endorsement.

### Verification Process

Regulatory credentials may be identified through CoinGecko's own research or submitted directly by an exchange. Regardless of the source, every credential goes through the same verification process:

1. **Claim identification.** We identify the specific license, registration, or authorization the exchange claims to hold, along with the issuing authority and jurisdiction.

2. **Register verification.** We check the claim against the official register or public record maintained by the relevant regulatory body. If the register does not confirm the credential, or if no publicly accessible register exists, the credential is excluded.

3. **Classification.** Verified credentials are classified along a regulatory oversight spectrum based on a standardized assessment.


Self-reported credentials that cannot be independently confirmed are not scored. An exchange may hold multiple credentials across jurisdictions; only the verified credential that ranks highest on the regulatory spectrum determines its score.

### Regulatory Spectrum

Regulatory regimes vary significantly in scope and depth. A regime that requires capital reserves, market integrity requirements, and client-asset segregation rules represents a fundamentally different level of oversight from one that requires only a voluntary registration.

We evaluate each verified credential on a regulatory oversight spectrum. On one end are regimes with the most comprehensive crypto exchange frameworks — rigorous authorization requirements, continuous rigorous supervision, and strong anti-money laundering, client-asset protection, and market-conduct requirements. On the other end are credentials that are issued under frameworks with limited scope and observable operating history.

### Scope and Limitations

- **Spot exchange activity only.** Credentials related solely to derivatives, payments, or other non-spot activities are outside the current scope.

- **Not a compliance certification.** A high Regulation Score means the exchange holds a credential issued by a regime with comprehensive oversight. It does not mean CoinGecko has audited the exchange's compliance with that regime's requirements.

- **Credentials may change.** Regulatory status can change — licenses can be suspended, revoked, or upgraded. We reverify credentials as part of our regular recalculation cycle and will update scores accordingly.

- **The rubric may evolve.** As regulatory frameworks mature and new regimes emerge, we may reclassify credentials or adjust the spectrum.


Exchanges that believe their regulatory credential is missing or misclassified may [submit a request](https://forms.gle/wW5e5qrvRxCeFuup9) with supporting documentation, including the credential reference number and a link to the relevant public register entry.

## Cybersecurity

The Cybersecurity Score is evaluated by [CORE3](https://core3.io/?utm_source=coingecko&utm_medium=referral&utm_campaign=exchange_trustscore), a Risk Infrastructure & Intelligence Platform powered by Hacken. Once your exchange has been verified by CORE3, your Cybersecurity Score will automatically be reflected on CoinGecko within 48 hours. For more information about CORE3's scoring methodology, please refer to this [article](https://support.coingecko.com/hc/en-us/articles/35848265328537).

## Past Incidents

Past incidents refer to whether an exchange has experienced security or functional issues that can potentially affect a user's fund safety.

| **Condition** | **Score** |
| Exchange has no incident recorded | Full score |
| Exchange has minor incident(s) recorded | Partial score |
| Exchange has major incident(s) recorded | No score |

## Proof of Reserves

Proof of Reserves (PoR) considers the availability of declared assets that can be publicly assessed, user-attestation processes, as well as financial audits. For more information, refer to our article on [Proof of Reserves](https://support.coingecko.com/hc/en-us/articles/33353806035865-Proof-of-Reserve-Updates-on-CoinGecko).

Exchanges can verify their Proof of Reserves via [DefiLlama](https://defillama.com/). Once it's verified on DefiLlama, please [submit a request](https://support.coingecko.com/hc/en-us/requests/new) for our further verification.

## Other Components

The following attributes are tracked and displayed on exchange pages but are not factored into the Trust Score calculation. These attributes may be incorporated into the Trust Score in future iterations, so we recommend that exchanges ensure this information is up-to-date.

### Team Presence

Team Presence evaluates the transparency and public availability of the exchange’s senior leadership. Certain exchanges may have public senior team members (doxxed), while other exchanges may have leadership teams with little or no public profiles (non-doxxed).

### API Coverage

We measure the completeness of an exchange API for the following criteria:

1. Tickers Data

2. Historical Trades Data

3. Order Book Data

4. Candlestick/OHLC

5. WebSocket API

6. API Trading

7. Public Documentation


Please provide the relevant endpoint and [submit a request](https://support.coingecko.com/hc/en-us/requests/new) for our verification.

Score parameters may be adjusted over time as we refine the model. Any methodology changes will be reflected in the published documentation.

In order to preserve the integrity of the Trust Score, we're only able to share the information outlined above. Revealing specific thresholds or metric breakdowns could open the door to manipulation, ultimately reducing its value as a reliable signal for the users who depend on it. We understand this may leave some questions unanswered, but we believe this approach best serves the community, and remain committed to providing transparency wherever we can.

## Further Reading

- [**Trust Score (Basilisk Update)**](https://support.coingecko.com/hc/en-us/articles/57645752611865-Trust-Score-Basilisk-Update)

- [Trust Score 3.0 - Proof of Reserves (Assets & Liabilities)](https://support.coingecko.com/hc/en-us/articles/14570182625817-Trust-Score-3-0-Proof-of-Reserves-Assets-Liabilities)


How helpful was this article?

😞😕😐🙂😊

Related Articles

[What can I do to improve my Trust Score?](https://support.coingecko.com/articles/4563421666-what-can-i-do-to-improve-my-trust-score) [How to Submit Regulatory Information for Exchange Trust Score](https://support.coingecko.com/articles/8491872459-how-to-submit-regulatory-information-for-exchange-trust-score) [Trust Score 3.0 - Proof of Reserves (Assets & Liabilities)](https://support.coingecko.com/articles/9349500558-trust-score-3-0-proof-of-reserves-assets-liabilities)

AI Support
![Chat avatar](https://www.coingecko.com/favicon-96x96.png)