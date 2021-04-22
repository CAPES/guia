[[_TOC_]]

## Introdução

O gerenciamento de armazenamento é um problema distinto do gerenciamento de sistemas de computação. O subsistema PersistentVolume fornece uma API para usuários e administradores que abstrai detalhes de como o armazenamento é fornecido e como é consumido. Para fazer isso, apresentamos dois novos recursos de API: PersistentVolume e PersistentVolumeClaim.

Um PersistentVolume (PV) é uma parte de armazenamento no cluster que foi provisionada por um administrador ou provisionada dinamicamente usando Classes de Armazenamento . É um recurso no cluster, assim como um nó é um recurso de cluster. PVs são plug-ins de volume como Volumes, mas têm um ciclo de vida independente de qualquer Pod individual que usa o PV. 

Um PersistentVolumeClaim (PVC) é uma solicitação de armazenamento por um usuário. É semelhante a um Pod. Os pods consomem recursos de nó e os PVCs consomem recursos de PV. Os pods podem solicitar níveis específicos de recursos (CPU e memória). As declarações podem solicitar tamanhos específicos e modos de acesso (por exemplo, eles podem ser montados ReadWriteOnce, ReadOnlyMany ou ReadWriteMany.


## Exemplos

### Persistent Volume

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: app-backend-des-log-pv
spec:
  capacity:
    storage: 2Gi
  nfs:
    server: nas08.capes.gov.br
    path: /vol/VOL_APP_OPENSHIFT_LOGS_DHT/DES/BACKEND
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  mountOptions:
    - intr
    - hard
    - sync
    - tcp
    - local_lock=all
```


