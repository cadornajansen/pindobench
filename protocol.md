# PindoBench Paper 1 Protocol

## Working title

**A Comparative Evaluation of GUI Grounding Models for Low Latency Desktop Assistance**

## Status

Protocol version: 0.1  
Status: pre-data-collection draft  
Repository: https://github.com/cadornajansen/pindobench

This protocol is frozen before the final benchmark results are collected. Any later change must be recorded in the changelog below and labeled as a protocol deviation.

## 1. Study objective

This study evaluates the practical tradeoff between grounding accuracy, latency, cost, and failure behavior for systems that locate desktop UI targets from screenshots and natural-language instructions.

The intended application is a human-in-the-loop desktop tutor: the system identifies and explains the next UI action, while the human performs the action.

## 2. Research questions

### RQ1

How accurately do evaluated GUI grounding systems locate the intended desktop UI target?

Primary outcomes:

- Point-in-box accuracy
- Semantic target accuracy

### RQ2

How do evaluated systems differ in response latency?

Primary outcomes:

- Median latency, p50
- Tail latency, p95

### RQ3

What accuracy-latency-cost tradeoffs emerge for real-time desktop assistance?

Primary outcomes:

- Accuracy-cost comparison
- Accuracy-latency comparison
- Practical operating-point analysis

### RQ4

What failure modes most often produce incorrect or unusable grounding?

Primary outcome:

- Categorized error frequency and rate

## 3. Study design

This is a quantitative controlled comparative evaluation.

The independent variable is the grounding system or architecture.

Dependent variables are:

- Point-in-box accuracy
- Semantic target accuracy
- Coordinate error
- Response latency
- Estimated cost
- Invalid-response rate
- Failure category

Controlled variables include:

- Identical screenshot
- Identical instruction
- Identical target annotation
- Identical image encoding and dimensions
- Identical prompt contract where technically possible
- Identical coordinate normalization rules
- Identical validation rules
- Identical repetition policy

Hosted models and local models will be reported separately when their cost and latency conditions are not directly comparable.

## 4. Benchmark unit

One benchmark sample consists of:

1. One screenshot
2. One natural-language instruction
3. One intended semantic target
4. One ground-truth bounding box
5. One coordinate-space specification
6. Application and environment metadata

A sample must represent a target that a user could reasonably be instructed to locate or click.

## 5. Initial benchmark scope

The first pilot will use a small manually reviewed set before expansion.

Candidate application categories:

- Productivity software
- Code editors
- Creative tools
- Web applications
- Dialogs and menus

The final application list, application versions, display resolution, and Windows display scaling must be recorded in the benchmark manifest.

The full benchmark should contain at least 30 samples before pilot conclusions are treated as meaningful. The target for the first complete release is 100 or more samples distributed across applications and target types.

## 6. Target taxonomy

Each sample should be assigned one or more target types:

- Text button
- Icon button
- Tab
- Menu item
- Toolbar control
- Dialog control
- Sidebar or panel control
- Input field
- Checkbox, toggle, or radio control
- Canvas or timeline target
- Other, with an explicit description

Target difficulty should also be recorded:

- Easy: large, isolated, clearly labeled target
- Medium: target is readable but surrounded by competing controls
- Hard: small, icon-only, dense, ambiguous, or visually complex target

## 7. Ground truth

Ground truth must include:

- Target bounding box in screenshot pixel coordinates
- Target center point
- Semantic label
- Application name
- Optional UI region or parent context
- Annotation notes for ambiguous or irregular targets

The target bounding box should include the usable clickable area, not only the visible glyph or text.

If a target has no meaningful rectangular clickable area, the annotation must explain the chosen point and evaluation rule.

## 8. Systems under evaluation

Every system must be identified by:

- Provider
- Exact model identifier
- Access method
- Model access date
- Prompt version
- Image format and dimensions
- Inference settings
- Timeout
- Retry policy
- Local hardware, if applicable

The initial comparison may include:

- Current Pindo Nova grounding path
- OpenRouter-hosted multimodal grounding path
- Additional hosted or local systems added only after their exact configuration is documented

Native Computer Use APIs and ordinary screenshot-to-coordinate prompting must be labeled as different system types. A model must not be described as using native Computer Use unless the native tool interface was actually used.

## 9. Request contract

Each system must receive:

- The benchmark instruction
- The benchmark screenshot
- The system-specific prompt required by its API

The normalized output must be converted into this common representation:

