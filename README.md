# aws-deploy-actions

Skill de agente para escribir, migrar y revisar workflows de GitHub Actions que construyen
una imagen Docker, la suben a ECR y la despliegan en AWS: servicio ECS, Lambda de
contenedor, o solo publicación de la imagen.

```
SKILL.md                     reglas, decisiones y errores que solo aparecen en producción
templates/ecs-service.yml    servicio ECS (secretos -> .env, tag por SHA, cache gha)
templates/lambda.yml         Lambda de contenedor (arm64, provenance=false, wait)
templates/ecr-image.yml      imagen base: solo build + push a ECR
reference/auditoria.md       comandos para auditar workflows existentes
```

Las plantillas usan `<placeholders>` para todo lo que cambia por organización (región,
cuenta, nombres de secrets y canales, convención de nombres de recursos). La skill explica
de dónde sacar cada valor antes de escribir nada.

## Instalar en un repo

```bash
git clone <este-repo> .agents/skills/aws-deploy-actions
```

## Complementa a

`JoseLuis21/docker-golang-skills` — esa skill arma el `Dockerfile` (cache de BuildKit,
cross-compilación, Lambda Insights); esta arma el workflow que lo construye y lo despliega.
