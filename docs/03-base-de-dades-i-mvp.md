# 03. Desplegament de Base de Dades i MVP Web (Sprint 2)

## 1. Topologia de Xarxa i Infraestructura Cloud
Dins de la VPC `P0.0-ASIXc2gC-Gnn-VPC`, s'ha configurat una arquitectura en dues capes per tal de garantir el seccionament de la xarxa:

- **Instància `R-NCC` (DMZ):**
  - IP Privada: `10.0.1.80` (Subnet-DMZ)
  - IP Pública dinàmica assignada per AWS EC2
  - Serveis: Apache2, mòdul PHP 8.x, encaminament de xarxa i accés SSH
  - Ports oberts: `22` (SSH), `80` (HTTP)

- **Instància `B-NCC` (Intranet):**
  - IP Privada: `10.0.2.192` (Subnet-Intranet)
  - Sense IP Pública (aïllada completament d'Internet)
  - Serveis: MySQL Server 8.x
  - Ports oberts: `22` (només des de la subxarxa DMZ `10.0.1.0/24`), `3306` (MySQL des de `10.0.1.0/24`)

---

## 2. Configuració de MySQL i Ingesta del Dataset (B-NCC)

### 2.1. Instal·lació Offline i Binding de Xarxa
A causa de l'aïllament de la subxarxa Intranet, els paquets `.deb` es van descarregar des de `R-NCC` i es van transferir mitjançant SCP per ser instal·lats amb `dpkg`:

```bash
sudo dpkg -i /home/bchecker/mysql_pkgs/*.deb
```

Es va configurar el fitxer `/etc/mysql/mysql.conf.d/mysqld.cnf` per permetre la connexió remota des de la DMZ i l'ús de fitxers locals:

```ini
bind-address = 0.0.0.0
local_infile = 1
```

### 2.2. Processament del Dataset (OpenData Barcelona)
El fitxer `equipaments_educacio.csv` (2.709 registres) presentava una codificació original UTF-16LE amb caràcters especials. Es va processar i netejar a UTF-8 pur mitjançant `iconv` i `sed`:

```bash
iconv -f UTF-16LE -t UTF-8 /home/bchecker/equipaments_educacio.csv | tr -d '\r' | sed '1s/^\xEF\xBB\xBF//' > /home/bchecker/equipaments_utf8.csv
```

### 2.3. Estructura de la Base de Dades i Importació SQL
S'ha creat la base de dades `barcelona_edu` i la taula `equipaments` amb el següent script SQL:

```sql
CREATE DATABASE IF NOT EXISTS barcelona_edu;
CREATE USER IF NOT EXISTS 'bchecker'@'%' IDENTIFIED BY 'bchecker121';
GRANT ALL PRIVILEGES ON barcelona_edu.* TO 'bchecker'@'%';
FLUSH PRIVILEGES;

USE barcelona_edu;
SET GLOBAL local_infile = 1;

CREATE TABLE IF NOT EXISTS equipaments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    codi_equipament VARCHAR(50),
    nom VARCHAR(255),
    seccio VARCHAR(100),
    nom_districte VARCHAR(100),
    nom_barri VARCHAR(100),
    adreca VARCHAR(255),
    telefon VARCHAR(50)
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

LOAD DATA LOCAL INFILE '/home/bchecker/equipaments_utf8.csv'
INTO TABLE equipaments
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n'
IGNORE 1 LINES
(
  @col1, @name, @col3, @col4, @col5, @col6, @col7, @col8, @col9,
  @street, @col11, @col12, @col13, @neigh, @col15, @district,
  @col17, @col18, @col19, @col20, @col21, @col22, @col23, @col24,
  @phone, @col26, @col27, @col28, @sec, @col30, @col31, @col32,
  @col33, @col34, @col35, @col36, @col37, @col38, @col39, @col40
)
SET
  codi_equipament = @col1,
  nom = NULLIF(@name, ''),
  seccio = NULLIF(@sec, ''),
  nom_districte = NULLIF(@district, ''),
  nom_barri = NULLIF(@neigh, ''),
  adreca = NULLIF(@street, ''),
  telefon = NULLIF(@phone, '');
```

---

## 3. Desplegament de l'Aplicació Web MVP (R-NCC)
En la instància `R-NCC` s'ha instal·lat el servidor web Apache2 amb el mòdul de PHP:

```bash
sudo apt update && sudo apt install -y apache2 php libapache2-mod-php php-mysql
```

Es va crear l'aplicació PHP `/var/www/html/index.php` que consulta directament la base de dades remota a `10.0.2.192`:

```php
<?php
$host = '10.0.2.192';
$user = 'bchecker';
$pass = 'bchecker121';
$db   = 'barcelona_edu';

$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die("<h1>Error de connexió:</h1> " . $conn->connect_error);
}

$sql = "SELECT nom, nom_districte, adreca FROM equipaments WHERE nom IS NOT NULL LIMIT 20";
$result = $conn->query($sql);
?>

<!DOCTYPE html>
<html lang="ca">
<head>
    <meta charset="UTF-8">
    <title>MVP Equipaments Educació BCN</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 30px; background-color: #f4f4f9; }
        h1 { color: #333; }
        table { width: 100%; border-collapse: collapse; background: #fff; }
        th, td { padding: 12px; border: 1px solid #ccc; text-align: left; }
        th { background-color: #0056b3; color: white; }
        tr:nth-child(even) { background-color: #f9f9f9; }
    </style>
</head>
<body>
    <h1>Equipaments d'Educació de Barcelona (Sprint 2 ASIX)</h1>
    <p><strong>Estat connexió BBDD:</strong> OK (Connectat a B-NCC 10.0.2.192)</p>
    <table>
        <tr>
            <th>Nom Escola</th>
            <th>Districte</th>
            <th>Adreça</th>
        </tr>
        <?php while($row = $result->fetch_assoc()): ?>
        <tr>
            <td><?php echo htmlspecialchars($row['nom']); ?></td>
            <td><?php echo htmlspecialchars($row['nom_districte']); ?></td>
            <td><?php echo htmlspecialchars($row['adreca']); ?></td>
        </tr>
        <?php endwhile; ?>
    </table>
</body>
</html>
```

---

## 4. Matriu de Proves i Verificació

| Prova | Comanda / Endpoint | Resultat Esperat | Estat |
|---|---|---|---|
| Accés SSH Intranet | `ssh bchecker@10.0.2.192` des de R-NCC | Accés correcte via contrasenya | PASS |
| Connexió MySQL | `mysql -h 10.0.2.192 -u bchecker -p` | Connexió establerta correctament | PASS |
| Integritat de Dades | `SELECT COUNT(*) FROM equipaments;` | 2.709 registres importats | PASS |
| Renderitzat Web | `http://<IP_PUBLICA_R_NCC>` | Visualització de la taula HTML amb dades reals | PASS |
