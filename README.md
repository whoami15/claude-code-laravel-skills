# Laravel Skills

**A curated collection of AI Skills for modern Laravel development.**

[What is this?](#what-is-this) • [Skills](#skills) • [Contributing](#contributing)

---

## What is this?

A collection of [Laravel Skills](https://skills.laravel.cloud) that teach AI agents how to use popular packages and patterns in modern Laravel applications (Laravel 11+/12+, PHP 8.2+).

Each skill provides focused, practical guidance - complete with real code examples, best practices, and common anti-patterns - so AI agents can generate correct, idiomatic code from the start.

## Skills

| Skill  | Description |
|--------|-------------|
| [saloon-for-laravel](saloon-for-laravel/SKILL.md) | Build elegant API integrations with SaloonPHP - connectors, requests, responses, authentication, testing, pagination, caching, rate limiting, OAuth2, and SDK patterns |

## Contributing

Contributions are welcome! Whether it's:

- Improvements to existing skills
- New skills for Laravel packages not yet covered
- Bug fixes in code examples
- Discussion about patterns and best practices

### Adding a New Skill

1. Create a new directory under `skills/` with your skill name
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`) and markdown content
3. Follow the structure of existing skills - see [saloon-for-laravel](saloon-for-laravel/SKILL.md) as a reference
4. Open a pull request

### Skill Structure

Each skill should include:

- **Context** - What packages/tools does the skill cover?
- **Rules** - Clear, actionable instructions for the AI agent
- **Examples** - Real code samples showing the correct approach
- **Anti-patterns** - Common mistakes to avoid
- **References** - Links to official documentation

```
skills/
└── saloon-for-laravel/
    └── SKILL.md                       # Skill definition
```

## References

- [Laravel Skills Platform](https://skills.laravel.cloud)
- [Laravel Skills Documentation](https://skills.laravel.cloud/docs)

## License

MIT License. See [LICENSE](LICENSE.md) for details.
