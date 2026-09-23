# PindoBench

PindoBench is an independent benchmark for evaluating GUI grounding systems for low-latency desktop assistance.

The first study measures whether a model can locate the intended desktop UI target from the same screenshot and natural-language instruction under controlled conditions.

## Paper 1

**A Comparative Evaluation of GUI Grounding Models for Low Latency Desktop Assistance**

Study type:

- Quantitative empirical evaluation
- Controlled comparative experiment
- AI systems and benchmark research

## Current status

- [x] Repository initialized
- [x] Research protocol drafted
- [x] Dataset and result schemas defined
- [ ] Benchmark samples collected
- [ ] Evaluation runner implemented
- [ ] Pilot experiment completed
- [ ] Full comparison completed

## Repository structure

```text
protocol.md
benchmark/
  manifest/
  screenshots/
  annotations/
experiments/
  raw-results/
  processed-results/
evaluation/
figures/
references/
schemas/
  sample.schema.json
  result.schema.json
```

## Reproducibility rule

The protocol, benchmark samples, prompts, model identifiers, raw responses, timing logs, and evaluation scripts must be versioned. Results must always identify the repository commit and configuration used to generate them.

## Scope boundary

This study evaluates screen-based GUI grounding for human-in-the-loop desktop assistance. It does not claim to measure complete autonomous computer-use performance or learning outcomes with human participants.
