# ADR 0001: Use Hybrid Edge + Cloud Moderation

## Context

The image moderation platform must process user-uploaded images with a p95 latency below 600 ms while ensuring that moderation mistakes are minimized due to their legal implications. The system must also handle traffic spikes of up to 20 uploads per second and support escalation of ambiguous content to human reviewers.

## Decision

We will use a hybrid moderation architecture consisting of a lightweight Edge Filter and a more powerful Cloud Inference service. The Edge Filter will perform an initial moderation pass to rapidly classify clearly safe content and obvious violations. Images that cannot be classified with sufficient confidence will be forwarded through a streaming pipeline to a cloud-based moderation model for deeper analysis. If the cloud model remains uncertain, the content will be escalated to a human reviewer queue.

## Alternatives Rejected

* **Cloud-only moderation:** Rejected because every image would require network transmission and heavy cloud inference, making it more difficult to consistently meet the 600 ms latency requirement while increasing infrastructure costs.

* **Edge-only moderation:** Rejected because lightweight edge models may not provide sufficient accuracy for legally sensitive moderation decisions and cannot easily support larger, more sophisticated models.

* **Human-only moderation:** Rejected because manual review for all uploaded content would not scale to peak traffic volumes and would introduce unacceptable delays.

## Consequences

* Most images receive moderation decisions quickly, helping the platform meet latency requirements.

* Cloud resources are used only for ambiguous cases, reducing operational costs compared to processing all images centrally.

* The architecture becomes more complex because it requires coordination between edge services, message queues, cloud inference, and human review workflows.

## Revisit If

* The latency requirement changes significantly or edge hardware becomes capable of running large moderation models with cloud-level accuracy.
