
# BLOQUE 2.1 – Diseño y creación del flujo ETL
## 1. Objetivo
Diseñar un pipeline en Apache Hop con el nombre `P02_transport_data.hpl` para
realizar el transporte de datos desde el ERP Odoo 18 hacia el esquema
`staging` del almacén de datos.
El pipeline debe:
1. Leer datos desde una tabla del ERP
   - La tabla res_partner pertenece a la base de datos de Odoo y almacena
información relacionada con contactos, clientes y empresas.
Se ha seleccionado esta tabla porque permite trabajar con:
     - campos de texto
     - posibles valores nulos
     - limpieza de espacios
     - conversión de tipos de datos
1. Aplicar transformaciones básicas
2. Insertar los datos procesados en una tabla del esquema `staging`
En este caso se ha utilizado como tabla de origen:
```sql
res_partner
```
### Modificar tabla staging.stg_res_partner
```sql
ALTER TABLE staging.stg_res_partner 
ADD COLUMN customer_rank INTEGER;

ALTER TABLE staging.stg_res_partner 
ADD COLUMN supplier_rank INTEGER;

ALTER TABLE staging.stg_res_partner 
ADD COLUMN name_clean TEXT,
ADD COLUMN email_clean TEXT,
ADD COLUMN phone_clean TEXT;
```