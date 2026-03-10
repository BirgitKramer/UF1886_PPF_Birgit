# BLOQUE 1.1 – Parámetros del sistema que influyen en el rendimiento
## 1. Objetivo
Identificar los principales parámetros técnicos que afectan al rendimiento del
entorno compuesto por:
- Odoo 18 en contenedor Docker
- PostgreSQL transaccional en contenedor Docker
- PostgreSQL para almacén de datos / staging
- Apache Hop ejecutándose en el host Windows
---
## 2. Análisis del sistema anfitrión (Windows)
### 2.1 CPU
**Comando utilizado:**
```powershell
Get-CimInstance Win32_Processor | Select-Object Name,NumberOfCores,NumberOfLogicalProcessors,MaxClockSpeed
```
Resultado resumido:

- Procesador: 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
- Núcleos: 4
- Procesadores lógicos: 8
- Velocidad máxima: 2419

Interpretación técnica:
- La CPU disponible influye en el rendimiento de Docker Desktop, Odoo,
PostgreSQL y Apache Hop. Una alta carga puede ralentizar la ejecución del ERP
y de los procesos ETL.

### 2.2 Memoria
**Comando utilizado:**
```powershell
Get-CimInstance Win32_OperatingSystem | Select-Object TotalVisibleMemorySize,FreePhysicalMemory
```

Resultado resumido:
TotalVisibleMemorySize FreePhysicalMemory
---------------------- ------------------
              16507816            6517980

- Memoria total: 16507816
- Memoria libre: 6517980

Interpretación técnica:
- La memoria disponible condiciona el comportamiento de Docker Desktop, los
contenedores y Apache Hop. Si la memoria libre es baja, puede producirse
degradación del rendimiento general.

### 2.3 Espacio en disco
**Comando utilizado:**
```powershell
Get-PSDrive -PSProvider FileSystem

```
Resultado resumido:

- Unidad principal: C:\  
- Espacio usado:  205,57 GB
- Espacio libre:  32,26 GB

Interpretación técnica:
- El espacio en disco afecta al almacenamiento de datos, logs, imágenes Docker,
volúmenes y ficheros temporales generados por PostgreSQL y Apache Hop.

### 2.4 Procesos activos
**Comandos utilizados:**
```powershell
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10
```
```
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    593      53   357604     321120     258,84  12048  23 Code
   1211      69   144064     162348     226,72  24468  23 com.docker.backend
    780     100   212796     143380     115,56   7652  23 Acrobat
    360      27    86124     134860     114,06  14436  23 Docker Desktop
   4695     130   247096     346004     105,98  14700  23 explorer
    779      29   163920     161484      95,16  20288  23 Code
   1268      75   130332     156704      92,31   7136  23 Code
    383      37   140856     154220      89,67  13368  23 Code
   1792      77    84932     181968      69,95  22304  23 chrome
    380      40   166508     218816      47,55  12092  23 chrome
```
```
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
      0       0     3712    1524616              3188   0 Memory Compression
      0       0  1814384     413860             24332   0 vmmemWSL
   4696     130   247120     345640     106,16  14700  23 explorer
    593      53   360972     321016     273,17  12048  23 Code
   1054     248   437204     227720              9008   0 MsMpEng
   1786      65    99008     198964       2,36  13344  23 msedge
   1790      77    84912     177904      70,86  22304  23 chrome
    779      29   164112     161304      99,14  20288  23 Code
   1213      69   141244     159440     232,17  24468  23 com.docker.backend
   1626     108    60852     158752       3,44  24572  23 PhoneExperienceHost
```

Resultado resumido:
- Procesos con mayor CPU:  258,84  Code
- Procesos con mayor uso de memoria: Memory Compression

Interpretación técnica:
- El análisis de procesos permite detectar si Docker Desktop, Java/Apache Hop u
otros procesos del sistema están consumiendo demasiados recursos.

## 3. Análisis de contenedores Docker
### 3.1 Contenedores activos
**Comando utilizado:**
```powershell
docker ps
```
Resultado resumido:
- Contenedor Odoo: odoo.18
- Contenedor PostgreSQL: postgres.db
- Estado: Up

CONTAINER ID   IMAGE                COMMAND                  CREATED       STATUS       PORTS                                                                                      
NAMES
8498e0473cbd   odoo:18.0            "/usr/bin/odoo -c /e…"   2 hours ago   Up 2 hours   0.0.0.0:8001->8069/tcp, [::]:8001->8069/tcp, 0.0.0.0:8002->8072/tcp, [::]:8002->8072/tcp   odoo.18
42f22f38e58d   postgres:16-alpine   "docker-entrypoint.s…"   2 hours ago   Up 2 hours   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp                                                
postgres.db

### 3.2 Consumo de recursos de contenedores
**Comando utilizado:**
```powershell
docker stats --no-stream
```

Resultado resumido:
- CPU usada por Odoo: 360.3MiB
- RAM usada por Odoo: 4.61%
- CPU usada por PostgreSQL:120,4MiB
- RAM usada por PostgreSQL:1,54 %

CONTAINER ID   IMAGE                COMMAND                  CREATED       STATUS       PORTS                                                                                      
NAMES
8498e0473cbd   odoo:18.0            "/usr/bin/odoo -c /e…"   2 hours ago   Up 2 hours   0.0.0.0:8001->8069/tcp, [::]:8001->8069/tcp, 0.0.0.0:8002->8072/tcp, [::]:8002->8072/tcp   odoo.18
42f22f38e58d   postgres:16-alpine   "docker-entrypoint.s…"   2 hours ago   Up 2 hours   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp                                                
postgres.db
PS C:\Users\Alumno\Documents\UF1886_PPF_Birgit> docker stats --no-stream
CONTAINER ID   NAME          CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
8498e0473cbd   odoo.18       0.01%     360.3MiB / 7.629GiB   4.61%     83.7MB / 87.1MB   80.8MB / 22.5MB   4
42f22f38e58d   postgres.db   0.01%     120.4MiB / 7.629GiB   1.54%     70.9MB / 83.4MB   22.1MB / 446MB    8


Interpretación técnica:
- El consumo de recursos de los contenedores permite valorar si existe
saturación en el ERP o en la base de datos.