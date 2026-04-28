# PHASE-00-BASELINE

## Objective
Create traceable baseline bindings between frontend, automation, route, and scenario assets.

## Inputs
- Frontend repo path and default branch
- Java BDD Playwright repo path and test command
- Seed business routes and target scenarios

## Outputs
- `config/project_binding.example.yaml` adapted to project reality
- Initial `route_map` and `locator_inventory` artifacts
- One baseline execution evidence bundle

## Exit Criteria
- A locator can be traced back to scenario, step, page object, route, and frontend file.
- One smoke scenario runs and produces evidence.

## Out of Scope
- Automated healing and patch generation.
