# 2. Arquitectura de Xarxa i Infraestructura (RA2)

## 2.1 Disseny del xarxament (VPC)
- **CIDR Principal VPC:** `10.0.0.0/16`
- **Subxarxa DMZ (Pública):** `10.0.1.0/24` (Conté el Router R-NCC i Servidor Web W-NCC)
- **Subxarxa Intranet (Privada):** `10.0.2.0/24` (Conté BBDD B-NCC, serveis interns i clients)

## 2.2 Taula de Matriu d'Equips i IPs
| Hostname | Rol / Servei | Subxarxa | IP Privada | IP Pública |
| :--- | :--- | :--- | :--- | :--- |
| **R-NCC** | Router / Gateway / SSH | DMZ | `10.0.1.254` | Sí (Elastic/Dynamic) |
| **W-NCC** | Web Server / Demo MVP | DMZ | `10.0.1.10` | Opcional |
| **B-NCC** | BBDD MySQL | Intranet | `10.0.2.10` | No |
| **F-NCC** | Servidor FTP | DMZ | `10.0.1.20` | No |

## 2.3 Seguretat i Usuaris
Tots els equips disposen de l'usuari d'administració estàndard del projecte:
- **Usuari:** `bchecker`
- **Password:** `bchecker121`
