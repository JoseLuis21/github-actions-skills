---
name: github-actions-skills
description: Escribe, migra o revisa el workflow de deploy de un repo bicom en AWS (us-east-2, cuenta 701355173252) — build de imagen, push a ECR y despliegue a un servicio ECS, a una Lambda de contenedor o solo a ECR (imágenes base). Úsala cuando pidan crear el `.github/workflows/*.yml` de un microservicio/API/consumer/lambda de bicom, clonar el deploy de otro repo bicom, pasar un deploy a ARM64/Graviton, arreglar un deploy que quedó viejo (`::set-output`, sin cache, sin tag por SHA, doble `docker login`), añadir las notificaciones de Slack, o inyectar los secretos de Secrets Manager como `.env` en la imagen.
---

# Deploys de bicom en GitHub Actions

Los ~45 workflows de `bicom/github` son el mismo esqueleto repetido con tres finales
distintos. Esta skill fija ese esqueleto, dice qué cambia en cada variante y lista la
deuda concreta que arrastran los repos viejos.

Los dos workflows de referencia — los únicos con cache, tag por SHA y `provenance:false` —
son `bicom-ms-stock-closing` (ECS) y `bicom-ms-stock-closing-lambda-cron` (Lambda). Cuando
haya duda, gana lo que hacen ellos, no lo que hace la mayoría.

## Constantes de la organización

No las inventes ni las preguntes: son iguales en todos los repos.

| Qué               | Valor                                                         |
| ----------------- | ------------------------------------------------------------- |
| Región            | `us-east-2`                                                   |
| Cuenta AWS        | `701355173252`                                                |
| Credenciales      | `secrets.AWS_ACCESS_KEY`, `secrets.AWS_SECRET_ACCESS_KEY`     |
| Webhook Slack     | `secrets.SLACK_CHANNEL_BICOM` → env `SLACK_WEBHOOK_URL`       |
| Canal de inicio   | `#github-actions-bicom-oficial`                               |
| Canal de cierre   | `#workflows`                                                  |
| Repos Go privados | `secrets.GH_USERNAME`, `secrets.GH_ACCESS_TOKEN` (build-args) |
| Concurrency       | `group: ci-${{ github.ref }}`, `cancel-in-progress: true`     |
| Permisos          | `permissions: contents: read`                                 |
| Timeout           | 15 min (ECS/imagen), 25 min (Lambda o build con Go)           |

## Elegir plantilla

| Destino                                     | Plantilla                   | Señal en el repo                                    |
| ------------------------------------------- | --------------------------- | --------------------------------------------------- |
| Servicio ECS (API, front Next.js, consumer) | `templates/ecs-service.yml` | hay `ECS_CLUSTER` / task definition                 |
| Lambda como imagen de contenedor            | `templates/lambda.yml`      | `Dockerfile` con base `public.ecr.aws/lambda/...`   |
| Imagen base (php-fpm, nginx, ecommerce)     | `templates/ecr-image.yml`   | nadie la despliega: otros repos la usan como `FROM` |

Sustituye todo lo que va entre `<...>`. Nada más.

## Datos que hay que averiguar antes de escribir

No adivines: cada uno tiene un comando que lo responde. Si no hay credenciales AWS a mano,
cópialos del workflow del repo hermano y **dilo explícitamente en el reporte**.

```bash
# Arquitectura real de la task definition (esto decide --platform, no el cluster)
aws ecs describe-task-definition --task-definition <TD> --region us-east-2 \
  --query 'taskDefinition.{arch:runtimePlatform.cpuArchitecture,containers:containerDefinitions[].name}'

# Nombre exacto del contenedor (CONTAINER_NAME) — un typo aquí hace un deploy verde que no cambia nada
aws ecs describe-services --cluster <CLUSTER> --services <SERVICE> --region us-east-2 \
  --query 'services[0].taskDefinition'

# ARN completo del secreto (lleva sufijo aleatorio: `-NMjPni`, no se puede escribir de memoria)
aws secretsmanager list-secrets --region us-east-2 --query 'SecretList[].[Name,ARN]' --output text

# Arquitectura actual de la Lambda
aws lambda get-function-configuration --function-name <fn> --region us-east-2 \
  --query '{arch:Architectures,entrypoint:ImageConfigResponse}'
```

`reference/inventario.md` tiene la tabla de los 45 workflows existentes (rama, ECR, cluster,
servicio, TD, contenedor, secreto, lambda). Úsala para copiar convenciones de nombres y para
saber qué repo es el hermano más parecido.

