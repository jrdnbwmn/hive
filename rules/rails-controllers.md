---
paths:
  - "app/controllers/**/*.rb"
---

# Controllers

- Thin controllers: params in, response out. Aim for ~12 lines per action.
- Business logic goes in models or service objects, never in controllers.
- Use `before_action` for shared logic (auth, loading records).
- Respond with appropriate HTTP status codes.
- RESTful actions only. Custom actions are a sign you need a new controller
  (e.g., `SubscriptionCancellationsController#create` not
  `SubscriptionsController#cancel`).
- Never `params.permit!`. Always explicit strong parameters with
  exact attribute lists.
