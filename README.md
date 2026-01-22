# 🚀 1839 - Administración de MySQL: Seguridad y Optimización - Parte 2

<div align="center">

**Curso Avanzado de Administración de Bases de Datos MySQL - Continuación**

[![MySQL Expert](https://img.shields.io/badge/MySQL-Administración_Avanzada-4479A1?style=for-the-badge&logo=mysql&logoColor=white)](https://www.mysql.com/)
[![Alura Latam](https://img.shields.io/badge/Plataforma-Alura_Latam-00C86F?style=for-the-badge)](https://www.aluracursos.com/)
[![High Availability](https://img.shields.io/badge/Alta-Disponibilidad-CC2927?style=for-the-badge)](https://www.w3schools.com/sql/)
[![Licencia](https://img.shields.io/badge/Licencia-MIT-yellow?style=for-the-badge)](LICENSE)

[🔄 Replicación](#-replicación-avanzada) • 
[📈 Performance](#️-tuning-avanzado-de-rendimiento) • 
[🔍 Troubleshooting](#-diagnóstico-y-solución-de-problemas) • 
[⚙️ Automatización](#️-automatización-y-orquestación) • 
[👨‍💻 Autor](#-autor)

</div>

---

## 🎯 Continuación del Curso de Administración MySQL

Este repositorio contiene los **comandos avanzados, configuraciones y estrategias** de la segunda parte del curso **"Administración de MySQL: Seguridad y optimización de la base de datos"** de Alura Latam. Esta continuación profundiza en temas críticos para administradores de bases de datos en entornos empresariales y de alta demanda.

### 🎓 Lo que Aprenderás en la Parte 2
- **Replicación avanzada** y configuración multi-maestro
- **Tuning experto** de rendimiento para cargas masivas
- **Diagnóstico profundo** de problemas complejos
- **Automatización** de tareas DBA críticas
- **Estrategias de escalabilidad** horizontal y vertical
- **Monitoreo proactivo** con alertas inteligentes
- **Planificación de capacidad** y forecasting

### 📊 Nivel de Complejidad
| Tema | Parte 1 | Parte 2 |
|------|---------|---------|
| **Replicación** | Básica (master-slave) | Avanzada (multi-master, GTID) |
| **Optimización** | Ajustes de memoria | Tuning de consultas complejas |
| **Seguridad** | Usuarios y permisos | Encriptación, auditing |
| **Backup** | Estrategias básicas | Point-in-time recovery avanzado |
| **Monitoreo** | Métricas básicas | Alertas predictivas |

---

## 🔄 Replicación Avanzada

### **1. Configuración Multi-Master con GTID**
```sql
-- Configurar GTID (Global Transaction ID) en todos los nodos
-- En my.cnf de cada servidor:
[mysqld]
server-id = [ID_UNICO]
gtid_mode = ON
enforce_gtid_consistency = ON
log-bin = mysql-bin
log-slave-updates = ON
binlog-format = ROW

-- Configurar replicación circular multi-master
-- En servidor 1 (ID=1):
CHANGE MASTER TO
MASTER_HOST = 'servidor2',
MASTER_USER = 'replicator',
MASTER_PASSWORD = 'SecurePass123!',
MASTER_AUTO_POSITION = 1;

-- En servidor 2 (ID=2):
CHANGE MASTER TO
MASTER_HOST = 'servidor1',
MASTER_USER = 'replicator',
MASTER_PASSWORD = 'SecurePass123!',
MASTER_AUTO_POSITION = 1;

-- Iniciar replicación en ambos
START SLAVE;

-- Verificar estado con GTID
SHOW SLAVE STATUS\G
SELECT * FROM mysql.gtid_executed;
```

### **2. Replicación Semi-Sincrónica**
```sql
-- Instalar plugin semi-sync
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

-- Configurar maestro
SET GLOBAL rpl_semi_sync_master_enabled = 1;
SET GLOBAL rpl_semi_sync_master_timeout = 1000;  -- 1 segundo timeout
SET GLOBAL rpl_semi_sync_master_wait_for_slave_count = 1;

-- Configurar esclavo
SET GLOBAL rpl_semi_sync_slave_enabled = 1;

-- Verificar estado
SHOW STATUS LIKE 'Rpl_semi_sync%';
```

### **3. Filtrado Avanzado de Replicación**
```sql
-- Replicar solo bases de datos específicas
CHANGE REPLICATION FILTER 
REPLICATE_DO_DB = (app_production, reporting_db);

-- Excluir tablas específicas
CHANGE REPLICATION FILTER 
REPLICATE_IGNORE_TABLE = (app_production.logs, app_production.temp_data);

-- Replicar solo ciertas operaciones
CHANGE REPLICATION FILTER 
REPLICATE_DO_TABLE = (app_production.users),
REPLICATE_WILD_DO_TABLE = ('app_production.orders_%');

-- Configuración en my.cnf para filtros permanentes
[mysqld]
replicate-do-db = app_production
replicate-ignore-db = mysql
replicate-wild-ignore-table = app_production.audit_%
```

### **4. Replicación con ProxySQL**
```sql
-- Configurar ProxySQL para balanceo de lectura
-- En ProxySQL admin interface:
INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES
(10, 'master1', 3306),
(20, 'slave1', 3306),
(20, 'slave2', 3306),
(20, 'slave3', 3306);

-- Configurar reglas de routing
INSERT INTO mysql_query_rules (rule_id, active, match_digest, destination_hostgroup, apply)
VALUES
(1, 1, '^SELECT.*FOR UPDATE', 10, 1),  -- SELECTs con locking al master
(2, 1, '^SELECT', 20, 1),              -- SELECTs normales a slaves
(3, 1, '.*', 10, 1);                   -- Todo lo demás al master

-- Configurar monitoreo
UPDATE mysql_servers SET max_replication_lag = 10;
LOAD MYSQL SERVERS TO RUNTIME;
SAVE MYSQL SERVERS TO DISK;
```

---

## 📈 Tuning Avanzado de Rendimiento

### **1. Optimización de Consultas a Nivel de Sistema**
```sql
-- Configuración avanzada de InnoDB para SSD
SET GLOBAL innodb_io_capacity = 20000;
SET GLOBAL innodb_io_capacity_max = 40000;
SET GLOBAL innodb_flush_neighbors = 0;  -- Desactivar para SSD
SET GLOBAL innodb_flush_method = O_DIRECT_NO_FSYNC;
SET GLOBAL innodb_buffer_pool_chunk_size = 134217728;  -- 128MB
SET GLOBAL innodb_buffer_pool_size = 8589934592;       -- 8GB

-- Ajustar para carga de trabajo específica
-- OLTP (alta concurrencia):
SET GLOBAL innodb_thread_concurrency = 0;
SET GLOBAL innodb_read_io_threads = 8;
SET GLOBAL innodb_write_io_threads = 8;

-- Data Warehouse (consultas pesadas):
SET GLOBAL innodb_buffer_pool_instances = 16;
SET GLOBAL innodb_old_blocks_time = 1000;
SET GLOBAL innodb_adaptive_hash_index = OFF;
```

### **2. Particionamiento Avanzado**
```sql
-- Particionamiento compuesto (RANGE + LIST)
CREATE TABLE ventas_detalladas (
    id BIGINT NOT NULL AUTO_INCREMENT,
    fecha DATE NOT NULL,
    region VARCHAR(50) NOT NULL,
    producto_id INT NOT NULL,
    cantidad INT NOT NULL,
    monto DECIMAL(12,2) NOT NULL,
    PRIMARY KEY (id, fecha, region)
)
PARTITION BY RANGE (YEAR(fecha))
SUBPARTITION BY LIST (REGION IN ('Norte', 'Sur', 'Este', 'Oeste')) 
SUBPARTITIONS 4 (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- Particionamiento por hash para distribución uniforme
CREATE TABLE sesiones_usuarios (
    session_id VARCHAR(128) NOT NULL,
    user_id INT NOT NULL,
    data TEXT,
    created_at TIMESTAMP,
    expires_at TIMESTAMP,
    PRIMARY KEY (session_id, user_id)
)
PARTITION BY HASH(user_id)
PARTITIONS 16;

-- Mantenimiento automático de particiones
DELIMITER $$
CREATE EVENT rotate_partitions
ON SCHEDULE EVERY 1 MONTH
DO
BEGIN
    -- Eliminar particiones antiguas
    SET @old_partition = DATE_FORMAT(DATE_SUB(NOW(), INTERVAL 13 MONTH), 'p%Y%m');
    SET @sql = CONCAT('ALTER TABLE ventas DROP PARTITION ', @old_partition);
    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
    
    -- Agregar nueva partición
    SET @new_partition = DATE_FORMAT(DATE_ADD(NOW(), INTERVAL 1 MONTH), 'p%Y%m');
    SET @sql = CONCAT('ALTER TABLE ventas ADD PARTITION (PARTITION ', @new_partition, 
                     ' VALUES LESS THAN (TO_DAYS("', 
                     DATE_FORMAT(DATE_ADD(NOW(), INTERVAL 2 MONTH), '%Y-%m-01'), '")))');
    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END $$
DELIMITER ;
```

### **3. Índices Avanzados y Estadísticas**
```sql
-- Crear índices funcionales (MySQL 8.0+)
CREATE INDEX idx_email_domain 
ON usuarios ((SUBSTRING_INDEX(email, '@', -1)));

CREATE INDEX idx_name_lower 
ON clientes ((LOWER(nombre_completo)));

-- Índices espaciales para datos geográficos
CREATE TABLE ubicaciones (
    id INT PRIMARY KEY,
    nombre VARCHAR(100),
    ubicacion POINT NOT NULL,
    SPATIAL INDEX(ubicacion)
);

-- Actualizar estadísticas de manera diferencial
ANALYZE TABLE ventas UPDATE HISTOGRAM ON fecha, total 
WITH 100 BUCKETS;

ANALYZE TABLE productos UPDATE HISTOGRAM ON precio 
WITH 50 BUCKETS;

-- Ver histogramas creados
SELECT * FROM information_schema.COLUMN_STATISTICS
WHERE TABLE_NAME = 'ventas';

-- Forzar uso de índice específico
SELECT /*+ INDEX(ventas idx_fecha_cliente) */ *
FROM ventas FORCE INDEX (idx_fecha_cliente)
WHERE fecha BETWEEN '2024-01-01' AND '2024-01-31';
```

### **4. Optimización de JOINs y Subconsultas**
```sql
-- Transformar subconsultas correlacionadas a JOINs
-- ANTES (ineficiente):
EXPLAIN
SELECT c.nombre, 
       (SELECT SUM(total) 
        FROM ventas v 
        WHERE v.cliente_id = c.id 
        AND v.fecha >= '2024-01-01') as total_2024
FROM clientes c;

-- DESPUÉS (optimizado):
EXPLAIN
SELECT c.nombre, COALESCE(SUM(v.total), 0) as total_2024
FROM clientes c
LEFT JOIN ventas v ON c.id = v.cliente_id 
                  AND v.fecha >= '2024-01-01'
GROUP BY c.id, c.nombre;

-- Usar derived condition pushdown (MySQL 8.0.22+)
SELECT /*+ SET_VAR(optimizer_switch = 'derived_condition_pushdown=on') */ *
FROM (
    SELECT cliente_id, SUM(total) as total_compras
    FROM ventas 
    GROUP BY cliente_id
) AS ventas_agrupadas
WHERE total_compras > 1000;

-- Optimizar JOINs con batch key access
SET GLOBAL optimizer_switch = 'mrr=on,mrr_cost_based=off,batched_key_access=on';
```

---

## 🔍 Diagnóstico y Solución de Problemas

### **1. Análisis de Deadlocks Avanzado**
```sql
-- Configurar logging detallado de deadlocks
SET GLOBAL innodb_print_all_deadlocks = ON;
SET GLOBAL innodb_status_output = ON;
SET GLOBAL innodb_status_output_locks = ON;

-- Consultar información de deadlocks recientes
SELECT * FROM information_schema.INNODB_TRX\G
SELECT * FROM information_schema.INNODB_LOCKS\G
SELECT * FROM information_schema.INNODB_LOCK_WAITS\G

-- Script para detectar y resolver deadlocks comunes
DELIMITER $$
CREATE PROCEDURE sp_deadlock_detector()
BEGIN
    DECLARE deadlock_count INT DEFAULT 0;
    
    -- Contar deadlocks en última hora
    SELECT VARIABLE_VALUE INTO deadlock_count
    FROM information_schema.GLOBAL_STATUS
    WHERE VARIABLE_NAME = 'Innodb_deadlocks';
    
    IF deadlock_count > 5 THEN
        -- Capturar información detallada
        CREATE TEMPORARY TABLE IF NOT EXISTS deadlock_info
        ENGINE=InnoDB
        SELECT NOW() as timestamp, *
        FROM information_schema.INNODB_TRX;
        
        -- Enviar alerta
        CALL sp_send_alert('deadlock_detected', 
            CONCAT('Se detectaron ', deadlock_count, ' deadlocks en la última hora'));
        
        -- Sugerir acciones correctivas
        INSERT INTO sistema.recomendaciones (tipo, descripcion, prioridad)
        VALUES ('DEADLOCK', 
                'Considerar revisar orden de acceso a tablas en transacciones concurrentes',
                'ALTA');
    END IF;
END $$
DELIMITER ;
```

### **2. Análisis de Rendimiento con sys Schema**
```sql
-- Consultas más costosas con contexto completo
SELECT * FROM sys.statement_analysis 
ORDER BY avg_latency DESC 
LIMIT 10;

-- Análisis de índice (qué índices faltan)
SELECT * FROM sys.schema_unused_indexes;

-- Análisis de rendimiento de usuarios
SELECT * FROM sys.user_summary 
ORDER BY total_latency DESC;

-- Identificar queries problemáticas
SELECT query, 
       db,
       exec_count,
       total_latency,
       avg_latency,
       rows_sent,
       rows_examined,
       rows_sent / rows_examined AS efficiency
FROM sys.statements_with_runtimes_in_95th_percentile
ORDER BY total_latency DESC;

-- Análisis de bloqueos en tiempo real
SELECT * FROM sys.innodb_lock_waits;

-- Verificar configuración no óptima
SELECT * FROM sys.schema_redundant_indexes;
SELECT * FROM sys.statements_with_full_table_scans;
```

### **3. Diagnóstico de Problemas de Memoria**
```sql
-- Análisis completo de uso de memoria
SELECT * FROM sys.memory_global_total;

-- Memoria por componente
SELECT * FROM sys.memory_global_by_current_bytes 
LIMIT 15;

-- Histórico de uso de buffer pool
SELECT 
    NOW() - INTERVAL (variable_value * 5) MINUTE as time_marker,
    variable_value * 5 as minutes_ago,
    (SELECT variable_value 
     FROM information_schema.GLOBAL_STATUS 
     WHERE variable_name = 'Innodb_buffer_pool_pages_total') as total_pages,
    (SELECT variable_value 
     FROM information_schema.GLOBAL_STATUS 
     WHERE variable_name = 'Innodb_buffer_pool_pages_free') as free_pages
FROM information_schema.GLOBAL_STATUS
WHERE variable_name = 'Uptime'
INTO OUTFILE '/tmp/buffer_pool_history.csv'
FIELDS TERMINATED BY ',' 
LINES TERMINATED BY '\n';

-- Detectar memory leaks
SELECT 
    event_name,
    current_alloc,
    high_alloc
FROM sys.memory_by_thread_by_current_bytes
WHERE current_alloc > 10485760  -- > 10MB
ORDER BY current_alloc DESC;
```

### **4. Troubleshooting de Replicación**
```sql
-- Diagnóstico completo de replicación
SELECT * FROM sys.replication_applier_status;
SELECT * FROM sys.replication_connection_status;

-- Detectar y corregir errores de replicación automáticamente
DELIMITER $$
CREATE PROCEDURE sp_replication_auto_heal()
BEGIN
    DECLARE repl_error INT DEFAULT 0;
    DECLARE last_error TEXT;
    
    -- Verificar si hay error
    SELECT LAST_ERROR_NUMBER, LAST_ERROR_MESSAGE 
    INTO repl_error, last_error
    FROM performance_schema.replication_applier_status_by_worker
    WHERE LAST_ERROR_NUMBER != 0
    LIMIT 1;
    
    IF repl_error != 0 THEN
        -- Intentar recuperación automática según tipo de error
        CASE 
            WHEN repl_error = 1062 THEN  -- Duplicate entry
                -- Saltar transacción duplicada
                SET @sql = CONCAT('SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1');
                PREPARE stmt FROM @sql;
                EXECUTE stmt;
                DEALLOCATE PREPARE stmt;
                
            WHEN repl_error = 1032 THEN  -- Can't find record
                -- Reconstruir datos faltantes
                CALL sp_reconstruct_missing_data();
                
            ELSE
                -- Error no manejado automáticamente, notificar
                CALL sp_send_alert('replication_error', 
                    CONCAT('Error ', repl_error, ': ', last_error));
        END CASE;
        
        -- Reiniciar esclavo
        STOP SLAVE;
        START SLAVE;
    END IF;
END $$
DELIMITER ;
```

---

## ⚙️ Automatización y Orquestación

### **1. Sistema de Alertas Inteligentes**
```sql
-- Tabla para configurar alertas
CREATE TABLE sistema.alert_config (
    id INT PRIMARY KEY AUTO_INCREMENT,
    metric_name VARCHAR(100) NOT NULL,
    threshold_value DECIMAL(10,2) NOT NULL,
    comparison_operator ENUM('>', '<', '=', '>=', '<=', '!=') NOT NULL,
    severity ENUM('LOW', 'MEDIUM', 'HIGH', 'CRITICAL') NOT NULL,
    check_interval_minutes INT DEFAULT 5,
    enabled BOOLEAN DEFAULT TRUE,
    notification_channels JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Procedimiento de monitoreo centralizado
DELIMITER $$
CREATE PROCEDURE sp_monitoring_engine()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE v_metric VARCHAR(100);
    DECLARE v_threshold DECIMAL(10,2);
    DECLARE v_operator VARCHAR(2);
    DECLARE v_severity VARCHAR(10);
    DECLARE v_current_value DECIMAL(10,2);
    
    DECLARE cur_alerts CURSOR FOR
        SELECT metric_name, threshold_value, comparison_operator, severity
        FROM sistema.alert_config 
        WHERE enabled = TRUE;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN cur_alerts;
    
    alert_loop: LOOP
        FETCH cur_alerts INTO v_metric, v_threshold, v_operator, v_severity;
        IF done THEN
            LEAVE alert_loop;
        END IF;
        
        -- Obtener valor actual de la métrica
        SET @sql = CONCAT('SELECT ', v_metric, ' INTO @current_val',
                         ' FROM sistema.metric_snapshot',
                         ' ORDER BY collected_at DESC LIMIT 1');
        PREPARE stmt FROM @sql;
        EXECUTE stmt;
        DEALLOCATE PREPARE stmt;
        
        SET v_current_value = @current_val;
        
        -- Evaluar condición
        SET @condition_met = FALSE;
        SET @eval_sql = CONCAT('SET @condition_met = ', 
                              v_current_value, ' ', v_operator, ' ', v_threshold);
        PREPARE stmt FROM @eval_sql;
        EXECUTE stmt;
        DEALLOCATE PREPARE stmt;
        
        IF @condition_met THEN
            -- Registrar alerta
            INSERT INTO sistema.alerts_triggered (
                alert_config_id, 
                metric_value, 
                threshold, 
                severity,
                triggered_at
            ) VALUES (
                (SELECT id FROM sistema.alert_config 
                 WHERE metric_name = v_metric LIMIT 1),
                v_current_value,
                v_threshold,
                v_severity,
                NOW()
            );
            
            -- Enviar notificaciones
            CALL sp_send_alert_notifications(v_metric, v_current_value, 
                                           v_threshold, v_severity);
        END IF;
    END LOOP;
    
    CLOSE cur_alerts;
END $$
DELIMITER ;
```

### **2. Orquestación de Tareas DBA**
```sql
-- Sistema de workflow para tareas DBA
CREATE TABLE sistema.dba_workflow (
    task_id INT PRIMARY KEY AUTO_INCREMENT,
    task_name VARCHAR(100) NOT NULL,
    task_type ENUM('MAINTENANCE', 'BACKUP', 'OPTIMIZATION', 'SECURITY', 'MONITORING'),
    schedule_cron VARCHAR(100),
    command TEXT NOT NULL,
    timeout_seconds INT DEFAULT 300,
    retry_count INT DEFAULT 3,
    enabled BOOLEAN DEFAULT TRUE,
    last_run TIMESTAMP NULL,
    last_status ENUM('SUCCESS', 'FAILED', 'RUNNING'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Ejecutor de tareas programadas
DELIMITER $$
CREATE PROCEDURE sp_dba_task_executor()
BEGIN
    DECLARE v_task_id INT;
    DECLARE v_task_name VARCHAR(100);
    DECLARE v_command TEXT;
    DECLARE v_timeout INT;
    DECLARE v_retries INT;
    DECLARE done INT DEFAULT FALSE;
    
    DECLARE cur_tasks CURSOR FOR
        SELECT task_id, task_name, command, timeout_seconds, retry_count
        FROM sistema.dba_workflow
        WHERE enabled = TRUE
          AND (last_run IS NULL 
               OR last_status = 'FAILED'
               OR (schedule_cron IS NOT NULL 
                   AND NOW() >= calculate_next_run(schedule_cron, last_run)));
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN cur_tasks;
    
    task_loop: LOOP
        FETCH cur_tasks INTO v_task_id, v_task_name, v_command, v_timeout, v_retries;
        IF done THEN
            LEAVE task_loop;
        END IF;
        
        -- Marcar como en ejecución
        UPDATE sistema.dba_workflow 
        SET last_run = NOW(), 
            last_status = 'RUNNING'
        WHERE task_id = v_task_id;
        
        -- Ejecutar comando con manejo de errores
        BEGIN
            DECLARE EXIT HANDLER FOR SQLEXCEPTION
            BEGIN
                UPDATE sistema.dba_workflow 
                SET last_status = 'FAILED'
                WHERE task_id = v_task_id;
                
                INSERT INTO sistema.task_errors (task_id, error_message, occurred_at)
                VALUES (v_task_id, CONCAT('Error ejecutando: ', v_task_name), NOW());
            END;
            
            -- Ejecutar el comando dinámico
            SET @sql = v_command;
            PREPARE stmt FROM @sql;
            EXECUTE stmt;
            DEALLOCATE PREPARE stmt;
            
            -- Marcar como exitoso
            UPDATE sistema.dba_workflow 
            SET last_status = 'SUCCESS'
            WHERE task_id = v_task_id;
            
        END;
    END LOOP;
    
    CLOSE cur_tasks;
END $$
DELIMITER ;
```

### **3. Automatización de Capacity Planning**
```sql
-- Sistema predictivo de capacidad
CREATE TABLE sistema.capacity_metrics (
    id INT PRIMARY KEY AUTO_INCREMENT,
    metric_date DATE NOT NULL,
    database_size_gb DECIMAL(10,2),
    table_count INT,
    row_count BIGINT,
    query_per_second DECIMAL(10,2),
    connection_count INT,
    buffer_pool_hit_rate DECIMAL(5,2),
    disk_io_per_second INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_metric_date (metric_date)
);

-- Procedimiento para proyección de crecimiento
DELIMITER $$
CREATE PROCEDURE sp_capacity_forecast(
    IN p_months_ahead INT
)
BEGIN
    -- Calcular tasa de crecimiento histórico
    WITH growth_rates AS (
        SELECT 
            metric_name,
            AVG(daily_growth) as avg_daily_growth,
            STDDEV(daily_growth) as std_daily_growth
        FROM (
            SELECT 
                'database_size' as metric_name,
                (database_size_gb - LAG(database_size_gb) 
                 OVER (ORDER BY metric_date)) / 
                 DATEDIFF(metric_date, LAG(metric_date) 
                 OVER (ORDER BY metric_date)) as daily_growth
            FROM sistema.capacity_metrics
            WHERE metric_date >= DATE_SUB(CURRENT_DATE, INTERVAL 90 DAY)
        ) AS daily_calc
        GROUP BY metric_name
    )
    -- Generar proyección
    SELECT 
        DATE_ADD(CURRENT_DATE, INTERVAL n.n MONTH) as forecast_date,
        mc.database_size_gb * 
            POWER(1 + (gr.avg_daily_growth / mc.database_size_gb), 
                  n.n * 30) as projected_size_gb,
        mc.connection_count * 
            POWER(1.05, n.n) as projected_connections,
        mc.query_per_second * 
            POWER(1.03, n.n) as projected_qps
    FROM sistema.capacity_metrics mc
    CROSS JOIN growth_rates gr
    CROSS JOIN (SELECT 0 as n UNION SELECT 1 UNION SELECT 2 
                UNION SELECT 3 UNION SELECT 6 UNION SELECT 12) n
    WHERE mc.metric_date = (
        SELECT MAX(metric_date) 
        FROM sistema.capacity_metrics
    )
    AND gr.metric_name = 'database_size'
    AND n.n <= p_months_ahead
    ORDER BY n.n;
END $$
DELIMITER ;
```

---

## 🏆 Habilidades de Administrador Nivel Experto

### **Competencias Desarrolladas**
| Habilidad | Descripción | Nivel Alcanzado |
|-----------|-------------|-----------------|
| **Replicación Avanzada** | Configuración multi-master, GTID, semi-sync | ⭐⭐⭐⭐⭐ |
| **Performance Tuning** | Optimización a nivel sistema y consulta | ⭐⭐⭐⭐⭐ |
| **Troubleshooting** | Diagnóstico de problemas complejos | ⭐⭐⭐⭐⭐ |
| **Automatización** | Scripting avanzado, orquestación | ⭐⭐⭐⭐ |
| **Capacity Planning** | Forecasting, planificación de recursos | ⭐⭐⭐⭐ |
| **High Availability** | Estrategias de failover, clustering | ⭐⭐⭐⭐ |

### **Herramientas Dominadas**
- **ProxySQL**: Balanceo de carga y routing inteligente
- **Percona Toolkit**: Herramientas avanzadas de administración
- **MySQL Shell**: Interfaz moderna con JavaScript/Python
- **Orchestrator**: Gestión visual de topologías de replicación
- **Prometheus + Grafana**: Monitoreo y dashboards avanzados
- **Ansible/Puppet**: Automatización de configuración

### **Buenas Prácticas Implementadas**
1. **Documentación exhaustiva** de configuraciones y procedimientos
2. **Versionado** de scripts y configuraciones DBA
3. **Testing** de cambios en entornos staging
4. **Rollback plans** para cada cambio en producción
5. **Comunicación proactiva** con equipos de desarrollo
6. **Continuous learning** sobre nuevas features de MySQL

---

## 👨‍💻 Autor

<div align="center">

**Darwin Manuel Ovalles Cesar**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Perfil_Profesional-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/darwin-manuel-ovalles-cesar-dev/)
[![GitHub](https://img.shields.io/badge/GitHub-Repositorios-black?style=flat&logo=github)](https://github.com/dovalless)

💼 **Administrador Senior de Bases de Datos**  
🎓 **Certificación Completa en Administración MySQL**  
🚀 **Especialista en Performance y Alta Disponibilidad**

*"La administración avanzada de MySQL no se trata solo de mantener el sistema funcionando, sino de optimizarlo para que escale, sea resiliente a fallos y proporcione insights valiosos para el negocio. Esta segunda parte del curso transforma administradores en arquitectos de bases de datos capaces de diseñar sistemas que soporten el crecimiento empresarial."*

**#MySQLExpert #DatabaseAdministration #HighAvailability #PerformanceTuning #DBAAdvanced**

</div>

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo [LICENSE](LICENSE) para más detalles.

```
MIT License
Copyright (c) 2024 Darwin Manuel Ovalles Cesar
```

---

## 🙏 Agradecimientos

- **Alura Latam** - Por un programa completo de administración de bases de datos
- **Comunidad MySQL** - Por las herramientas open source y el conocimiento compartido
- **Ingenieros de rendimiento** - Por las técnicas y mejores prácticas documentadas
- **Equipos de producción** - Por los desafíos reales que perfeccionan nuestras habilidades

<div align="center">

### ⭐ Un gran DBA anticipa problemas antes de que ocurran ⭐

### 🚀 ¡De administrador a arquitecto de bases de datos! 🚀

**Conocimiento experto aplicado con 🎯 y ⚡ por Darwin Ovalles**

---
*Administración avanzada de MySQL | Entornos empresariales | Alta disponibilidad*

</div>