## Reglas que no se negocian

- **Etiqueta por SHA además de `latest`.** Casi todos los repos publican solo `:latest`: con eso
  un rollback obliga a reconstruir y a rezar por que el commit anterior compile igual. `:${{ github.sha }}`
  y desplegar por ese tag convierte el rollback en apuntar a la imagen anterior.
- **`--platform` tiene que coincidir con la task definition.** El cluster no lo decide:
  dentro de `BICOM-ECS-APIs` conviven servicios arm64 (api-go, ml-api, api-sii) y amd64
  (api-frontend, control-apibh). Una imagen amd64 en una TD arm64 arranca y muere en bucle
  con `exec format error`, y `wait-for-service-stability` lo transforma en un job colgado 10
  minutos antes de fallar.
- **QEMU solo si la etapa final ejecuta algo.** Con Go la etapa builder cross-compila
  (`--platform=$BUILDPLATFORM`) y la final solo copia → `docker/setup-qemu-action` sobra y
  suma minutos. Con Node/TS o PHP, la etapa target sí corre `npm ci` / `composer install`:
  ahí QEMU es obligatorio (`platforms: linux/arm64`).
- **`cache-from/to: type=gha, mode=max`.** Solo 4 de 45 workflows lo tienen. Es la diferencia
  entre un deploy de 2 minutos y uno de 8, y no cuesta nada.
- **`aws-actions/amazon-ecr-login@v2` ya deja hecho el `docker login`.** Repetir
  `aws ecr get-login-password | docker login` después es ruido que aparece en 8 repos.
- **La task definition se lee siempre de la familia, sin `:revision`.**
  `describe-task-definition --task-definition <FAMILIA>` devuelve la última ACTIVE, así el
  deploy respeta un cambio de CPU/memoria/rol hecho a mano en la consola en vez de pisarlo.
- **`wait-for-service-stability: true`.** Sin esto el job da verde cuando AWS _acepta_ el
  deploy, no cuando la tarea queda viva; un crash loop pasa desapercibido.
- **Consumers: `minimumHealthyPercent = 0` en el servicio ECS.** Es config del servicio, no
  del workflow, pero sin eso el redeploy solapa dos instancias procesando el mismo tenant.
- **Lambda: `--provenance=false --sbom=false`, `--architectures` y `wait function-updated`.**
  Lambda rechaza los manifests OCI con attestations que buildx adjunta por defecto
  (_"The image manifest, config or layer media type ... is not supported"_). Sin
  `--architectures` la función sigue en la arquitectura vieja y la imagen nueva no arranca.
  Sin el `wait`, cualquier `update-function-configuration` posterior falla con
  `ResourceConflictException`.
- **Una Lambda no lleva multi-arch.** Una sola arquitectura: `arm64` (Graviton, ~20% más
  barato) salvo que la función ya sea x86_64 y no se quiera migrar en el mismo PR.
- **`$GITHUB_OUTPUT`, nunca `::set-output`.** Está deprecado desde 2023 y ya avisa en los logs.

## Secretos → `.env` horneado en la imagen

Es el patrón de la casa y hay que mantenerlo por coherencia, pero conviene saber qué implica:

```yaml
- run:
    aws secretsmanager get-secret-value --secret-id "$SECRET_ARN" --region "$AWS_REGION" \
    --query 'SecretString' --output text > secrets.json
- run: jq -r 'to_entries|map("\(.key)=\(.value|tostring)")|.[]' secrets.json > "$ENV_FILE"
```

- El Dockerfile hace `COPY .env* /app/` **al final**, para que un cambio de secreto no
  invalide las capas caras de compilación. El glob evita que el build local reviente sin `.env`.
- Por eso las task definitions de bicom tienen `environment: []` y `secrets: []`: no hay que
  tocarlas al desplegar.
- **`.env` no puede estar en `.dockerignore`.** Si alguien lo agrega "por seguridad", el
  contenedor arranca sin configuración y el fallo aparece en runtime, no en el build.
- Nunca imprimas el `.env` en el log; como mucho `wc -l`.
- Trade-off real: el secreto queda horneado en cada imagen de ECR, así que rotarlo obliga a
  reconstruir y quien pueda hacer `docker pull` del repo ECR lo lee. La alternativa es
  `secrets:` en la task definition (valueFrom del ARN). No lo cambies por tu cuenta: es un
  cambio de plataforma para todos los servicios, propónlo aparte.
