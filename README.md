# bicom-github-actions

Skill de agente para escribir, migrar y revisar los workflows de deploy de los repos
`bicom/github` en AWS: build de imagen, push a ECR y despliegue a ECS, a una Lambda de
contenedor o solo a ECR.

Destilada de los 45 workflows existentes; los de referencia son `bicom-ms-stock-closing`
(ECS) y `bicom-ms-stock-closing-lambda-cron` (Lambda).

```
SKILL.md                     reglas, decisiones y deuda conocida
templates/ecs-service.yml    servicio ECS  (secretos -> .env, tag por SHA, cache gha)
templates/lambda.yml         Lambda de contenedor (arm64, provenance=false, wait)
templates/ecr-image.yml      imagen base: solo build + push a ECR
reference/inventario.md      los 45 deploys: rama, ECR, cluster, servicio, TD, secreto
```

## Instalar en un repo

Igual que `docker-golang-skills`: se copia en `.agents/skills/` (y se referencia en
`skills-lock.json`), o se apunta el agente a este repo.

```bash
git clone <este-repo> .agents/skills/bicom-github-actions
```

## Complementa a

`JoseLuis21/docker-golang-skills` — esa skill arma el `Dockerfile` (cache de BuildKit,
cross-compilación, Lambda Insights); esta arma el workflow que lo construye y lo despliega.
