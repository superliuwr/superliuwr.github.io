---
title: Code Review Checklist
date: 2018-06-10 22:18:40
categories:
- Techical Leadership
tags:
- Code Review
---
# Checklist

1. Are all requirements implemented?
2. Single responsibility of a single PR. No mixed changes.
2. Is the code change the best way to implement the feature?
3. Are there any regression issues introduced in the existing features?
4. If it's a breaking change, is it well communicated?
5. Is the code change complete?
  1. Documents updated
  2. Migration included
  2. Configurations updated
  3. Other systems updated/notified
  4. Infrastructure(Database, etc.)
  5. Non-functional requirements
    1. Metrics
    2. Tracing
    3. Logging
6. Are there any major issues found?
  1. Performance
  2. Security
  3. Thread safe
  4. Edge cases
7. Are errors are gracefully handled?
8. Are there sufficient level of unit tests/e2e tests?
9. Is the code change generally readable?
  1. Can I get an understanding of the desired behavior just by doing quick scans through unit and acceptance tests?
  2. Do the methods do what the name of the method claims that they’ll do? Same for classes?
  3. Method is too long
  4. Method has too many parameters
  5. Deeply nested if/else controls
10. Are general coding best practices followed?
    1. No hard coded stuff
    2. No duplications
    3. Separate of concerns
    4. Easy to extend
    5. Comments
    6. Naming convention followed

# General Rules
1. Automating the Easy Stuff
  1. Lint
  2. Test coverage
  3. Git hooks

# Reference
1. https://medium.com/@mrjoelkemp/giving-better-code-reviews-16109e0fdd36
2. https://github.com/thoughtbot/guides/tree/master/code-review#code-review
3. https://github.com/joho/awesome-code-review