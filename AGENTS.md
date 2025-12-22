# Rhseung's Design System, RDS

## Makers' Principle

See [Makers' Principle](./PRINCIPLE.md)

## Commit messages

Commit messages should follow this format:

```
<type>(<domain>): <title>
```

- Use the imperative mood for `<title>` (e.g. add, fix, update)


### `<type>` is one of the following:

- `feat`: Introduces a new feature or capability
- `fix`: Fixes a bug or incorrect behavior
- `docs`: Documentation-only changes
- `style`: Code style or formatting changes with no behavior impact
- `refactor`: Code changes that improve structure without changing behavior
- `test`: Adds or updates tests
- `chore`: Maintenance tasks that don’t affect runtime behavior
- `ci`: Changes to CI/CD configuration

### `<domain>` is one of the following:

- `frontend`: User-facing UI and client-side logic
- `backend`: Server-side application logic and APIs
- `mobile`: Mobile application code
- `infra`: Infrastructure, deployment, and environment setup
- `data`: Data models, databases, and data pipelines
- `design`: Design systems and UX-related work
- `product`: Product requirements, specs, and planning
- `ai`: AI-assisted development, agent rules, prompts, or model-related work

### Examples

feat(frontend): add onboarding flow  
fix(backend): handle null user session  
docs(ai): add AGENTS.md  
chore(infra): update deployment config  
ci(infra): optimize CI pipeline  

## Pull request titles

Pull request titles must follow the same convention as commit messages:

```
<type>(<domain>): <title>
```

- Use a single primary type per PR
- Ensure the `<type>` and `<domain>` match the associated Linear issue

### Examples

feat(frontend): add onboarding flow  
fix(backend): resolve session timeout issue  
docs(ai): introduce agent workflow guidelines  
chore(infra): clean up deployment scripts  
ci(infra): speed up CI jobs