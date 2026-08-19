# Inventario de deploys bicom

Foto de `bicom/github` al 2026-08-19: 45 workflows de deploy repartidos en 38 repos
(se excluyen los de `node_modules/` y `vendor/`). Sirve para copiar convenciones de
nombres y para encontrar el repo hermano más parecido al que se está tocando.

Constantes en todos: región `us-east-2`, cuenta `701355173252`, credenciales
`AWS_ACCESS_KEY`/`AWS_SECRET_ACCESS_KEY`, Slack `SLACK_CHANNEL_BICOM`.

Columna **Tipo**: `ECS` servicio, `LAMBDA` función de contenedor, `ECR` solo push de imagen.
Columna **Arq**: la que pasa el workflow a `--platform`; tiene que coincidir con el
`cpuArchitecture` de la task definition (o con `Architectures` de la Lambda).

| Repo | Workflow | Rama | Tipo | Arq | ECR | Cluster | Servicio / Lambda | Task definition | Contenedor | Secreto |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| bicom-base-conta-php-fpm | master-deploy.yml | master | ECR | amd64 | bicom-imagen-base-conta-php-fpm | — | — | — | — | — |
| bicom-base-conta-php-fpm | tenant-deploy.yml | tenant | ECR | amd64 | bicom-imagen-base-conta-php-fpm | — | — | — | — | — |
| bicom-ecommerce-v2 | push-image.yml | main | ECR | arm64 | bicom-ecommerce-v2 | — | — | — | — | — |
| bicom-imagen-base-nginx | production-deploy.yml | production | ECR | amd64 | bicom-imagen-base-nginx | — | — | — | — | — |
| bicom-imagen-base-php-fpm | alpine-deploy.yml | alpine | ECR | amd64 | bicom-image-php-fpm-prod-2025 | — | — | — | — | — |
| bicom-imagen-base-php-fpm | beta-deploy.yml | beta | ECR | amd64 | bicom-image-php-fpm-prod-2025 | — | — | — | — | — |
| bicom-imagen-base-php-fpm | control-deploy.yml | control | ECR | amd64 | bicom-image-php-fpm-prod-2025 | — | — | — | — | — |
| bicom-imagen-base-php-fpm | prod-deploy.yml | production | ECR | amd64 | bicom-image-php-fpm-prod-2025 | — | — | — | — | — |
| bicom-test-aws-secrets | production.yml | production | ECR | arm64 | bicom-test-aws | — | — | — | — | prod/bicom/api-external-zXy425 |
| -bicom-ms-import-promotion | production-deploy.yml | master | ECS | arm64 | bicom-ms-import-promotion | BICOM-CONSUMERS-ARM | BICOM-SERVICE-IMPORT-PROMOTION | BICOM-IMPORT-PROMOTIONS | BICOM-CNT-IMPORT-PROMOTION | prod/bicom/consumers-NMjPni |
| bicom-api-bh-go | deploy-api.yml | production | ECS | arm64 | bicom-api-bh-v2 | BICOM-ECS-APIs | BICOM-SERVICE-BALLOT-HONORARY-API | BICOM-API-BALLOT-HONORARY | CNT-BH-API | — |
| bicom-api-dte | api-dte-ext.yml | main | ECS | arm64 | bicom-api-dte | BICOM-DTE-API | BICOM-SERVICE-API-DTE | BICOM-TD-API-DTE-EXT | CNT-DTE-API-EXT | — |
| bicom-api-dte | api-dte.yml | main | ECS | arm64 | bicom-api-dte | BICOM-DTE | BICOM-SERVICE-API-DTE | BICOM-TD-API-DTE | CNT-API-DTE | — |
| bicom-api-frontend | deploy.yml | main | ECS | amd64 | bicom-api-go-frontend | BICOM-ECS-APIs | BICOM-NEXTJS-API-SERVICE-PROD | bicom-api-go-frontend | CNT-BICOM-API-GO-FRONTEND | — |
| bicom-api-go | main.yml | main | ECS | arm64 | bicom-api-go | BICOM-ECS-APIs | BICOM-API-GO-SERVICE-PROD | bicom-td-api-go | CNT-BICOM-API-GO | prod/bicom/bicom-api-go-SXYYQi |
| bicom-api-sii | deploy-api.yml | production | ECS | arm64 | bicom-api-sii-v2 | BICOM-ECS-APIs | BICOM-SERVICE-SII-V2-API | BICOM-TD-SII-V2-API | CNT-SII-V2-API | — |
| bicom-contabilidad-front | development-deploy.yml | development | ECS | arm64 | bicom-conta-frontend-nextjs | BICOM-CONTABILIDAD | BICOM-CONTA-SERVICE-FRONTEND-NEXTJS | BICOM-CONTA-FRONTEND-NEXTJS | CNT-FRONT | prod/bicom-conta/front-nextjs-hB5Nzd |
| bicom-contabilidad-front | production-deploy.yml | production | ECS | arm64 | bicom-conta-frontend-nextjs | BICOM-CONTABILIDAD | BICOM-CONTA-SERVICE-FRONTEND-NEXTJS-PROD | BICOM-CONTA-FRONTEND-NEXTJS-PROD | CNT-FRONT | prod/bicom-conta/front-nextjs-prod-08ZfUG |
| bicom-contabilidad-laravel-backend | development-deploy.yml | development | ECS | amd64 | bicom-conta-laravel-backend | BICOM-CONTABILIDAD | BICOM-CONTA-SERVICE-BACKEND-LARAVEL-API | BICOM-CONTA-TD-BACKEND-LARAVEL-API | php-fpm | prod/bicom-conta/backend-laravel-api-2QrE0H |
| bicom-contabilidad-laravel-backend | production-deploy.yml | production | ECS | amd64 | bicom-conta-laravel-backend | BICOM-CONTABILIDAD | BICOM-CONTA-SERVICE-BACKEND-LARAVEL-API-PROD | BICOM-CONTA-TD-BACKEND-LARAVEL-API-PROD | php-fpm | prod/bicom-conta/backend-laravel-api-prod-BK5Nc3 |
| bicom-contabilidad-master | development-deploy.yml | development | ECS | amd64 | bicom-contabilidad-master | BICOM-CONTABILIDAD | BICOM-CONTA-SERVICE-MASTER | BICOM-CONTA-TD-MASTER | php-fpm | prod/bicom-conta/master-gZrr8j |
| bicom-contabilidad-master | production-deploy.yml | production | ECS | amd64 | bicom-contabilidad-master | BICOM-CONTABILIDAD | BICOM-CONTA-SERVICE-MASTER-PROD | BICOM-CONTA-TD-MASTER-PROD | php-fpm | prod/bicom-conta/master-prod-BCNqo3 |
| bicom-control-api-bh | production-deploy.yml | production | ECS | amd64 | bicom-control-apibh | BICOM-ECS-APIs | BICOM-CONTROL-SERVICE-APIBH | BICOM-TD-CONTROL-APIBH | php-fpm | prod/bicom/controlbh-PuSG6U |
| bicom-control-v2 | prod-deploy.yml | main | ECS | amd64 | bicom-control-v2 | BICOM-ECS-WEB-01 | BICOM-SERVICE-CONTROL-V2 | BICOM-TD-CONTROL-V2 | php-fpm | prod/bicom/control-v2-PchWDD |
| bicom-crt-frontend | deploy.yml | main | ECS | amd64 | bicom-certification-frontend | BICOM-ECS-WEB-01 | BICOM-NEXTJS-CERTIFICATION-SERVICE-PROD | bicom-certification-frontend | CNT-BICOM-CERTIFICATION-FRONTEND | bicom/prod/bicom-certification-frontend-KrDT3A |
| bicom-crt-stage1 | deploy.yml | main | ECS | arm64 | bicom-ms-certification-stage1 | BICOM-CONSUMERS-ARM | BICOM-MS-CERTIFICATION-STAGE1-SERVICE-PROD | BICOM-TD-MS-CERTIFICATION-STAGE1 | CNT-BICOM-MS-CERTIFICATION-STAGE1 | prod/bicom/bicom-ms-certification-stage1-LNXANb |
| bicom-crt-stage3 | deploy.yml | main | ECS | arm64 | bicom-ms-certification-stage2 | BICOM-CONSUMERS-ARM | BICOM-MS-CERTIFICATION-STAGE2-SERVICE-PROD | BICOM-TD-MS-CERTIFICATION-STAGE2 | CNT-BICOM-MS-CERTIFICATION-STAGE2 | prod/bicom/bicom-ms-certification-stage2-Rfe4vY |
| bicom-licitaciones-mp | deploy.yml | main | ECS | amd64 | bicom-mercado-publico | BICOM-ECS-APIs | BICOM-NEXTJS-MP-SERVICE-PROD | bicom-td-mp | CNT-BICOM-MP | — |
| bicom-ml-api | main.yml | main | ECS | arm64 | bicom-ml-api | BICOM-ECS-APIs | BICOM-ML-API-SERVICE-PROD | BICOM-TD-ML-API | CNT-BICOM-ML-API | prod/bicom/bicom-ml-api-8bZdDq |
| bicom-ml-sync-products | main.yml | main | ECS | arm64 | bicom-ml-sync-products | BICOM-ECS-APIs | BICOM-MS-ML-SYNC-SERVICE-PROD | BICOM-TD-MS-ML-SYNC-PRODUCTS | CNT-BICOM-ML-SYNC-PRODUCTS | prod/bicom/ml-sync-products-6N4fSF |
| bicom-ml-sync-stock | main.yml | main | ECS | arm64 | bicom-ml-sync-stocks | BICOM-ECS-APIs | BICOM-MS-ML-SYNC-STOCKS-SERVICE-PROD | BICOM-TD-MS-ML-SYNC-STOCKS | CNT-BICOM-ML-SYNC-STOCKS | prod/bicom/ml-sync-stocks-6ZiG4x |
| bicom-ms-customer-suppliers-ts | production-deploy.yml | master | ECS | amd64 | bicom-ms-customer-suppliers | BICOM-CONSUMERS-ARM | BICOM-SERVICE-CUSTOMER-SUPPLIERS | BICOM-TD-CUSTOMER-SUPPLIERS | CNT-CUSTOMER-SUPPLIERS | prod/bicom/consumers-NMjPni |
| bicom-ms-email-reader | ms-email-reader.yml | main | ECS | arm64 | bicom-ms-email-reader | BICOM-DTE | BICOM-SERVICE-MS-EMAIL-READER | BICOM-TD-MS-EMAIL-READER | CNT-MS-EMAIL-READER | — |
| bicom-ms-logistic-pdf-go | deploy-prod.yml | master | ECS | arm64 | bicom-ms-logistic-pdf-go | BICOM-ECS-APIs | BICOM-SERVICE-LOGISTIC-PDF | BICOM-TD-LOGISTIC-PDF-GO | CNT-LOGISTIC-PDF | — |
| bicom-ms-stock-alert-ts | production-deploy.yml | master | ECS | arm64 | bicom-ms-info-stock-alert-ts | BICOM-CONSUMERS-ARM | BICOM-SERVICE-CONSUMER-STOCK-ALERT | BICOM-TD-CONSUMER-STOCK-ALERT | BICOM-CNT-CONSUMER-STOCK-ALERT | prod/bicom/consumers-NMjPni |
| bicom-ms-stock-closing | production.yml | main | ECS | arm64 | bicom-ms-stock-closing | BICOM-CONSUMERS-ARM | BICOM-SERVICE-MS-STOCK-CLOSING | BICOM-TD-MS-STOCK-CLOSING | CNT-BICOM-MS-STOCK-CLOSING | prod/bicom/consumers-NMjPni |
| bicom-ms-subscription-cash | production-deploy.yml | master | ECS | amd64 | bicom-ms-subscription-coin | BICOM-CONSUMERS-ARM | BICOM-SERVICE-MS-SUB-COIN | BICOM-TD-MS-SUB-COIN | CNT-MS-COIN | ms/bicom/subscription-coin-zBIfKu |
| bicom-report-agreements-docs | production-deploy.yml | master | ECS | arm64 | bicom-report-agreements-docs | BICOM-CONSUMERS-ARM | BICOM-SERVICE-REPORT-AGREEMENTS-DOCS | BICOM-TD-REPORT-AGREEMENTS-DOCS | CNT-REPORT-AGR-DOC | prod/bicom/consumers-NMjPni |
| bicom-report-cash | production-deploy.yml | master | ECS | arm64 | bicom-report-cash | BICOM-CONSUMERS-ARM | BICOM-SERVICE-REPORT-CASH-V2 | BICOM-REPORT-CASH | BICOM-REPORT-CASH | prod/bicom/consumers-NMjPni |
| bicom-contabilidad-general-balance-report | production.yml | development | LAMBDA | arm64 | bicom-contabilidad-general-balance | — | bicom-lambda-contabilidad-general-balance | — | — | — |
| bicom-contabilidad-rp-daily-book | production.yml | production | LAMBDA | arm64 | bicom-contabilidad-daily-book | — | bicom-lambda-contabilidad-daily-book | — | — | — |
| bicom-duemint-cron-lambda-go | production.yml | production | LAMBDA | arm64 | bicom-lambda-cron-duemint | — | bicom-lambda-lambda-cron-duemint | — | — | — |
| bicom-duemint-webhook-lambda | production.yml | production | LAMBDA | amd64 | bicom-lambda-webhook-duemint | — | bicom-lambda-lambda-webhook-duemint | — | — | — |
| bicom-lambda-rcof | production.yml | main | LAMBDA | amd64 | bicom-ms-dte-rcof | — | bicom-lambda-ms-dte-rcof | — | — | — |
| bicom-ms-stock-closing-lambda-cron | production.yml | main | LAMBDA | arm64 | bicom-stock-closing-lambda-cron | — | bicom-lambda-stock-closing-lambda-cron | — | — | — |
| bicom-rp-pos-authorizations | production.yml | master | LAMBDA | arm64 | bicom-rp-pos-authorizations | — | bicom-lambda-rp-pos-authorizations | — | — | — |

