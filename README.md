# Board Health Monitor

Stateful telemetry evaluation for development boards, test fixtures, prototypes, and deployed embedded systems.

The monitor turns timestamped metrics into debounced incidents using thresholds, ranges, changes, rates, statistical outliers, and staleness rules. It tracks recovery, computes rolling statistics, aggregates fleets, and exports Prometheus text and SARIF diagnostics.

## Quick start

```bash
npm install
npm test
node src/cli.js examples/config.json examples/telemetry.jsonl \
  --output health.json \
  --prometheus metrics.prom \
  --sarif incidents.sarif
```

```js
import {
  BoardHealthMonitor, FleetHealthMonitor, toPrometheus
} from '@opencircuitlabs/board-health-monitor';

const monitor = new BoardHealthMonitor('board-42', config);
monitor.on('incident', ({ type, incident }) => notify(type, incident));
monitor.ingest({
  timestamp: Date.now(),
  metrics: { voltage: 3.31, current: 0.12, temperature: 37 }
});
monitor.tick(Date.now()); // evaluates staleness
```

## Rule types

- `above` and `below` scalar thresholds;
- `outside` and `inside` inclusive bands;
- `changeAbove` absolute sample-to-sample movement;
- `rateAbove` absolute change per second;
- `zscoreAbove` rolling statistical anomaly with minimum-history and window controls;
- `stale` maximum telemetry age.

Each rule configures severity, activation count (`forSamples`), recovery count (`clearSamples`), and message. Consecutive-sample requirements prevent noisy readings from opening or immediately resolving incidents.

## Incident lifecycle

Incidents retain device, rule, metric, severity, opening time/value, message, status, and—after recovery—resolution time/value and duration. Fresh samples explicitly resolve active staleness incidents. History remains queryable after resolution.

Input timestamps must increase strictly per device, and metric values must be finite. History is bounded by `maxHistory` to prevent unbounded memory use.

## Fleet and observability

`FleetHealthMonitor` lazily creates isolated per-device monitors, ingests mixed-device streams, evaluates fleet staleness, and reports healthy/unhealthy counts plus device details.

Prometheus export emits board health, latest metric gauges, and active incident count with sanitized names and escaped labels. SARIF export turns currently open incidents into severity-aware diagnostics suitable for CI or operations review.

## License

MIT
