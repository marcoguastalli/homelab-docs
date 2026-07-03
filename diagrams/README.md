# Diagrams

Sources and exports for every non-ASCII diagram referenced from the
documentation.

## Rules

1. Commit the **editable source** (`.drawio` or `.excalidraw`) and the
   **exported `.svg`** together, same basename, same PR.
2. Documents embed only the `.svg` (renders on GitHub).
3. Naming: `<area>-<topic>.{drawio,svg}` — e.g. `network-topology.svg`,
   `cicd-pipeline.svg`.
4. ASCII diagrams inside markdown documents are preferred while they stay
   readable — a file lands here when ASCII stops being enough.

## Wanted (not yet drawn)

- `network-topology` — LAN/VPN/Docker networks with real addresses
- `deploy-flow` — PR → CI → runner → health → rollback sequence
