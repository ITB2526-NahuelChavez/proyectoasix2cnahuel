# 1. Estudi Previ de Tecnologies i Anàlisi de Mercat (RA1)

## 1.1 Introducció
L'objectiu d'aquest projecte és dissenyar i desplegar una infraestructura multicapa segura sobre AWS Cloud per a l'assignatura d'ASIX.

## 1.2 Comparativa de Tecnologies de Virtualització
| Criteri | On-Premises (VirtualBox) | AWS Cloud (VPC + EC2) |
| :--- | :--- | :--- |
| **Cost inicial** | Alt (requereix maquinari local de gran capacitat) | Zero (pagament per ús / AWS Academy) |
| **Escalabilitat** | Limitada per la RAM i CPU físiques | Quasi il·limitada instantàniament |
| **Aïllament de Xarxa** | Complex de simular amb interfícies virtuals | Nativa mitjançant VPC, Subnets i Security Groups |
| **Disponibilitat** | Depèn de l'host local | Alta disponibilitat multizona |

## 1.3 Justificació de la Solució Seleccionada
S'ha seleccionat **AWS Cloud** utilitzant **instàncies EC2 Ubuntu 24.04 LTS**. Aquesta opció permet simular la topologia física d'una xarxa d'empresa real (DMZ + Intranet + NAT) garantint la seguretat per disseny mitjançant Grups de Seguretat i Taules d'Enrutament separat.