```json
{
  "point": { "x": 0.0, "y": 0.0 },
  "box": { "x": 0.0, "y": 0.0, "width": 0.0, "height": 0.0 },
  "label": "target label",
  "raw_response": "preserved separately"
}
```

Coordinates are normalized to the screenshot dimensions:

- x range: 0 to 1
- y range: 0 to 1

A system may return a point, a box, or both. If both are returned, the point must lie inside the returned box to be considered internally consistent.

## 10. Experimental procedure

For each system and sample:

1. Load the exact benchmark screenshot.
2. Start the request timer immediately before dispatch.
3. Send the instruction and screenshot.
4. Stop the timer when a response is received or the request reaches timeout.
5. Preserve the raw response without modification.
6. Parse and validate the normalized output.
7. Convert the prediction into screenshot pixel coordinates.
8. Compare the prediction against the ground-truth annotation.
9. Record the result, failure category, latency, and cost metadata.

The initial pilot should use at least 3 repetitions per system and sample. The full benchmark should use enough repetitions to report stable latency distributions and to distinguish deterministic from variable model behavior.

Requests should be randomized by sample order where practical. Warm-up requests, retries, timeouts, and provider errors must be logged rather than silently discarded.

## 11. Evaluation metrics

### 11.1 Point-in-box accuracy

A prediction is correct when the predicted point lies within the ground-truth target bounding box.

Report:

```text
correct predictions / valid evaluated predictions
```

Also report the denominator and invalid-response count.

### 11.2 Semantic target accuracy

A prediction is semantically correct when it identifies the intended control. This may be determined from a returned label, returned box, or manual adjudication of the raw response.

Semantic adjudication rules must be documented before final analysis.

### 11.3 Coordinate error

Report Euclidean distance from the predicted point to the center of the ground-truth box.

Report both:

- Pixel distance
- Distance normalized by screenshot diagonal

### 11.4 Latency

Report:

- p50
- p95
- Mean only as a supplementary statistic
- Number of successful, failed, timed-out, and retried requests

The paper must state whether image encoding, network time, queueing, parsing, and validation are included.

### 11.5 Cost

Estimate cost per 1,000 requests using the provider pricing and image or token assumptions recorded at the time of the experiment.

For self-hosted systems, report:

- Hardware
- VRAM
- Throughput
- Infrastructure cost assumption
- Whether startup or loading time is included

### 11.6 Invalid-response rate

Count:

- Malformed output
- Missing coordinate
- Out-of-range coordinate
- Refusal or no-answer response
- Timeout
- Provider or infrastructure error
- Validation failure

## 12. Failure taxonomy

Use these categories initially:

- Small or dense control
- Duplicate or ambiguous label
- Wrong application region
- Wrong panel or dialog
- Semantic target correct but coordinate outside box
- Coordinate-space or scaling error
- Unreadable text
- Icon interpretation error
- Invalid output
- Timeout or provider failure
- Other, with explanation

A sample may receive one primary failure category and optional secondary tags.

## 13. Analysis plan

The primary analysis will compare systems using:

- Accuracy with exact denominators
- 95 percent confidence intervals where sample size permits
- Paired per-sample comparisons
- Median and p95 latency
- Cost per 1,000 requests
- Failure-category distributions

Do not declare a universal winner from a single metric. The discussion must report tradeoffs and state the tested operating conditions.

Exploratory analyses, such as foreground-window crops or UI Automation plus vision fallback, must be labeled separately from the primary model comparison.

## 14. Privacy and licensing

Screenshots must not contain:

- API keys
- Passwords
- Private messages
- Personal identifiers
- Sensitive documents
- Unlicensed material that cannot be redistributed

If a screenshot cannot be released, publish its metadata and evaluation results only, and document the restriction.

Raw model responses must be scrubbed for secrets before publication.

## 15. Reproducibility artifacts

The repository should contain:

- This protocol
- Benchmark manifest
- Screenshots or release restrictions
- Ground-truth annotations
- Canonical prompts
- Raw model responses
- Timing logs
- Processed results
- Evaluation scripts
- Environment information
- Figures and source data
- References

Every reported result must identify the repository commit, experiment identifier, model identifier, and benchmark version.

## 16. Protocol deviations

Record any change made after data collection begins.

| Date | Section | Change | Reason | Effect on analysis |
|---|---|---|---|---|
| [date] | [section] | [change] | [reason] | [effect] |

## 17. Changelog

| Version | Date | Change |
|---|---|---|
| 0.1 | 2026-09-23 | Initial protocol for Paper 1 |
