## Serving Pattern: Online + Streaming

I have chosen a hybrid **Online + Streaming** serving pattern to satisfy the requirements of the image moderation scenario.

**Online Serving:** The platform requires moderation decisions to be made within a p95 latency budget of 600 ms. To meet this requirement, a lightweight Edge Filter performs an initial moderation check immediately after image upload. This allows clearly safe content and obvious policy violations to be identified quickly without requiring expensive cloud processing. As a result, most users receive near real-time moderation outcomes, improving the overall user experience.

**Streaming Serving:** While many moderation decisions can be made rapidly, some images may contain ambiguous or borderline content that requires more sophisticated analysis. These cases are forwarded to a streaming pipeline through a message queue. By decoupling image ingestion from deeper moderation analysis, the platform can continue accepting uploads even during traffic spikes while ensuring that uncertain cases receive additional scrutiny. This design supports scalability and prevents expensive cloud inference from becoming a bottleneck.

The combination of Online and Streaming serving patterns enables the system to achieve low latency for straightforward cases while maintaining moderation quality for complex cases.

## Inference: Hybrid

A **Hybrid Inference** strategy is used to balance latency, accuracy, throughput, and cost.

**Edge (Lightweight Inference):**

* Placement: Edge service located close to the user and integrated with the ingestion layer.
* Purpose: Perform rapid first-pass moderation and identify high-confidence safe content or obvious violations.
* Rationale: Executing lightweight inference near the point of upload reduces response times and minimizes unnecessary cloud processing. Since a large proportion of uploaded images are expected to be straightforward cases, filtering them at the edge helps the system consistently meet its latency target.

**Cloud (Heavy Inference):**

* Placement: Centralized cloud infrastructure with access to more powerful compute resources.
* Purpose: Analyze images that the Edge Filter classifies as ambiguous or uncertain.
* Rationale: Larger moderation models typically provide higher accuracy but require significantly more computational resources. Restricting cloud inference to ambiguous cases allows the platform to benefit from improved decision quality without incurring the cost of running heavy models on every uploaded image. This is particularly important because moderation mistakes may have legal consequences.

## Targets & Strategy

**Optimization 1: Latency (Target: p95 < 600 ms)**

Latency is the primary user-facing performance metric. The Edge Filter enables most moderation decisions to be completed rapidly by eliminating the need for every image to be transmitted to and processed by cloud infrastructure. This helps ensure compliance with the strict p95 latency requirement.

**Optimization 2: Throughput (Target: 20 uploads/s peak)**

The streaming architecture improves throughput by separating image ingestion from advanced moderation processing. A message queue absorbs traffic spikes and distributes work to cloud inference workers, preventing overload during peak upload periods and allowing the system to scale horizontally as demand increases.

**Constraint: Cost**

Cost is a key operational constraint. Processing every uploaded image with a large cloud-based model would require substantial computational resources and increase infrastructure expenses. The hybrid approach reduces costs by reserving expensive cloud inference only for ambiguous content while allowing straightforward cases to be handled efficiently by the Edge Filter.

## Fallback Strategy: Fail-Safe to Human Review

Because moderation errors can have legal and reputational consequences, the system adopts a fail-safe escalation strategy.

**Model Unavailability:** If the cloud moderation service becomes unavailable or encounters an error, affected images are automatically redirected to the Human Review Queue. This prevents content from bypassing moderation controls due to system failures.

**Low Confidence Decisions:** When the cloud model produces a confidence score below the operational threshold, the image is treated as ambiguous and escalated to a human reviewer. This ensures that high-risk moderation decisions are verified manually rather than relying on uncertain automated predictions.

This fail-safe approach prioritizes moderation accuracy and platform integrity while maintaining compliance with the scenario's requirement that ambiguous content be reviewed by humans.