## Anomalías a verificar antes de copiar

- **`bicom-ms-customer-suppliers-ts`** construye `--platform linux/amd64` pero despliega al
  cluster `BICOM-CONSUMERS-ARM`. **`bicom-ms-subscription-cash`** ni siquiera pasa
  `--platform`, así que sale amd64 (la del runner), también contra `BICOM-CONSUMERS-ARM`.
  El nombre del cluster no impone la arquitectura — la fija el `runtimePlatform` de cada task
  definition — así que puede ser correcto. Confírmalo con
  `aws ecs describe-task-definition --task-definition <TD> --query 'taskDefinition.runtimePlatform'`
  antes de tomar cualquiera de los dos como plantilla.
- **`bicom-contabilidad-general-balance-report`** despliega a producción desde la rama
  `development`. Es lo que dice el workflow; si vas a tocar ese repo, pregunta antes de
  "arreglarlo".
- **`bicom-crt-stage3`** despliega recursos llamados `...STAGE2` (ECR `bicom-ms-certification-stage2`,
  servicio y TD stage2). El nombre del repo y el del recurso no coinciden: no lo deduzcas.
- **`bicom-api-dte`** tiene dos workflows sobre la misma rama `main` y el mismo servicio
  `BICOM-SERVICE-API-DTE` en clusters distintos (`BICOM-DTE` y `BICOM-DTE-API`), sin bloque
  `concurrency`: los dos corren en paralelo en cada push.
- **`bicom-report-cash`** usa el mismo nombre (`BICOM-REPORT-CASH`) para la task definition y
  para el contenedor. Es válido, pero cuidado al copiarlo a un repo nuevo.
