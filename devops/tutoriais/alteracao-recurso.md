## Introdução

Neste Guia será descrito o passo a passo de como realizar a alteração de recursos nas aplicações do Openshift OCP.

## Pré-requisitos 

Para implementação das mudanças, deve ser realizado um novo deploy nos ambientes.

### Ambiente de desenvolvimento

- Para o ambiente de desenvolvimento, as alterações devem ser feitas diretamente na branch `develop`; ou
- Deve ser criado uma branch chamada `recurso`, realizar as alterações nessa branch e realizar o Merge Request para a branch `develop`.

### Ambiente de teste

- Caso, tenha ambiente de teste, deve ser realizado o deploy manual para este ambiente, após o commit na branch `develop`.

### Ambiente de homologação

- No ambiente de homologação, deve ser aberto um MR da branch `develop` para a `master`, com as alterações de recursos;

### Ambiente de pré-prod

- Em pré-prod, caso tenha esse ambiente, deve ser aprovado o MR para a branch `master`.


## Passo a passo

### Alteração dos arquivos DHT

Estando com os arquivos do seu projeto git da aplicação `app-xpto`:

As alterações devem ser feitas tanto nas camadas de `backend` e `frontend`.

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
          memory: "XXXMi" // <<< Alterar para o valor da coluna "MEM Recomendado"
          cpu: "XXXm"     // <<< Alterar para o valor da coluna "CPU Recomendado"
      ....
```


As alterações devem ser feitas somente no item `resource.requests`.

**O `resource.limits` não deve ser alterado.**


