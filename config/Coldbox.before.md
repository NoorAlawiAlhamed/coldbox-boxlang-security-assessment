# Coldbox.bx: before (default settings)

File: `/mycoldbox/app/config/Coldbox.bx`, inside `configure()`

```
reinitPassword          : "",
reinitKey               : "fwreinit",
handlersIndexAutoReload : true,
```

- `reinitPassword` is empty, so anyone can restart the framework with `?fwreinit=1`.
- `handlersIndexAutoReload` is on. That's useful while developing, but not needed in production.
