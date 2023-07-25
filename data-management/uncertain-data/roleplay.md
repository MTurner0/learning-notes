# [Roleplay: board games as interfaces to probabilistic databases](https://www.hytradboi.com/2022/roleplay-board-games-as-interfaces-to-probabilistic-databases)

## About

Graphical generation of probabilistic models.

**Media type:** Talk, [HYTRADBOI](https://www.hytradboi.com/)

**Year:** 2022

## Summary

- Most data visualization tools:

    - Tables (which represent state) can be edited.

    - Legends (which map tables to visual marks) _may_ be edited.

    - Visual marks are not directly edited.

- **Roleplay**:

    - Marks (which represent state) are drawn on the canvas.

    - Legends (which assign meaning to marks) are user-specified.

    - Marks are mapped to tables, _interpreting_ the visualizations.

- Once a few marks establish a pattern, Roleplay can generate random samples from what it assumes to be the underlying distribution.

- Differential dataflow

    - *Core state* (including UI state) represented by normalized relational.

    - Gestures and keystrokes transformed into diffs: `[(old_lens, -1), (new_lens, 1)]`

    - Custom diff-taking renderer draws visual objects using *tabletop dataflow*.

    - *Recognizer dataflow* applies lens definitions (queries) to the current state of the board.

