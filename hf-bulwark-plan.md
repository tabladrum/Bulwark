# HuggingFace (hf) Bulwark: Threat Model, Market Gaps, and Rule Plan

## 1. Introduction
This document presents a comprehensive plan for a HuggingFace (hf) Bulwark proxy module, focused on securing the model supply chain for organizations consuming models from the HuggingFace Hub. It covers threat vectors, actionable rules, market analysis, and SME-level recommendations.

## 2. Threat Vectors in HuggingFace Model Supply Chain
- **Malicious Model Artifacts**: Models may contain poisoned weights, backdoors, or payloads that trigger on specific inputs.
- **Model Card/Metadata Injection**: Model cards or metadata may include malicious scripts, links, or misleading information.
- **Dependency Confusion**: Models referencing external files/scripts (e.g., custom code, requirements.txt) can pull in malicious dependencies.
- **Pickle/RCE Payloads**: PyTorch and other frameworks may load models via pickle, enabling arbitrary code execution if not sandboxed.
- **Model Version Spoofing**: Attackers may upload new versions of popular models with subtle malicious changes.
- **Typosquatting**: Malicious models with names similar to trusted models (e.g., "bert-base-uncasedd").
- **Model Card Social Engineering**: Model cards may mislead users about model provenance, safety, or intended use.
- **Poisoned Training Data**: Models trained on poisoned data may exhibit harmful or biased behavior.
- **API Abuse**: Abuse of HuggingFace APIs to exfiltrate data or trigger excessive downloads (DoS).

## 3. Market Gaps and Existing Tools
- **Lack of Model Registry Proxies**: No open-source or commercial proxies exist for HuggingFace models, unlike PyPI/npm proxies.
- **Limited Model Scanning**: Existing tools (e.g., HuggingFace's own scanning, OSS tools) focus on basic metadata, not deep inspection or policy enforcement.
- **No Policy-as-Code for Models**: No standard for expressing model trust/deny policies, version pinning, or provenance requirements.
- **No Automated Typosquat Detection**: No tools to detect or block typosquatting in model names.
- **No RCE/Deserialization Guardrails**: No proxy to block unsafe deserialization (e.g., pickle) or enforce safe loading.
- **No Model Card Linting**: No enforcement of model card content, links, or license compliance.

## 4. Actionable Rules for HuggingFace Bulwark
### 4.1 Trusted and Deny Model Lists
- Allow only models on an explicit trusted list (by name, owner, hash, or version).
- Block models on a deny list (by name, owner, hash, or version).

### 4.2 Model Name and Owner Validation
- Enforce allowed model name patterns (regex, length, character set).
- Enforce allowed/blocked model owners (users/organizations).
- Detect and block typosquatting (Levenshtein distance, common confusables).

### 4.3 Model Version and Hash Pinning
- Require explicit version pinning (no floating/latest tags).
- Validate model file hashes against known-good values.

### 4.4 Model Card and Metadata Inspection
- Lint model cards for required fields (license, description, intended use).
- Block models with suspicious/missing metadata or links.
- Enforce license allow/deny lists.

### 4.5 Deserialization and RCE Guardrails
- Block or warn on unsafe deserialization (e.g., pickle, custom code).
- Enforce use of safe loading APIs (e.g., torch.load with map_location="cpu").
- Optionally scan model files for known RCE payload patterns.

### 4.6 Dependency and External Reference Controls
- Block models referencing external scripts, requirements.txt, or custom code unless explicitly allowed.
- Enforce dependency allow/deny lists for model requirements.

### 4.7 Model Age and Update Velocity
- Require models to be older than a configurable minimum age (e.g., 7 days) to avoid zero-day attacks.
- Optionally block models with excessive update velocity (potential churn/instability).

### 4.8 Download Rate Limiting and API Abuse Protection
- Enforce per-user/IP rate limits on model downloads.
- Block or alert on suspicious API usage patterns.

## 5. Implementation Plan
- **Proxy Architecture**: Mirror Bulwark’s existing modular proxy pattern (one binary per ecosystem).
- **Rule Engine**: Reuse and extend the deterministic rule evaluation pipeline from existing Bulwark modules.
- **Config-Driven Policy**: Support YAML config for trusted/deny lists, patterns, and rule parameters.
- **Logging and Metrics**: Structured logging (slog), atomic counters for metrics, admin endpoints for health/metrics.
- **Testing**: Unit, regression, and E2E tests (mocking HuggingFace API responses, model downloads, and edge cases).
- **Documentation**: Update ARCHITECTURE.md and README.md with new module, rules, and usage examples.

## 6. SME Recommendations
- **Default Deny**: Ship with a default-deny policy; require explicit trust for all models.
- **Safe Loading by Default**: Block unsafe deserialization unless explicitly allowed.
- **Continuous Threat Intelligence**: Integrate with threat feeds for known malicious models/hashes.
- **Community/Enterprise Modes**: Support both open community and enterprise (private registry) use cases.
- **Transparency and Auditability**: Log all model fetches, rule decisions, and admin actions for audit.

---

*This document is authored by an SME in AI supply chain security. For questions or contributions, see the project README.*
