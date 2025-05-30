lcbo database with signoz demo

```bash
ansible-playbook up.yml
```

This does docker compose up on an "empty" signoz in a codespace. Use this as a starting point for instrumenting your app. It also creates a postgresql/pgadmin container with an old version of the LCBO database in it.

```bash
ansible-playbook down.yml
```

This does docker compose down on the `docker-compose.yml` (the same docker-compose file from up.yml)






# INFO8985 Database Monitoring with SigNoz and OpenTelemetry

This project integrates OpenTelemetry `sqlqueryreceiver` and `postgresqlreceiver` into a SigNoz-based observability system for monitoring PostgreSQL metrics and custom SQL queries.

## ✅ Project Setup Overview

### 📦 Technologies Used
- Docker + Docker Compose
- PostgreSQL
- SigNoz (OpenTelemetry-based)
- Otel Collector
- ClickHouse (backend for SigNoz)
- Custom SQL metric collection via `sqlqueryreceiver`

---

## 📁 Directory Structure
```
.
├── docker-compose.yml
├── signoz/
│   ├── otel-collector-config.yaml
│   ├── docker-compose-minimal.yaml
│   └── prometheus.yml
└── ...
```

---

## 🛠️ Configuration Summary

### 1. Enabled Receivers in `otel-collector-config.yaml`

#### `sqlqueryreceiver`
```yaml
sqlquery:
  driver: "postgres"
  datasource: "host=db user=devtedsuser password=devtedspass dbname=demodb sslmode=disable"
  queries:
    - sql: "SELECT COUNT(*) AS count FROM inventories WHERE product_id = 18 AND quantity = 0"
      metrics:
        - metric_name: sqlquery.inventories.product_18_out_of_stock
          value_column: count
          value_type: int
          description: "Out-of-stock count for product 18"
          unit: "1"
```

#### `postgresqlreceiver`
```yaml
postgresql:
  endpoint: db:5432
  username: devtedsuser
  password: devtedspass
  databases: [demodb]
  tls:
    insecure: true
```

### 2. Added Both Receivers to Pipeline
```yaml
metrics:
  receivers: [otlp, postgresql, sqlquery]
  processors: [batch]
  exporters: [clickhousemetricswrite]
```

---

## 🧪 Testing Steps

1. Connected to PostgreSQL container:
```bash
docker exec -it inclass2-db-1 psql -U devtedsuser -d demodb
```
2. Created the required table:
```sql
CREATE TABLE inventories (
  product_id INT PRIMARY KEY,
  quantity INT NOT NULL
);
```
3. Inserted test data:
```sql
INSERT INTO inventories (product_id, quantity) VALUES (18, 0), (19, 5), (20, 0);
```

4. Validated metric: `sqlquery.inventories.product_18_out_of_stock` in SigNoz panel.
5. Verified PostgreSQL metrics like inserts, connections via [Postgres overview dashboard](https://signoz.io/dashboards/).

---

## 📊 Dashboard Setup in SigNoz

- ✅ Imported [PostgreSQL dashboard](https://signoz.io/dashboards/)
- ✅ Created custom panel in SigNoz to show `sqlquery.inventories.product_18_out_of_stock`

---

## 🎯 Conclusion

The integration of `sqlqueryreceiver` and `postgresqlreceiver` successfully provides both system-level and business-level observability for the PostgreSQL `demodb` instance.