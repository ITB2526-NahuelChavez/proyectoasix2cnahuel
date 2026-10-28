# 4. Pla de Prevenció de Riscos i Contingència (RA3)

## 4.1 Prevenció de Riscos Laborals (PRL)
1. **Ergonomia digital:** Ajust de la il·luminació i posició de la pantalla durant les sessions de configuració cloud.
2. **Pauses mecano-estàtiques:** Pauses de 5 minuts per cada hora de treball d'administració de sistemes.

## 4.2 Riscos Tècnics i Mitigació
- **Agotament del saldo de $50 d'AWS:** Ús d'instàncies de baix consum (`t2.micro`) i apagat d'equips en períodes d'inactivitat.
- **Pèrdua d'accés per claus SSH:** Configuració de l'usuari `bchecker` amb autenticació per contrasenya i clave SSH compartida com a fallback.