- Varios repos usan `--region $AWS_DEFAULT_REGION` en ese paso. Funciona porque
  `configure-aws-credentials` exporta esa variable, pero escribe `$AWS_REGION`, que sí está
  declarada en el `env:` del workflow.

## Slack

Dos pasos, siempre con `if: always()`: uno al principio con `status: starting` al canal
`#github-actions-bicom-oficial`, y uno al final con `status: ${{ job.status }}` y
`steps: ${{ toJson(steps) }}` a `#workflows`. El mensaje de inicio dice el servicio concreto
("Starting Deploy MS Stock Closing"), no "Starting Deploy MS Bicom" copiado del vecino —
con 45 repos notificando al mismo canal, un mensaje genérico no sirve para nada.

## Deuda conocida (para revisiones y migraciones)

Al tocar uno de estos repos, arregla de paso lo que aplique y dilo en el PR:

| Problema                                                                    | Repos                                                                                                                                                                                               |
| --------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `::set-output` deprecado                                                    | `bicom-api-dte` (x2), `bicom-ms-email-reader`, `bicom-test-aws-secrets`                                                                                                                             |
| `configure-aws-credentials@v1`                                              | `bicom-test-aws-secrets`                                                                                                                                                                            |
| Sin bloque `concurrency`                                                    | `bicom-api-dte` (x2), `bicom-ms-email-reader`, `bicom-test-aws-secrets`                                                                                                                             |
| `docker build` plano, sin buildx ni cache                                   | `bicom-crt-frontend`, `bicom-lambda-rcof`, `bicom-duemint-webhook-lambda`, `bicom-api-frontend`, `bicom-licitaciones-mp`                                                                            |
| `docker login` duplicado tras `amazon-ecr-login`                            | los 5 anteriores + `bicom-contabilidad-front` (x2), `bicom-contabilidad-rp-daily-book`, `bicom-rp-pos-authorizations`, `bicom-duemint-cron-lambda-go`, `bicom-contabilidad-general-balance-report`  |
| Solo tag `:latest` (sin rollback)                                           | todos menos `bicom-ms-stock-closing`, `bicom-ms-stock-closing-lambda-cron`, `bicom-crt-stage1`, `bicom-crt-stage3`, `bicom-api-dte`, `bicom-ms-email-reader`, `bicom-ecommerce-v2`                  |
| Sin `cache-from/to: type=gha`                                               | todos menos los 4 de referencia                                                                                                                                                                     |
| Lambda sin `--architectures` / `wait function-updated` / `provenance=false` | `bicom-lambda-rcof`, `bicom-duemint-webhook-lambda`, `bicom-duemint-cron-lambda-go`, `bicom-rp-pos-authorizations`, `bicom-contabilidad-rp-daily-book`, `bicom-contabilidad-general-balance-report` |
| QEMU innecesario en repos Go                                                | `bicom-api-go`, `bicom-api-bh-go`, `bicom-ms-logistic-pdf-go`, `bicom-api-sii`                                                                                                                      |

Migrar el tag a SHA cambia el contrato de rollback del equipo (ya no basta con "redeploy de
`latest`"): menciónalo al proponerlo, no lo cueles.

## Verificación antes de dar por hecho el deploy

```bash
# Sintaxis del workflow
actionlint .github/workflows/<archivo>.yml   # o: yq . .github/workflows/<archivo>.yml >/dev/null

# La imagen sale con la arquitectura correcta
docker buildx build --platform linux/arm64 --load -t check:local .
docker image inspect check:local --format '{{.Architecture}}'   # -> arm64

# Post-deploy ECS: la revisión nueva corre y no está reciclando tareas
aws ecs describe-services --cluster <CLUSTER> --services <SERVICE> --region us-east-2 \
  --query 'services[0].{td:taskDefinition,running:runningCount,desired:desiredCount,events:events[0:3].message}'

# Post-deploy Lambda: la imagen y la arquitectura quedaron como se esperaba
aws lambda get-function-configuration --function-name <fn> --region us-east-2 \
  --query '{arch:Architectures,state:State,update:LastUpdateStatus}'
```

Si no pudiste correr nada de esto (sin credenciales, sin daemon de Docker), dilo tal cual:
un `--platform` equivocado no lo detecta la revisión de YAML, lo detecta producción.
