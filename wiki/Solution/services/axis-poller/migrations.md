# Axis Poller: Migrations

How initial data will be populated.

```sql
CREATE TABLE dbo.EventQueue
(
    ID         INT,
    Topic      VARCHAR(50),
    Type       VARCHAR(8) NOT NULL CHECK (Type IN ('notice', 'delta', 'document')),
    Message    TEXT,
    ReserveID  VARCHAR(36) DEFAULT NULL,
    Status     TINYINT     DEFAULT NULL,
    CreatedAt  TIMESTAMP,
    ReservedAt TIMESTAMP   DEFAULT NULL,
    CONSTRAINT EventQueue_ID PRIMARY KEY CLUSTERED (ID)
)
```

No initial records required.