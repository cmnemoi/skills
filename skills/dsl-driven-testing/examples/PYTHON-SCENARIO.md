# Example: Python Scenario Builder (france-travail-api style)

Complete reference implementation of the DSL-driven testing pattern in Python.

> Source: [cmnemoi/france-travail-api — tests/](https://github.com/cmnemoi/france-travail-api/tree/main/tests)

---

## Project Structure

```
tests/
├── dsl/
│   ├── scenario.py          # Main DSL — Scenario dataclass (builder)
│   ├── http_scenario.py     # Specialized DSL for HTTP transport tests
│   └── expectations.py      # Fluent assertion helpers
├── test_doubles/
│   └── fake_http_client.py  # Handcrafted fake (no mock framework)
├── unit/                    # scenario().unit()        → FakeHttpClient
├── integration/             # scenario().integration() → Real HttpClient
└── e2e/                     # scenario().e2e()         → Full client + real creds
```

---

## The Scenario DSL

```python
from __future__ import annotations
import datetime
import os
import uuid
from dataclasses import dataclass, field
from typing import Any

from myapp.client import HttpClient, RealApiClient
from myapp.models import Offre, Scope, Sort, CodeTypeContrat
from tests.test_doubles.fake_http_client import FakeHttpClient


@dataclass
class Scenario:
    # Deterministic time control — no patching needed
    now: datetime.datetime = field(
        default_factory=lambda: datetime.datetime(2025, 12, 25, 10, 0, 0, tzinfo=datetime.UTC)
    )

    # Internal state (private)
    _http_client: HttpClient | FakeHttpClient | None = None
    _client: RealApiClient | None = None
    _captured_exception: Exception | None = None
    _offers: list[Offre] | None = None

    # -------------------------
    # Driver selection (Given)
    # -------------------------

    def unit(self) -> "Scenario":
        """In-memory driver — no network, milliseconds."""
        self._http_client = FakeHttpClient()
        return self

    def integration(self) -> "Scenario":
        """Real HTTP driver — validates transport layer."""
        self._http_client = HttpClient()
        return self

    def e2e(self) -> "Scenario":
        """Full client with real credentials from environment."""
        self._client = RealApiClient(
            client_id=os.environ["CLIENT_ID"],
            client_secret=os.environ["CLIENT_SECRET"],
            scopes=[Scope.OFFRES],
        )
        return self

    def auto(self) -> "Scenario":
        """Auto-detect highest available driver from environment."""
        if os.environ.get("CLIENT_ID"):
            return self.e2e()
        if os.environ.get("INTEGRATION"):
            return self.integration()
        return self.unit()

    # -------------------------
    # Fixture setup (Given) — Business-focused inputs
    # -------------------------

    def with_authenticated(self, expires_in: int = 1_499) -> "Scenario":
        """Pre-program a successful authentication (business language)."""
        return self._with_token(
            access_token="my_token",
            expires_in=expires_in,
        )

    def _with_token(self, access_token: str, expires_in: int) -> "Scenario":
        """Internal: programs a token response (driver detail)."""
        return self._with_http_response(status=200, body={
            "access_token": access_token,
            "expires_in": expires_in,
            "token_type": "Bearer",
            "scope": "api_offresdemploiv2",
        })

    def given_offers(self, offers: list[Offre]) -> "Scenario":
        """Provide offers directly from business domain — no HTTP details."""
        self._offers = offers
        return self

    def given_offres_from_response(self, response_body: dict) -> "Scenario":
        """Parse business data from API response format."""
        self._offers = [Offre(**data) for data in response_body.get("resultats", [])]
        return self

    def with_credentials(
        self,
        client_id: str = "client-id",
        client_secret: str = "client-secret",
        scopes: list[Scope] | None = None,
    ) -> "Scenario":
        from myapp.auth import Credentials
        self._credentials = Credentials(
            client_id=client_id,
            client_secret=client_secret,
            scopes=scopes or [Scope.OFFRES],
        )
        return self

    def with_offres_client(self) -> "Scenario":
        from myapp.offres import OffresClient
        self._offres_client = OffresClient(
            http_client=self._http_client,
            credentials=self._credentials,
            now=self.now,
        )
        return self

    def _with_http_response(self, status: int, body: dict) -> "Scenario":
        response = HTTPResponse(
            status_code=status,
            body=body,
            headers={},
            request_id=uuid.uuid4(),
        )
        self._http_client.add_response(response)
        return self

    # -------------------------
    # Actions (When)
    # -------------------------

    def when_searching_offres(self, **kwargs: Any) -> "Scenario":
        try:
            self._offers = self._offres_client.search(**kwargs)
        except Exception as e:
            self._captured_exception = e
        return self

    async def when_searching_offres_async(self, **kwargs: Any) -> "Scenario":
        try:
            self._offers = await self._offres_client.search_async(**kwargs)
        except Exception as e:
            self._captured_exception = e
        return self

    # -------------------------
    # Assertions (Then) — Business-focused outputs
    # -------------------------

    def then_all_offers_are(self, expected_type: type) -> "Scenario":
        assert self._offers is not None, "No offers captured — did you call when_searching_offres?"
        assert all(isinstance(offer, expected_type) for offer in self._offers), \
            f"Expected all offers to be {expected_type.__name__}"
        return self

    def then_offers_count(self, expected_count: int) -> "Scenario":
        """Assert on business outcome — number of offers found."""
        assert self._offers is not None, "No offers captured"
        assert len(self._offers) == expected_count, \
            f"Expected {expected_count} offers, got {len(self._offers)}"
        return self

    def then_first_offer_has(self, **expected_fields: Any) -> "Scenario":
        """Assert on domain entity fields — no HTTP/URL details."""
        assert self._offers and len(self._offers) > 0, "No offers to check"
        offer = self._offers[0]
        for field, expected in expected_fields.items():
            actual = getattr(offer, field, None)
            assert actual == expected, \
                f"Expected {field}={expected!r}, got {actual!r}"
        return self

    def then_raises(self, exception_type: type, match: str | None = None) -> "Scenario":
        """Assert on business exception — not HTTP status."""
        assert self._captured_exception is not None, "No exception was captured"
        assert isinstance(self._captured_exception, exception_type), \
            f"Expected {exception_type.__name__}, got {type(self._captured_exception).__name__}"
        if match:
            assert match in str(self._captured_exception)
        return self

    # -------------------------
    # Legacy assertions (Driver-internal — avoid in new code)
    # -------------------------

    def then_last_get_url_contains(self, expected: str) -> "Scenario":
        """⚠️ Driver-internal — checks HTTP URL. Prefer business assertions."""
        assert self._http_client.last_get_url is not None, "No GET request was made"
        assert expected in self._http_client.last_get_url, \
            f"Expected URL to contain '{expected}', got: {self._http_client.last_get_url}"
        return self

    # -------------------------
    # Cleanup
    # -------------------------

    def close(self) -> None:
        if self._client:
            self._client.close()

    async def close_async(self) -> None:
        if self._client:
            await self._client.close_async()


def scenario() -> Scenario:
    """Factory function — entry point for all tests."""
    return Scenario()
```

---

## The FakeHttpClient

```python
from __future__ import annotations
from dataclasses import dataclass, field


@dataclass
class HTTPResponse:
    status_code: int
    body: dict
    headers: dict
    request_id: object


class FakeHttpClient:
    """Handcrafted fake — not a mock. Has real state and behavior."""

    def __init__(self) -> None:
        self.responses: list[HTTPResponse] = []
        self.last_get_url: str | None = None
        self.last_post_url: str | None = None
        self.last_post_body: dict | None = None

    def add_response(self, response: HTTPResponse) -> None:
        self.responses.append(response)

    def get(self, url: str, headers: dict | None = None) -> HTTPResponse:
        self.last_get_url = url
        return self.responses.pop(0)

    async def get_async(self, url: str, headers: dict | None = None) -> HTTPResponse:
        self.last_get_url = url
        return self.responses.pop(0)

    def post(self, url: str, body: dict, headers: dict | None = None) -> HTTPResponse:
        self.last_post_url = url
        self.last_post_body = body
        return self.responses.pop(0)

    async def post_async(self, url: str, body: dict, headers: dict | None = None) -> HTTPResponse:
        self.last_post_url = url
        self.last_post_body = body
        return self.responses.pop(0)
```

---

## Expectations DSL (Fluent Assertions)

```python
from typing import Any
import pytest


class Expectation:
    def __init__(self, value: Any) -> None:
        self._value = value

    def to_equal(self, expected: Any) -> None:
        assert self._value == expected, f"Expected {expected!r}, got {self._value!r}"

    def to_contain(self, expected: Any) -> None:
        assert expected in self._value, f"Expected {expected!r} to be in {self._value!r}"

    def to_be_instance_of(self, expected_type: type) -> None:
        assert isinstance(self._value, expected_type), \
            f"Expected {expected_type.__name__}, got {type(self._value).__name__}"

    def to_all_be_instance_of(self, expected_type: type) -> None:
        assert all(isinstance(item, expected_type) for item in self._value), \
            f"Expected all items to be {expected_type.__name__}"

    def to_raise(self, exception_type: type, match: str | None = None) -> None:
        with pytest.raises(exception_type, match=match):
            self._value()

    async def to_raise_async(self, exception_type: type, match: str | None = None) -> None:
        with pytest.raises(exception_type, match=match):
            await self._value()


def expect(value: Any) -> Expectation:
    return Expectation(value)
```

---

## Tests — All Three Levels

### Unit test (in-memory, < 10ms)

```python
# ✅ Business-focused DSL — no driver internals
def test_should_search_job_offers_with_filters() -> None:
    flow = (
        scenario()
        .unit()
        .with_authenticated()  # Business language
        .given_offres_from_response({"resultats": []})  # Business input
        .with_credentials(client_id="client-id", client_secret="client-secret")
        .with_offres_client()
    )

    flow.when_searching_offres(
        mots_cles="développeur python",
        sort=Sort.DATE_CREATION,
        departement="75",
        type_contrat=CodeTypeContrat.CDI,
    )

    flow.then_offers_count(0)  # Business outcome
    flow.then_first_offer_has(titre="Développeur Python")  # Domain entity check
```
```

### Integration test (real HTTP)

```python
@pytest.mark.integration
def test_should_search_offers_with_real_http() -> None:
    flow = (
        scenario()
        .integration()
        .with_live_credentials(scopes=[Scope.OFFRES])
        .with_offres_client()
    )

    flow.when_searching_offres(mots_cles="développeur", departement="75", range_param="0-4")
    flow.then_all_offers_are(Offre)

    flow.close()
```

### E2E test (full client with real credentials)

```python
@pytest.mark.e2e
def test_should_find_job_offers() -> None:
    flow = scenario().e2e()

    flow.when_searching_offres(mots_cles="developpeur", range_param="0-2")
    flow.then_all_offers_are(Offre)

    flow.close()
```

---

## conftest.py — Single Suite, Multiple Drivers

```python
import os
import pytest
from tests.dsl.scenario import Scenario


def pytest_addoption(parser):
    parser.addoption(
        "--driver",
        default="unit",
        choices=["unit", "integration", "e2e", "auto"],
        help="Test driver level",
    )


@pytest.fixture
def scenario(request):
    driver = request.config.getoption("--driver")
    s = Scenario()
    return getattr(s, driver)()


# Or: parametrize over all available drivers
def available_drivers() -> list[str]:
    drivers = ["unit"]
    if os.environ.get("INTEGRATION"):
        drivers.append("integration")
    if os.environ.get("CLIENT_ID"):
        drivers.append("e2e")
    return drivers


@pytest.fixture(params=available_drivers())
def multi_driver_scenario(request):
    s = Scenario()
    return getattr(s, request.param)()
```

```bash
# Usage
pytest                       # unit (default)
pytest --driver=integration  # real HTTP
pytest --driver=e2e          # full E2E
pytest --driver=auto         # auto-detect from env vars
```
