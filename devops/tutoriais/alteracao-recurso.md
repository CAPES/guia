## Introdução

Neste Guia será descrito o passo a passo de como realizar a alteração de recursos nas aplicações do Openshift OCP.

## Pré-requisitos

### Verificar a planilha de recomendações

### Alteração na branch develop

- As alterações devem ser feitas diretamente na branch `develop`; ou
- Deve ser criado uma branch chamada `recurso`, realizar as alterações nessa branch e realizar o Merge Request para a branch `develop`.


## Passo a passo

### Alteração dos arquivos DHT

Estando com os arquivos do seu projeto git da aplicação <app-xpto>:

Na pasta `/devops/<backend/frontend>/<app-xpto>`, alterar em todos os arquivos `values` dos ambientes de DHT. (Não alterar o arquivo `values-prod.yaml`)
```
values-des.yaml
values-teste.yaml
values-hom.yaml
values-preprod.yaml
values-review.yaml
```

Alterar os seguintes recursos nos arquivos:

```
app:
    ....
    containers:
      ....
      resources:
        requests:
          memory: "XXXMi" // <<< Alterar para o valor da coluna "CPU Recomendado"
          cpu: "XXXm"     // <<< Alterar para o valor da coluna "MEM Recomendado"
      ....
```





