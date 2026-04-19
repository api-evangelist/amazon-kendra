# Amazon Kendra (amazon-kendra)
Amazon Kendra is an intelligent enterprise search service powered by machine learning that enables organizations to index and search across multiple data sources, delivering highly accurate and relevant answers to natural language queries.

**URL:** [https://aws.amazon.com/kendra/](https://aws.amazon.com/kendra/)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AI, AWS, Enterprise Search, Knowledge Management, Machine Learning, Natural Language

## Timestamps

- **Created:** 2024-01-15
- **Modified:** 2026-04-19

## APIs

### Amazon Kendra API
The Amazon Kendra API provides programmatic access to create and manage intelligent search indexes, configure data source connectors, submit queries, and manage relevance tuning for ML-powered enterprise search.

**Human URL:** [https://aws.amazon.com/kendra/](https://aws.amazon.com/kendra/)

#### Tags:

 - Enterprise Search, ML Search, Natural Language Processing

#### Properties

- [Documentation](https://docs.aws.amazon.com/kendra/latest/dg/what-is-kendra.html)
- [OpenAPI](https://api.apis.guru/v2/specs/amazonaws.com/kendra/2019-02-03/openapi.yaml)
- [Pricing](https://aws.amazon.com/kendra/pricing/)
- [GettingStarted](https://aws.amazon.com/kendra/getting-started/)
- [FAQ](https://aws.amazon.com/kendra/faqs/)
- [Features](https://aws.amazon.com/kendra/features/)
- [APIReference](https://docs.aws.amazon.com/kendra/latest/APIReference/Welcome.html)
- [OpenAPI](openapi/amazon-kendra-openapi.yml)
- [JSONSchema](json-schema/amazon-kendra-index-schema.json)
- [JSONSchema](json-schema/amazon-kendra-data-source-schema.json)
- [JSONSchema](json-schema/amazon-kendra-query-result-schema.json)
- [JSONSchema](json-schema/amazon-kendra-faq-schema.json)
- [JSONLD](json-ld/amazon-kendra-context.jsonld)

## Common Properties

- [Blog](https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-kendra/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Console](https://console.aws.amazon.com/kendra/home)
- [CLI](https://docs.aws.amazon.com/cli/latest/reference/kendra/)
- [SDK](https://aws.amazon.com/tools/)
- [StatusPage](https://status.aws.amazon.com/)
- [Compliance](https://aws.amazon.com/compliance/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [Portal](https://aws.amazon.com/kendra/)
- [Documentation](https://docs.aws.amazon.com/kendra/)
- [Pricing](https://aws.amazon.com/kendra/pricing/)
- [GettingStarted](https://aws.amazon.com/kendra/getting-started/)
- [FAQ](https://aws.amazon.com/kendra/faqs/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [GitHubOrganization](https://github.com/aws)
- [SpectralRules](rules/amazon-kendra-spectral-rules.yml)
- [NaftikoCapability](capabilities/amazon-kendra-enterprise-search.yaml)
- [Vocabulary](vocabulary/amazon-kendra-vocabulary.yaml)

## Features

| Name | Description |
|------|-------------|
| Intelligent Search | ML-powered semantic search that understands natural language queries and context to return highly accurate answers from enterprise content. |
| GenAI RAG Support | Kendra Retriever API enables retrieval-augmented generation workflows with optimized passage chunking and ACL-based filtering for LLM integration. |
| Data Source Connectors | Native connectors for Amazon S3, SharePoint, Salesforce, ServiceNow, Google Drive, Confluence, and many more data repositories. |
| Relevance Tuning | Fine-tune search results based on document freshness, authoritative sources, and custom synonyms without ML expertise. |
| Experience Builder | No-code visual interface to build, customize, and launch search applications with drag-and-drop components. |
| Search Analytics Dashboard | Visibility into quality and usability metrics and user interaction patterns to identify content gaps. |
| Custom Document Enrichment | Preprocessing capabilities for metadata enrichment, document classification, entity extraction, and AWS AI service integration. |
| Incremental Learning | Learns from user interactions and feedback to promote preferred documents to the top of search results over time. |

## Use Cases

| Name | Description |
|------|-------------|
| Employee Productivity | Help employees find accurate answers and data-driven insights across internal knowledge bases and document repositories. |
| Customer Service | Power self-service chatbots and agent-assist solutions for contact centers with intelligent search. |
| SaaS Application Integration | Integrate intelligent search and conversational AI into customer-facing applications via the Kendra API. |
| Generative AI Applications | Use Kendra GenAI indices in Amazon Q Business and Amazon Bedrock knowledge bases to build RAG applications. |
| Enterprise Knowledge Management | Index and search across multiple heterogeneous data sources to create a unified knowledge search experience. |

## Integrations

| Name | Description |
|------|-------------|
| Amazon Bedrock | Use Kendra GenAI indices as knowledge bases in Amazon Bedrock for building generative AI applications. |
| Amazon Q Business | Integrate Kendra indices with Amazon Q Business for AI-powered enterprise assistant experiences. |
| Amazon Lex | Power Lex chatbots with Kendra search for FAQ-based conversational experiences. |
| Amazon S3 | Native data source connector for indexing documents stored in Amazon S3 buckets. |
| Microsoft SharePoint | Native connector to index and search SharePoint Online and SharePoint Server content. |
| Salesforce | Index Salesforce objects and knowledge articles for enterprise search. |
| ServiceNow | Connect to ServiceNow to index knowledge base articles and service catalog items. |
| Amazon Comprehend | Use Comprehend for entity extraction and metadata enrichment during custom document enrichment. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [Amazon Kendra API](openapi/amazon-kendra-openapi.yml)

### JSON Schema

- [Index](json-schema/amazon-kendra-index-schema.json)
- [Data Source](json-schema/amazon-kendra-data-source-schema.json)
- [Query Result](json-schema/amazon-kendra-query-result-schema.json)
- [FAQ](json-schema/amazon-kendra-faq-schema.json)

### JSON Structure

- [Index](json-structure/amazon-kendra-index-structure.json)
- [Data Source](json-structure/amazon-kendra-data-source-structure.json)
- [Query Result](json-structure/amazon-kendra-query-result-structure.json)
- [FAQ](json-structure/amazon-kendra-faq-structure.json)

### JSON-LD

- [Amazon Kendra Context](json-ld/amazon-kendra-context.jsonld)

### Examples

- [Index Example](examples/amazon-kendra-index-example.json)
- [Data Source Example](examples/amazon-kendra-data-source-example.json)
- [Query Result Example](examples/amazon-kendra-query-result-example.json)
- [FAQ Example](examples/amazon-kendra-faq-example.json)

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [Amazon Kendra](capabilities/shared/kendra.yaml) — 6 operations for intelligent enterprise search

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Amazon Kendra Enterprise Search](capabilities/amazon-kendra-enterprise-search.yaml) | Kendra | 6 | Knowledge Manager, Developer, IT Administrator |

## Vocabulary

- [Amazon Kendra Vocabulary](vocabulary/amazon-kendra-vocabulary.yaml) — Unified taxonomy mapping 8 resources, 8 actions, 1 workflow, and 3 personas across operational (OpenAPI) and capability (Naftiko) dimensions

## Rules

- [Amazon Kendra Spectral Rules](rules/amazon-kendra-spectral-rules.yml) — 18 rules across 8 categories enforcing Amazon Kendra API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
