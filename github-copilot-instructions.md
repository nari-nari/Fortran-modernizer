# Copilot Instructions

## Project purpose

This repository investigates and develops a modern lubrication-analysis solver using an existing legacy Fortran solver as an important reference.

The long-term target may include Python, Firedrake FEM, automatic differentiation, shaft equilibrium, thermal coupling, mass-conserving cavitation, and connection to a separate topology-optimization project.

The detailed migration and architecture strategy has not yet been decided.

Do not assume in advance that the legacy Fortran should either be fully translated or fully rewritten.

## Legacy code

Treat the legacy Fortran code as an important source of:

- existing behavior,
- physical and numerical models,
- implementation knowledge,
- regression results.

When analyzing legacy code, distinguish confirmed facts from interpretation.

Do not silently guess the physical meaning of unfamiliar variables, equations, modes, or empirical formulas.

## Evidence

For technical conclusions, record concrete evidence where practical, such as source file, procedure, variable, input option, or reference.

Distinguish:

- confirmed from code,
- confirmed from reference,
- inferred,
- requires verification,
- requires human confirmation.

## Literature

Published literature and benchmark data may be used to understand or validate the implementation.

Whenever external literature materially influences an implementation or validation decision, record the source and what information was taken from it.

Prefer traceable references such as DOI, publication information, equation, table, figure, or page where practical.

Do not introduce undocumented empirical formulas when their origin can reasonably be identified.

## Verification

Do not treat agreement with the legacy Fortran solver alone as sufficient proof of correctness.

Where appropriate, distinguish:

- legacy regression,
- mathematical/numerical verification,
- literature or benchmark validation,
- derivative verification such as Taylor tests.

## Development

Use the repository's Superpowers / Agent Skills when appropriate.

For substantial changes, investigate and plan before implementation.

Prefer small, testable, reviewable changes.

Do not modify legacy code merely to make analysis easier unless the task explicitly requires it.

Keep generated implementation decisions and documentation consistent with evidence discovered during the project.