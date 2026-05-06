# Skill: add-api-auth (Symfony)
# See rules/symfony.md for Symfony-specific patterns.
# See skills/shared/ for universal patterns that apply here.

## When to use
Add Api Auth in a Symfony project.

## Checklist
- [ ] Follow the pattern in rules/symfony.md
- [ ] Use Symfony DI container — autowire via constructor
- [ ] Validate with #[Assert\*] attributes — never manual array checks
- [ ] Test with PHPUnit WebTestCase or KernelTestCase
- [ ] Run: ./vendor/bin/php-cs-fixer fix && php bin/phpunit
