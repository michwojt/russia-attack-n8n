# AI News Intelligence & Alert System

An event-driven AI news monitoring workflow built with n8n, Jev, News API, and SMTP.

The system continuously monitors news, removes duplicate articles, evaluates their relevance and potential impact using an AI decision model, and sends an email alert when configurable thresholds are met.

## Architecture

```text
                    ┌─────────────────┐
                    │    News API     │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  Split Articles │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ URL Deduplication│
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │     Jev AI      │
                    │  Classification │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  Decision Logic │
                    │                 │
                    │ Relevance       │
                    │ Direction       │
                    │ Thresholds      │
                    └────────┬────────┘
                             │
                       Alert condition
                             │
                             ▼
                    ┌─────────────────┐
                    │   SMTP Email    │
                    └─────────────────┘
```

## What It Does

The workflow processes newly published news articles and evaluates each article against a defined geopolitical event.

For each article, Jev evaluates:

- **Relevance** — whether the article is relevant to a potential Russia–NATO military confrontation.
- **Less probable** — whether the information makes a Russian attack less probable.
- **More probable** — whether the information makes a Russian attack more probable.

The workflow then applies configurable thresholds to determine whether an alert should be sent.

## Alert Logic

An article passes the first filter when:

```text
relevance >= relevance_threshold
```

and at least one directional probability exceeds the configured threshold:

```text
less_probable >= direction_threshold
OR
more_probable >= direction_threshold
```

When an alert is triggered, the stronger directional signal determines the email classification:

```text
more_probable > less_probable
        → HIGH

less_probable >= more_probable
        → LOW
```

The thresholds are stored in a dedicated configuration node rather than being hard-coded throughout the workflow.

## Example

Jev might return:

```json
{
  "relevant": 0.96,
  "less_probable": 0.26,
  "more_probable": 0.47
}
```

With:

```text
relevance_threshold = 0.80
direction_threshold = 0.30
```

the article passes the relevance test and the `more_probable` direction exceeds the threshold.

The resulting alert is classified as:

```text
HIGH - Higher chance of Russia attack
```

## Technologies

- n8n
- Jev (`jev-1.13`)
- News API
- Gmail SMTP
- n8n Data Tables
- REST APIs
- JSON
- Expression-based decision logic

## Workflow Components

### News Ingestion

The workflow retrieves current news articles through a REST API.

### Deduplication

Article URLs are stored in an n8n Data Table.

Before an article is processed by Jev, the workflow checks whether its URL already exists.

This prevents the same article from repeatedly consuming AI/API resources.

### AI Analysis

New articles are sent to Jev through an HTTP Request node.

The article title, description, and content are provided as the model input.

Jev returns structured probabilities rather than free-form text.

### Decision Engine

The workflow uses deterministic n8n logic to combine the AI output with configurable thresholds.

This separates:

- AI-based classification
- deterministic business rules
- notification delivery

### Notification

Articles that satisfy the alert criteria are sent via SMTP email.

The email contains:

- alert direction
- article title
- article description
- article URL
- Jev probabilities

## Configuration

The workflow contains a configuration node with parameters such as:

```text
relevance_threshold
direction_threshold
```

API credentials and SMTP credentials are configured through n8n credentials and are **not included in the repository**.

## Setup

1. Import the workflow JSON into n8n.
2. Configure the News API credential.
3. Configure the Jev API credential.
4. Configure SMTP credentials.
5. Configure the n8n Data Table.
6. Set the desired thresholds in the Config node.
7. Activate the workflow.

## Security

No API keys, passwords, SMTP credentials, or other secrets should be committed to this repository.

Credentials should be configured through n8n's credential system.

## Project Status

**Working prototype**

The current version implements:

- automated news ingestion
- article deduplication
- AI classification
- configurable decision thresholds
- directional alert classification
- email notifications

Future improvements may include:

- Telegram notifications
- additional news sources
- configurable market/event definitions
- historical evaluation of Jev classifications
- alert performance metrics
- monitoring and error handling
