# Estrutura de Pastas

Organização de diretórios e principais arquivos.

```mermaid
graph LR
    raiz(app)
    raiz --> devops(devops)
    raiz --> system(system)
    system --> tiersystem("backend | frontend")
    tiersystem --> codigo("código-fonte")
    devops --> gitlab(".gitlab-ci.yml")
    devops --> tier("backend | frontend")
    tier --> Dockerfile
    tier --> app(chart)
    app --> C[Chart.yaml]
    app --> E("values-<ambiente>.yaml")
```

Exemplo de estrutura de pastas de uma aplicação `xpto-api`.

```
└── xpto-api/
    └── devops/
        ├── .gitlab-ci.yml
        └── backend/
            ├── Dockerfile
            └── xpto-api/
                ├── Chart.yaml
                ├── values-des.yaml
                ├── values-prod.yaml
        └── frontend
            ├── Dockerfile
            └── xpto-api/
                ├── Chart.yaml
                ├── values.yam
                ├── values-prod.yaml
    └── system/
        └── backend/
            ├── composer.json
            ├── index.php
        └── frontend/
            ├── index.html
            └── css/
            └── js/
```



## Pasta `devops`

A pasta `devops` é responsável por contém as configurações de pipeline, a instrução de build da aplicação (Dockerfile) e todos os arquivos que contém as configurações de release e deploy.

