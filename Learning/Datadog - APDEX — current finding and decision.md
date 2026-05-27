
[[APDEX]] is **not a simple blanket monitor** that can be safely copied across all services.

What I found:

* APDEX depends on APM/tracing being present for the service.
* The code is instrumented for APM/tracing; APDEX is then calculated in Datadog from those traces.
* APDEX has two important values:

  * **T threshold**: the latency target, e.g. `0.5s`. This defines what "fast enough" means.
  * **Monitor threshold**: the APDEX score that triggers an alert, e.g. `< 0.7`.
* Both values are service/business dependent.
* Some services already have APDEX configured, but missing APDEX monitors may simply mean no alert exists yet, not that APDEX itself is unavailable.
* Grouping all services into one APDEX monitor is risky because it forces the same alert threshold on services with different performance expectations.

Decision for now:

> Park APDEX expansion until we understand the services better. Existing APDEX monitors can stay, but adding new APDEX alerts should be done per service after reviewing current baseline latency/APDEX behavior and confirming business expectations for that API/service.

For Day 1 coverage, prioritize more objective monitors first: CPU, memory, error rate, latency, task health, and disk where applicable.
