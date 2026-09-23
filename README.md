# OpenSearch boolean ad-hoc filter repro

Reproduces Grafana OpenSearch datasource boolean ad-hoc filtering with these pins:

| Component | Version |
| --- | --- |
| Grafana | `12.4.3-security-02` |
| OpenSearch datasource | `2.34.4` |
| OpenSearch | `2.19.0` |
| Seed image | `curlimages/curl:8.12.1` |

## Start

Docker Desktop must run Linux containers. From this directory:

```sh
docker compose up -d
```

First run downloads the Grafana, OpenSearch, OpenSearch Dashboards, curl, and datasource-plugin images.

- Grafana dashboard: [http://localhost:3000/d/boolean-repro](http://localhost:3000/d/boolean-repro)
- OpenSearch Dashboards: [http://localhost:5601](http://localhost:5601)


## Reproduce

1. Add ad-hoc filter `build.problem.muted = true`; repeat with `false`.
2. On **Matching document count**, open panel menu -> **Inspect** -> **Query**, then refresh.
3. Observe the filter submitted by the datasource. This repro expects `build.problem.muted:\"1\"` for `true`, or `build.problem.muted:\"0\"` for `false`.
4. Clear the ad-hoc filter. Baseline count returns to six.

The OpenSearch dropdown labels are correct because its boolean terms aggregation returns numeric `key` plus string `key_as_string`:

```sh
curl http://localhost:9200/boolean-repro/_search -H 'Content-Type: application/json' -d '{"size":0,"aggs":{"muted":{"terms":{"field":"build.problem.muted"}}}}'
```

Direct valid boolean queries each match three documents:

```sh
curl http://localhost:9200/boolean-repro/_count -H 'Content-Type: application/json' -d '{"query":{"term":{"build.problem.muted":true}}}'
curl http://localhost:9200/boolean-repro/_count -H 'Content-Type: application/json' -d '{"query":{"term":{"build.problem.muted":false}}}'
```

Confirm versions after startup:

```sh
curl http://localhost:3000/api/health
curl http://localhost:9200
docker compose exec grafana grafana cli plugins ls
```

The Compose file seeds the explicitly mapped boolean field during every clean start. Stable document IDs make a second `docker compose up -d` idempotent.

## Cleanup

```sh
docker compose down
```

The stack has no persistent volume. Cleanup removes its data; the next start seeds six documents again.
