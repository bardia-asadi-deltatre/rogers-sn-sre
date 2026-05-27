# Rogers Sportsnet+ — AWS Compute Inventory
_Last updated: May 2026_

## Three AWS Accounts

| Alias | Account ID | Contents |
|-------|-----------|----------|
| aws-rogers-diva | 321254714924 | EKS (divabo, vi), self-hosted MongoDB, RabbitMQ, mediaservices |
| aws-rogers-axis-torino | 388700888837 | EKS (forge prd/stg/dev), forge MongoDB, entitlement nodes |
| aws-rogers-discovery-axis | 878223349253 | ISL OData EC2 fleet, ESB, DC, RDGW, ISL integration, staging MongoDB |

---

## Account: aws-rogers-diva (321254714924)

### EKS Clusters
| Cluster | Nodegroup | Instance type | Min/Max/Desired |
|---------|-----------|--------------|-----------------|
| rogers-divabo-prd-cluster | rogersDivaBOprdnodegroup | c6a.2xlarge | 1/6/1 |
| rogers-vi-prd-cluster | rogersVIprdnodegroup | m6a.xlarge | 1/6/1 |

### EC2 — Standalone
| Name | Type | Product | Notes |
|------|------|---------|-------|
| rogers-main-prd-jumpbox | t3a.small | main | Jumpbox only |
| rogers-cac1-mediaservices-prd-tag-vs | c6a.8xlarge | mediaservices | Video/transcoding server, 32 vCPUs |
| rogers-prd-cac1-divabo-mongodb1/2/3 | m6i.xlarge ×3 | divabo | Self-hosted Mongo replica set |
| rogers-prd-cac1-vi-core-mongodb1/2/3 | m6a.large ×3 | vi-core | Self-hosted Mongo replica set |
| rogers-prd-cac1-vi-core-rabbitmq1/2/3 | m6a.large ×3 | vi-core | RabbitMQ cluster |

---

## Account: aws-rogers-axis-torino (388700888837)

### EKS Clusters
| Cluster | Nodegroup | Instance type | Min/Max/Desired |
|---------|-----------|--------------|-----------------|
| rogers-prd-cc1-eks1 | main | m6i.2xlarge | 7/10/7 |
| rogers-prd-cc1-eks1 | entitlement | c6i.4xlarge | 4/90/7 |
| rogers-stg-cc1-eks1 | nodepool1 | t3.xlarge | 3/3/3 |
| rogers-stg-cc1-eks1 | entitlement | c6i.4xlarge | 4/90/7 |
| rogers-dev-cc1-eks1 | default | t3.xlarge | 3/3/3 |

### EC2 — Standalone (MongoDB for forge product)
| Name | Type | Env |
|------|------|-----|
| rogers-prd-mongo-db1/2/3 | m6i.xlarge ×3 | production |
| rogers-prd-mongo-log-db1/2/3 | t3.xlarge ×3 | production |
| rogers-stg-mongo-db1/2/3 | t3.xlarge ×3 | staging |
| rogers-dev-mongo-db1 | t3.xlarge ×1 | development |

---

## Account: aws-rogers-discovery-axis (878223349253)

### EC2 — Fleet (ECS EC2 backing nodes, no ECS cluster/service data captured)
| Name pattern | Count | Env | Notes |
|-------------|-------|-----|-------|
| production-isl-odata-host | ~50+ | production | Massive ISL OData fleet |
| stag-isl-odata-host | few | staging | |
| test-isl-odata-host | few | test | |
| production-isl-integration-host | multiple | production | |
| stag-isl-integration-host | few | staging | |
| test-isl-integration-host | few | test | |
| production-core / stag-core / test-core | 1 each | all | |
| production-dc1/dc2 / stag-dc1/dc2 / test-dc1/dc2 | 2 each | all | Domain controllers |
| production-rdgw / stag-rdgw / test-rdgw | 1 each | all | Remote desktop gateway |
| production-esb-0/1 / stag-esb-0 / test-esb-0 | few | all | Enterprise service bus |

### EC2 — Staging/Test MongoDB
| Name | Env |
|------|-----|
| rogers-diva-stag8-mongo-catalog-1/2/3 | staging |
| rogers-diva-stag8-geo-api-mongo-catalog-1 | staging |
| rogers-diva-test8-mongo-catalog-1 | test |
| rogers-diva-test8-geo-api-mongo-catalog-1 | test |

---

## Notes
- ECS clusters/services returned empty in all three account inventories — either not tagged or not discoverable via the script used
- Tag inconsistency: some resources use `env:prd`, others `env:production` — this affects DD monitor scoping
- DD agent status on standalone EC2 (MongoDB, RabbitMQ, ISL OData hosts, mediaservices) is unknown — needs verification before host-level monitors can be created
