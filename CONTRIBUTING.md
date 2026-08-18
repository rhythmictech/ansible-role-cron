# Contributing

Thanks for your interest in improving this role. Issues and pull requests are
welcome.

## Development setup

```bash
pip install ansible-core ansible-lint molecule "molecule-plugins[docker]"
```

Docker is required for Molecule tests (systemd-enabled containers).

## Before opening a PR

1. **Lint**: `ansible-lint` must pass with no violations (production profile;
   yamllint runs as part of it).
2. **Test**: `molecule test` must pass. If your change is distro-sensitive, run
   it against each supported platform:

   ```bash
   for distro in rockylinux9 rockylinux10; do
     MOLECULE_DISTRO=$distro molecule test
   done
   ```

3. **Cover new behavior**: if you add a variable or change template output,
   extend `molecule/default/converge.yml` and `molecule/default/verify.yml` to
   exercise it, and document it in the README and `meta/argument_specs.yml`.

## Conventions

- Branch from `master`; name branches `<type>/<short-description>`
  (e.g. `fix/timer-restart`).
- Use [Conventional Commits](https://www.conventionalcommits.org/)
  (`feat:`, `fix:`, `docs:`, ...).
- Keep changes backward compatible — this role is consumed by existing
  playbooks. Breaking changes need a strong justification and a major version
  bump.

CI runs lint and the full Molecule matrix on every pull request; a green build
and one approving review are required to merge.
