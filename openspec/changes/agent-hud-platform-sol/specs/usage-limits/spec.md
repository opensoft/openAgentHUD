## Purpose

Present provider/account usage limits, balances, and costs as scoped evidence with visible age, accurate reset behavior, and explicitly enabled external queries rather than fabricated quota certainty.

## ADDED Requirements

### Requirement: Scoped limit and balance records

Usage data SHALL be a list of independently identified records with provider, account scope key, window/limit identity, metric kind, unit, observation time, and available amount/capacity/reset evidence. Quota, credit balance, measured monetary cost, and estimated monetary cost MUST remain distinct. The number of windows MUST NOT be restricted to primary/secondary slots, and records from different accounts, currencies, units, or overlapping windows MUST NOT be summed without declared compatible semantics.

#### Scenario: More than two quota windows
- **WHEN** a provider supplies hourly, daily, and monthly limits for one account
- **THEN** all three retain independent identities, units, observation times, and reset information

#### Scenario: Credits alongside monetary cost
- **WHEN** a provider supplies 20 credits and a measured cost of 2 USD
- **THEN** credits and cost appear as separate metric kinds and are not combined into one usage number

### Requirement: Derived usage and reset displays

The HUD SHALL display reported or validly derived used/remaining percentages, absolute reset time, and reset countdown where supported. Derived percentages MUST use compatible units and a positive known capacity. At a reset deadline, the HUD MUST request reconciliation or mark the record uncertain; it MUST NOT fabricate zero usage, replenished credits, a fresh window, or a new reset deadline.

#### Scenario: Reset occurs without fresh provider data
- **WHEN** the displayed reset deadline passes while the provider cannot be queried
- **THEN** the deadline is shown as elapsed/pending refresh, the old record remains cached or stale with age, and usage is not reset to zero

### Requirement: Account-safe cache and visible freshness

Quota and balance records SHALL retain provider freshness, cache age, source provenance, and scope. Cached records MUST remain visibly cached until revalidated. A different or unknown account key MUST invalidate the old account's records, and provider errors MUST NOT display an empty healthy quota meter.

#### Scenario: Account switch with a populated cache
- **WHEN** the selected session switches from account A to account B
- **THEN** A's records are excluded even if their cache TTL has not expired

#### Scenario: Provider query fails
- **WHEN** a previously successful quota query times out
- **THEN** its retained values show cache age and degraded freshness and the timeout is reported without exposing credentials

### Requirement: Explicit bounded external usage providers

External usage providers SHALL run only after explicit configuration or command-line opt-in; inherited environment hints MUST NOT enable them. Command providers MUST receive explicit executable/argument arrays without implicit shell evaluation or substitution from telemetry, and a documented minimal environment rather than the agent's complete environment. HTTP providers MUST validate the exact configured scheme/host/port and all resolved addresses on every attempt, pin approved connection addresses, reject URL credentials/non-HTTP schemes, and reject private/link-local/loopback destinations unless individually allowed as trusted endpoints. Redirects SHALL be disabled by default and any explicitly enabled redirect MUST pass the same checks without forwarding credentials across origins. Both provider types MUST enforce timeout/output/parsing/concurrency/rate/retry bounds and cancellability. Credential access MUST be explicitly referenced and request-scoped, never retained in telemetry, visible argv, raw errors, or copied stores. Returned data MUST pass the normalized usage contract before publication.

#### Scenario: Default offline startup
- **WHEN** the HUD starts with default configuration
- **THEN** it makes no external usage query and requires no provider credentials

#### Scenario: Redirect leaves an allowed destination
- **WHEN** an opted-in HTTP provider redirects to a destination outside its configured allowlist
- **THEN** the query is rejected before forwarding credentials or accepting the redirected response

#### Scenario: Command output exceeds its bound
- **WHEN** an enabled command provider exceeds its configured output or execution limit
- **THEN** the command is stopped, its result is rejected, and the provider is reported degraded without disrupting the agent

#### Scenario: Destination changes during resolution
- **WHEN** an allowed hostname resolves to mixed approved and private IPv4/IPv6 addresses or changes after validation
- **THEN** unapproved addresses are rejected and the connection uses only an approved pinned address without leaking request credentials

#### Scenario: Agent environment contains credentials
- **WHEN** an opted-in command provider runs beside an agent with unrelated secret environment values
- **THEN** only the provider's documented minimal environment and explicit request-scoped credential inputs are available, and no secret appears in argv or diagnostics

### Requirement: Pricing and estimate provenance

Optional cost estimates SHALL identify their currency, pricing source, pricing date, included token categories, and estimate quality. Missing pricing evidence MUST leave cost unknown. Estimated cost MUST NOT be presented as a provider bill or balance, and price lookup MUST NOT cause implicit network activity.

#### Scenario: Local dated price table
- **WHEN** an enabled estimate uses verified token categories and a dated local price table
- **THEN** the result is labeled estimated with currency, pricing date, and source, independently of any reported monetary cost

### Requirement: Provider extensions remain isolated

Provider-specific quota acquisition and extra fields SHALL stay in provider namespaces and capability declarations. Adding a usage provider MUST NOT require modifying common quota rendering or translating provider windows into fixed common slots.

#### Scenario: Provider adds a request-count limit
- **WHEN** a new provider supplies a request-count window through the normalized record contract
- **THEN** existing usage presentation displays its unit and scope without provider-specific rendering changes
