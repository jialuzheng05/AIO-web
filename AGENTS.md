# Project Conventions

## Project Navigation

- Add every new top-level project as the first item in `projectNavItems` in `src/App.vue`.
- Add its corresponding project value as the first item in `projects`.
- The first item in `projects` is the default project shown after a fresh page load. Keep `selectedProject` initialized from `projects[0]` instead of hard-coding a project name.
- Keep multiple prototypes for the same project grouped under one top-level navigation item. Prototype switching belongs in the page-level debug switcher.
