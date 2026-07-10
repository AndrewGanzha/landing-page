Two days ago, I was exploring the templates repository on our company's GitLab and found something that I believe should be replaced with a more modern solution.

Currently, we use the `os` library to access environment variables. The question is: how can we migrate this legacy approach to something better?

We work extensively with AI agents such as Codex, Claude, and others. When we ask them to generate functionality for working with environment variables in `.env` files, they usually rely on the `os` library because it has been the standard and most widely adopted approach for many years.

However, the fact that a solution is older and more popular is not, by itself, a good reason to keep using it. When I discovered the `pydantic-settings` package, I became interested in adopting it in our FastAPI application templates.

The next challenge was how to tell AI agents, "Don't use `os.environ`; use `pydantic-settings` instead"—without repeating the same instruction in every prompt. The answer is simple: provide the right context through well-designed instructions and reusable skills.

I spent only five minutes refactoring this part. First, I added a new skill with the following instruction:

`When working with environment variables, replace the usage of os.environ with pydantic-settings.`

A few moments later, I had a new reusable skill ready for my repository and updated the `config.py` template accordingly.

```python
%% before %%
import os
from functools import lru_cache

from backend_auth.config import AuthSettingsFields
from backend_shared.config import CorsSettingsFields, EnvName, ServerSettingsFields
from fastapi_observability_logging import ObservabilitySettingsFields
from pydantic_settings import BaseSettings, SettingsConfigDict

from src.api.config.api import ApiSettingsFields
from src.api.config.database import DEFAULT_DATABASE_URL, DatabaseSettingsFields
from src.api.config.service import ServiceSettingsFields


class BaseAppSettings(
    ServiceSettingsFields,
    ApiSettingsFields,
    ServerSettingsFields,
    DatabaseSettingsFields,
    CorsSettingsFields,
    ObservabilitySettingsFields,
    AuthSettingsFields,
    BaseSettings,
):
    model_config = SettingsConfigDict(
        env_prefix="APP_",
        env_file=".env",
        env_file_encoding="utf-8",
        extra="ignore",
    )

class DevSettings(BaseAppSettings):
    env: EnvName = "dev"
    app_reload: bool = True

class TestSettings(BaseAppSettings):
    env: EnvName = "test"
    app_reload: bool = False
    database_url: str = DEFAULT_DATABASE_URL

class ProdSettings(BaseAppSettings):
    env: EnvName = "prod"
    app_reload: bool = False

@lru_cache
def get_settings() -> BaseAppSettings:
    env = os.getenv("APP_ENV", "dev").lower()
    if env == "test":
        return TestSettings()
    if env == "prod":
        return ProdSettings()
    return DevSettings()

```

```python
%% after %%
from functools import lru_cache

from backend_auth.config import AuthSettingsFields
from backend_shared.config import CorsSettingsFields, EnvName, ServerSettingsFields
from fastapi_observability_logging import ObservabilitySettingsFields
from pydantic_settings import BaseSettings, SettingsConfigDict

from src.api.config.api import ApiSettingsFields
from src.api.config.database import DEFAULT_DATABASE_URL, DatabaseSettingsFields
from src.api.config.service import ServiceSettingsFields


class BaseAppSettings(
    ServiceSettingsFields,
    ApiSettingsFields,
    ServerSettingsFields,
    DatabaseSettingsFields,
    CorsSettingsFields,
    ObservabilitySettingsFields,
    AuthSettingsFields,
    BaseSettings,
):
    model_config = SettingsConfigDict(
        env_prefix="APP_",
        env_file=".env",
        env_file_encoding="utf-8",
        extra="ignore",
    )


class EnvSettings(BaseSettings):
    env: EnvName = "dev"

    model_config = SettingsConfigDict(
        env_prefix="APP_",
        env_file=".env",
        env_file_encoding="utf-8",
        extra="ignore",
    )


class DevSettings(BaseAppSettings):
    env: EnvName = "dev"
    app_reload: bool = True


class TestSettings(BaseAppSettings):
    env: EnvName = "test"
    app_reload: bool = False
    database_url: str = DEFAULT_DATABASE_URL


class ProdSettings(BaseAppSettings):
    env: EnvName = "prod"
    app_reload: bool = False


_SETTINGS_MAP: dict[EnvName, type[BaseAppSettings]] = {
    "dev": DevSettings,
    "test": TestSettings,
    "prod": ProdSettings,
}


@lru_cache
def get_settings() -> BaseAppSettings:
    env = EnvSettings().env
    settings_cls = _SETTINGS_MAP.get(env, DevSettings)
    return settings_cls()
```

This is just a small improvement, but software engineering is often about accumulating small improvements over time. One refactoring may not make a noticeable difference, but if you find a hundred opportunities like this, the overall quality and maintainability of your project can improve significantly.

Never stop learning and exploring new solutions. Stay curious, challenge existing approaches, and keep experimenting. That's how you become a better developer.
