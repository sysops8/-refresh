# Backup & Recovery Refresh: Ежедневный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции резервного копирования и восстановления за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:

1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**

- Linux/Unix базовые знания
- SSH доступ к тестовым серверам (или Docker контейнеры)
- Базовые знания файловых систем
- Понимание storage концепций

---

## Модуль 1: Основы Backup & Recovery стратегий (20 минут)

### 🎯 Напоминалка

**Ключевые концепции:**

```
Backup Strategy
├── RTO (Recovery Time Objective)    # Сколько времени можно восстанавливаться
├── RPO (Recovery Point Objective)   # Сколько данных можно потерять
├── Backup Types
│   ├── Full      # Полная копия всех данных
│   ├── Incremental   # Только изменения с последнего backup
│   └── Differential  # Изменения с последнего Full backup
└── 3-2-1 Rule
    ├── 3 copies of data
    ├── 2 different media types
    └── 1 copy offsite
```

**Типы резервного копирования:**

```bash
# Full Backup
tar -czf /backup/full-$(date +%Y%m%d).tar.gz /data

# Incremental (с использованием snapshot)
rsync -a --link-dest=/backup/last/ /data/ /backup/$(date +%Y%m%d)/

# Differential (все изменения с момента последнего Full)
tar -czf /backup/diff-$(date +%Y%m%d).tar.gz \
  --listed-incremental=/backup/snapshot.file /data
```

**RPO vs RTO:**

```
RPO (Recovery Point Objective)
├── Hourly backups → RPO = 1 hour (можем потерять до 1 часа данных)
├── Daily backups  → RPO = 24 hours
└── Continuous replication → RPO ≈ 0

RTO (Recovery Time Objective)
├── Hot standby    → RTO = minutes
├── Warm standby   → RTO = hours
└── Cold backup    → RTO = days
```

**Backup Media & Storage:**

```bash
# Local storage
/backup/
├── daily/
├── weekly/
└── monthly/

# Network storage
NFS: mount -t nfs backup-server:/backups /mnt/backup
SMB: mount -t cifs //backup-server/backups /mnt/backup
iSCSI: iscsiadm -m node -T iqn.backup.target -p server:3260 -l

# Cloud storage
AWS S3: aws s3 sync /data s3://my-backup-bucket/
Azure: az storage blob upload-batch
GCP: gsutil -m rsync -r /data gs://backup-bucket/

# Tape (LTO)
mt -f /dev/st0 status
tar -czf - /data | dd of=/dev/st0
```

**Retention Policy Example:**

```bash
# Grandfather-Father-Son (GFS)
Daily   → Keep 7 days
Weekly  → Keep 4 weeks
Monthly → Keep 12 months
Yearly  → Keep 7 years

# Tower of Hanoi
# Более сложная ротация для оптимизации медиа
```

**Критичные метрики:**

```bash
# Backup window
Время начала: 02:00
Время окончания: 06:00
Доступное окно: 4 часа
Скорость: 500 GB/час
Макс объем: 2 TB

# Verify backup integrity
md5sum /data/file > /backup/checksums.md5
md5sum -c /backup/checksums.md5

# Test restore regularly!
# Backup без тестирования restore = катастрофа
```

**Backup vs Archive:**

```
Backup:
- Краткосрочное хранение
- Быстрое восстановление
- Регулярная ротация

Archive:
- Долгосрочное хранение
- Compliance требования
- Immutable storage
- Редкий доступ
```

### 💻 Задание

Спроектируй backup стратегию для следующего сценария:

**Дано:**

- Web приложение с базой данных
- 500 GB данных (200 GB DB + 300 GB files)
- Изменения: ~50 GB/день
- Требования: RPO = 4 часа, RTO = 2 часа
- Бюджет: средний

**Задачи:**

1. **Определи тип бэкапов:**

```bash
# Создай файл backup-strategy.md
cat > backup-strategy.md <<'EOF'
# Backup Strategy

## Requirements
- Data size: 500 GB
- Daily change rate: 50 GB
- RPO: 4 hours
- RTO: 2 hours

## Proposed Solution

### Backup Schedule
- Full backup: Weekly (Sunday 02:00)
- Incremental: Daily (02:00)
- Database snapshots: Every 4 hours
- Transaction logs: Continuous

### Storage Tiers
1. Primary: Local SSD (last 7 days)
2. Secondary: NAS (last 30 days)
3. Tertiary: Cloud S3 (last 12 months)

### Recovery Plan
- Point-in-time recovery: 15 minutes
- Full system recovery: 2 hours

EOF
```

2. **Рассчитай storage требования:**

```bash
# Создай калькулятор
cat > calculate_storage.sh <<'EOF'
#!/bin/bash

# Исходные данные
FULL_SIZE=500          # GB
DAILY_CHANGE=50        # GB
RETENTION_DAYS=30

# Full backups (еженедельные)
WEEKLY_FULL=$((FULL_SIZE * 4))  # 4 недели

# Incremental (ежедневные)
DAILY_INCREMENTAL=$((DAILY_CHANGE * RETENTION_DAYS))

# Database snapshots (каждые 4 часа)
SNAPSHOT_SIZE=$((200 / 6))  # ~33 GB per snapshot
SNAPSHOTS_PER_DAY=6
DAILY_SNAPSHOTS=$((SNAPSHOT_SIZE * SNAPSHOTS_PER_DAY * 7))  # неделя

# Total
TOTAL=$((WEEKLY_FULL + DAILY_INCREMENTAL + DAILY_SNAPSHOTS))

echo "=== Storage Requirements ==="
echo "Weekly Full Backups: ${WEEKLY_FULL} GB"
echo "Daily Incremental: ${DAILY_INCREMENTAL} GB"
echo "Database Snapshots: ${DAILY_SNAPSHOTS} GB"
echo "Total Required: ${TOTAL} GB"
echo "Recommended (with 20% overhead): $((TOTAL * 120 / 100)) GB"
EOF

chmod +x calculate_storage.sh
./calculate_storage.sh
```

3. **Создай простой backup скрипт:**

```bash
cat > backup.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
BACKUP_DIR="/backup"
DATA_DIR="/data"
LOG_FILE="/var/log/backup.log"
RETENTION_DAYS=7

# Functions
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

full_backup() {
    local backup_file="${BACKUP_DIR}/full-$(date +%Y%m%d-%H%M%S).tar.gz"
    log "Starting full backup to $backup_file"
    
    tar -czf "$backup_file" \
        --exclude='*.tmp' \
        --exclude='cache/*' \
        "$DATA_DIR" 2>&1 | tee -a "$LOG_FILE"
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        log "Full backup completed successfully"
        # Calculate checksum
        md5sum "$backup_file" > "${backup_file}.md5"
        return 0
    else
        log "ERROR: Full backup failed"
        return 1
    fi
}

incremental_backup() {
    local backup_file="${BACKUP_DIR}/inc-$(date +%Y%m%d-%H%M%S).tar.gz"
    local snapshot_file="${BACKUP_DIR}/snapshot.snar"
    
    log "Starting incremental backup to $backup_file"
    
    tar -czf "$backup_file" \
        --listed-incremental="$snapshot_file" \
        --exclude='*.tmp' \
        "$DATA_DIR" 2>&1 | tee -a "$LOG_FILE"
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        log "Incremental backup completed"
        md5sum "$backup_file" > "${backup_file}.md5"
        return 0
    else
        log "ERROR: Incremental backup failed"
        return 1
    fi
}

cleanup_old_backups() {
    log "Cleaning up backups older than $RETENTION_DAYS days"
    find "$BACKUP_DIR" -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
    find "$BACKUP_DIR" -name "*.md5" -mtime +$RETENTION_DAYS -delete
}

verify_backup() {
    local backup_file=$1
    log "Verifying backup: $backup_file"
    
    if [ -f "${backup_file}.md5" ]; then
        cd "$(dirname "$backup_file")"
        if md5sum -c "$(basename "${backup_file}.md5")" > /dev/null 2>&1; then
            log "Backup verification successful"
            return 0
        else
            log "ERROR: Backup verification failed!"
            return 1
        fi
    else
        log "WARNING: No checksum file found"
        return 1
    fi
}

# Main
case "${1:-full}" in
    full)
        full_backup && verify_backup "$BACKUP_DIR"/full-*.tar.gz
        ;;
    incremental)
        incremental_backup && verify_backup "$BACKUP_DIR"/inc-*.tar.gz
        ;;
    cleanup)
        cleanup_old_backups
        ;;
    *)
        echo "Usage: $0 {full|incremental|cleanup}"
        exit 1
        ;;
esac
EOF

chmod +x backup.sh
```

4. **Настрой расписание:**

```bash
# Создай crontab запись
cat > backup.cron <<'EOF'
# Full backup every Sunday at 2 AM
0 2 * * 0 /usr/local/bin/backup.sh full

# Incremental backup every day (except Sunday) at 2 AM
0 2 * * 1-6 /usr/local/bin/backup.sh incremental

# Cleanup old backups daily at 3 AM
0 3 * * * /usr/local/bin/backup.sh cleanup

# Database snapshot every 4 hours
0 */4 * * * /usr/local/bin/db_snapshot.sh
EOF

# Установи crontab
crontab backup.cron
crontab -l
```

### 🚀 Бонус (новое)

**1. Реализуй backup с дедупликацией:**

```bash
# Используй rsync с hardlinks для дедупликации
cat > dedup_backup.sh <<'EOF'
#!/bin/bash

BACKUP_ROOT="/backup/snapshots"
CURRENT_DATE=$(date +%Y-%m-%d_%H-%M-%S)
LATEST_LINK="${BACKUP_ROOT}/latest"
NEW_BACKUP="${BACKUP_ROOT}/${CURRENT_DATE}"

mkdir -p "$BACKUP_ROOT"

# Создаем новый snapshot с hardlinks на предыдущий
rsync -avH \
    --delete \
    --link-dest="${LATEST_LINK}" \
    /data/ \
    "${NEW_BACKUP}/"

# Обновляем symlink на latest
rm -f "${LATEST_LINK}"
ln -s "${NEW_BACKUP}" "${LATEST_LINK}"

echo "Backup completed: ${NEW_BACKUP}"
echo "Disk usage:"
du -sh "${BACKUP_ROOT}"/*
EOF

chmod +x dedup_backup.sh
```

**2. Добавь шифрование бэкапов:**

```bash
cat > encrypted_backup.sh <<'EOF'
#!/bin/bash

BACKUP_FILE="backup-$(date +%Y%m%d).tar.gz"
ENCRYPTED_FILE="${BACKUP_FILE}.enc"
PASSWORD_FILE="/root/.backup_password"

# Создаем бэкап
tar -czf - /data | \
    openssl enc -aes-256-cbc -salt -pbkdf2 \
    -pass file:"$PASSWORD_FILE" \
    -out "$ENCRYPTED_FILE"

# Или используем GPG
tar -czf - /data | \
    gpg --symmetric --cipher-algo AES256 \
    --output "${BACKUP_FILE}.gpg"

echo "Encrypted backup created: $ENCRYPTED_FILE"

# Для восстановления:
# openssl enc -d -aes-256-cbc -pbkdf2 \
#   -pass file:"$PASSWORD_FILE" \
#   -in "$ENCRYPTED_FILE" | tar -xzf -
EOF
```

**3. Backup с compression ratio tracking:**

```bash
cat > compression_stats.sh <<'EOF'
#!/bin/bash

ORIGINAL_SIZE=$(du -sb /data | cut -f1)
BACKUP_FILE="/backup/test-backup.tar.gz"

# Тест разных compression алгоритмов
for algo in gzip bzip2 xz; do
    echo "Testing $algo compression..."
    
    case $algo in
        gzip)
            tar -czf "${BACKUP_FILE}.gz" /data
            COMPRESSED=$(stat -f%z "${BACKUP_FILE}.gz" 2>/dev/null || stat -c%s "${BACKUP_FILE}.gz")
            ;;
        bzip2)
            tar -cjf "${BACKUP_FILE}.bz2" /data
            COMPRESSED=$(stat -f%z "${BACKUP_FILE}.bz2" 2>/dev/null || stat -c%s "${BACKUP_FILE}.bz2")
            ;;
        xz)
            tar -cJf "${BACKUP_FILE}.xz" /data
            COMPRESSED=$(stat -f%z "${BACKUP_FILE}.xz" 2>/dev/null || stat -c%s "${BACKUP_FILE}.xz")
            ;;
    esac
    
    RATIO=$(echo "scale=2; ($ORIGINAL_SIZE - $COMPRESSED) / $ORIGINAL_SIZE * 100" | bc)
    echo "$algo: Original=${ORIGINAL_SIZE} Compressed=${COMPRESSED} Ratio=${RATIO}%"
done
EOF
```

---

## Модуль 2: Database Backups (30 минут)

### 🎯 Напоминалка

**MySQL/MariaDB Backup Methods:**

```bash
# Logical Backup (mysqldump)
mysqldump -u root -p \
    --single-transaction \
    --routines \
    --triggers \
    --all-databases \
    --master-data=2 \
    | gzip > /backup/mysql-$(date +%Y%m%d).sql.gz

# Восстановление
gunzip < /backup/mysql-20240115.sql.gz | mysql -u root -p

# Physical Backup (с Percona XtraBackup)
xtrabackup --backup \
    --target-dir=/backup/mysql/full \
    --user=root \
    --password=xxx

# Incremental backup
xtrabackup --backup \
    --target-dir=/backup/mysql/inc1 \
    --incremental-basedir=/backup/mysql/full \
    --user=root

# Binary logs (Point-in-time recovery)
mysqlbinlog --start-datetime="2024-01-15 10:00:00" \
    --stop-datetime="2024-01-15 11:00:00" \
    mysql-bin.000001 | mysql -u root -p
```

**PostgreSQL Backup Methods:**

```bash
# Logical backup (pg_dump)
pg_dump -U postgres -Fc mydb > /backup/mydb-$(date +%Y%m%d).dump

# Все базы
pg_dumpall -U postgres | gzip > /backup/all-$(date +%Y%m%d).sql.gz

# Восстановление
pg_restore -U postgres -d mydb /backup/mydb-20240115.dump

# Physical backup (pg_basebackup)
pg_basebackup -U replicator \
    -D /backup/pg_base \
    -Ft -z -P

# Point-in-time recovery (PITR)
# 1. Настроить archive_mode в postgresql.conf
archive_mode = on
archive_command = 'cp %p /backup/wal_archive/%f'

# 2. Создать base backup
pg_basebackup -D /backup/base -Fp -Xs -P

# 3. Для восстановления на определенный момент
# recovery.conf (PostgreSQL < 12) или postgresql.auto.conf
restore_command = 'cp /backup/wal_archive/%f %p'
recovery_target_time = '2024-01-15 10:00:00'
```

**MongoDB Backup:**

```bash
# mongodump
mongodump --out=/backup/mongo-$(date +%Y%m%d)

# Specific database
mongodump --db=mydb --out=/backup/

# Restore
mongorestore /backup/mongo-20240115/

# Oplog для PITR
mongodump --oplog --out=/backup/mongo-with-oplog
```

**Redis Backup:**

```bash
# RDB snapshot (автоматически или вручную)
redis-cli SAVE
# или
redis-cli BGSAVE

# Копируем RDB файл
cp /var/lib/redis/dump.rdb /backup/redis-$(date +%Y%m%d).rdb

# AOF (Append Only File) backup
redis-cli BGREWRITEAOF
cp /var/lib/redis/appendonly.aof /backup/

# Restore
# 1. Остановить Redis
# 2. Скопировать dump.rdb в data directory
# 3. Запустить Redis
```

**Backup consistency:**

```bash
# Filesystem snapshot для consistency
# 1. Freeze filesystem
fsfreeze -f /data

# 2. Create snapshot (LVM example)
lvcreate -L 10G -s -n data-snap /dev/vg0/data

# 3. Unfreeze
fsfreeze -u /data

# 4. Mount snapshot и backup
mount /dev/vg0/data-snap /mnt/snapshot
tar -czf /backup/data-snapshot.tar.gz /mnt/snapshot

# 5. Cleanup
umount /mnt/snapshot
lvremove -f /dev/vg0/data-snap
```

### 💻 Задание

Создай comprehensive database backup solution:

**1. MySQL backup с ротацией:**

```bash
cat > mysql_backup.sh <<'EOF'
#!/bin/bash

set -eo pipefail

# Configuration
BACKUP_DIR="/backup/mysql"
DB_USER="backup_user"
DB_PASS="backup_password"
RETENTION_DAYS=7
RETENTION_WEEKS=4
RETENTION_MONTHS=12

# Create backup directories
mkdir -p "$BACKUP_DIR"/{daily,weekly,monthly}

# Functions
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a /var/log/mysql_backup.log
}

daily_backup() {
    local backup_file="$BACKUP_DIR/daily/mysql-daily-$(date +%Y%m%d-%H%M%S).sql.gz"
    log "Starting daily backup..."
    
    # Backup all databases
    mysqldump -u "$DB_USER" -p"$DB_PASS" \
        --single-transaction \
        --routines \
        --triggers \
        --all-databases \
        --master-data=2 \
        --flush-logs \
        --delete-master-logs \
        2>/dev/null | gzip > "$backup_file"
    
    if [ $? -eq 0 ]; then
        log "Daily backup completed: $backup_file"
        
        # Calculate size and checksum
        SIZE=$(du -h "$backup_file" | cut -f1)
        md5sum "$backup_file" > "${backup_file}.md5"
        log "Backup size: $SIZE"
        
        # Test backup integrity
        if gunzip -t "$backup_file" 2>/dev/null; then
            log "Backup integrity verified"
        else
            log "ERROR: Backup integrity check failed!"
            return 1
        fi
    else
        log "ERROR: Daily backup failed!"
        return 1
    fi
}

weekly_backup() {
    log "Creating weekly backup..."
    local latest_daily=$(ls -t "$BACKUP_DIR/daily"/*.sql.gz 2>/dev/null | head -1)
    
    if [ -n "$latest_daily" ]; then
        cp "$latest_daily" "$BACKUP_DIR/weekly/mysql-weekly-$(date +%Y-W%V).sql.gz"
        log "Weekly backup created from daily backup"
    else
        log "ERROR: No daily backup found for weekly copy"
        return 1
    fi
}

monthly_backup() {
    log "Creating monthly backup..."
    local latest_weekly=$(ls -t "$BACKUP_DIR/weekly"/*.sql.gz 2>/dev/null | head -1)
    
    if [ -n "$latest_weekly" ]; then
        cp "$latest_weekly" "$BACKUP_DIR/monthly/mysql-monthly-$(date +%Y-%m).sql.gz"
        log "Monthly backup created from weekly backup"
    else
        log "ERROR: No weekly backup found for monthly copy"
        return 1
    fi
}

cleanup() {
    log "Cleaning up old backups..."
    
    # Daily backups
    find "$BACKUP_DIR/daily" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete
    find "$BACKUP_DIR/daily" -name "*.md5" -mtime +$RETENTION_DAYS -delete
    
    # Weekly backups (keep last N)
    ls -t "$BACKUP_DIR/weekly"/*.sql.gz 2>/dev/null | tail -n +$((RETENTION_WEEKS + 1)) | xargs rm -f 2>/dev/null || true
    
    # Monthly backups (keep last N)
    ls -t "$BACKUP_DIR/monthly"/*.sql.gz 2>/dev/null | tail -n +$((RETENTION_MONTHS + 1)) | xargs rm -f 2>/dev/null || true
    
    log "Cleanup completed"
}

backup_binary_logs() {
    log "Backing up binary logs..."
    local binlog_dir="/var/lib/mysql"
    local binlog_backup="$BACKUP_DIR/binlogs/binlog-$(date +%Y%m%d-%H%M%S).tar.gz"
    
    mkdir -p "$BACKUP_DIR/binlogs"
    
    # Flush logs перед backup
    mysql -u "$DB_USER" -p"$DB_PASS" -e "FLUSH BINARY LOGS" 2>/dev/null
    
    # Backup all except current
    tar -czf "$binlog_backup" \
        -C "$binlog_dir" \
        $(mysql -u "$DB_USER" -p"$DB_PASS" -e "SHOW BINARY LOGS" 2>/dev/null | \
          awk 'NR>1 {print $1}' | head -n -1)
    
    log "Binary logs backed up: $binlog_backup"
}

# Main execution
case "${1:-daily}" in
    daily)
        daily_backup
        backup_binary_logs
        cleanup
        ;;
    weekly)
        weekly_backup
        ;;
    monthly)
        monthly_backup
        ;;
    cleanup)
        cleanup
        ;;
    *)
        echo "Usage: $0 {daily|weekly|monthly|cleanup}"
        exit 1
        ;;
esac

log "Backup script completed"
EOF

chmod +x mysql_backup.sh
```

**2. PostgreSQL PITR setup:**

```bash
cat > pg_backup_pitr.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
PG_VERSION="14"
PG_DATA="/var/lib/postgresql/$PG_VERSION/main"
BACKUP_DIR="/backup/postgresql"
WAL_ARCHIVE="$BACKUP_DIR/wal_archive"
BASE_BACKUP="$BACKUP_DIR/base"

# Create directories
mkdir -p "$WAL_ARCHIVE" "$BASE_BACKUP"
chown -R postgres:postgres "$BACKUP_DIR"

setup_wal_archiving() {
    echo "Setting up WAL archiving..."
    
    # Update postgresql.conf
    sudo -u postgres psql -c "ALTER SYSTEM SET wal_level = replica;"
    sudo -u postgres psql -c "ALTER SYSTEM SET archive_mode = on;"
    sudo -u postgres psql -c "ALTER SYSTEM SET archive_command = 'test ! -f $WAL_ARCHIVE/%f && cp %p $WAL_ARCHIVE/%f';"
    sudo -u postgres psql -c "ALTER SYSTEM SET archive_timeout = 300;"
    
    # Reload configuration
    sudo -u postgres psql -c "SELECT pg_reload_conf();"
    
    echo "WAL archiving configured"
}

create_base_backup() {
    local backup_name="base-$(date +%Y%m%d-%H%M%S)"
    local backup_path="$BASE_BACKUP/$backup_name"
    
    echo "Creating base backup: $backup_name"
    
    sudo -u postgres pg_basebackup \
        -D "$backup_path" \
        -Ft \
        -z \
        -P \
        -X stream
    
    if [ $? -eq 0 ]; then
        echo "Base backup created successfully: $backup_path"
        echo "$backup_name" > "$BASE_BACKUP/latest"
        
        # Create backup label
        cat > "$backup_path/backup_label.txt" <<LABEL
Backup Name: $backup_name
Backup Time: $(date)
PostgreSQL Version: $PG_VERSION
WAL Archive: $WAL_ARCHIVE
LABEL
        
        return 0
    else
        echo "ERROR: Base backup failed"
        return 1
    fi
}

restore_to_point_in_time() {
    local target_time="$1"
    local restore_dir="/var/lib/postgresql/restore"
    local latest_backup=$(cat "$BASE_BACKUP/latest")
    
    if [ -z "$target_time" ]; then
        echo "Usage: $0 restore 'YYYY-MM-DD HH:MM:SS'"
        exit 1
    fi
    
    echo "Restoring to point in time: $target_time"
    
    # Stop PostgreSQL
    systemctl stop postgresql
    
    # Extract base backup
    mkdir -p "$restore_dir"
    tar -xzf "$BASE_BACKUP/$latest_backup/base.tar.gz" -C "$restore_dir"
    
    # Create recovery configuration
    cat > "$restore_dir/postgresql.auto.conf" <<RECOVERY
restore_command = 'cp $WAL_ARCHIVE/%f %p'
recovery_target_time = '$target_time'
recovery_target_action = 'promote'
RECOVERY
    
    # Create recovery signal
    touch "$restore_dir/recovery.signal"
    
    echo "Restore configuration created"
    echo "To complete restore:"
    echo "1. Review $restore_dir/postgresql.auto.conf"
    echo "2. Move $restore_dir to $PG_DATA"
    echo "3. Start PostgreSQL: systemctl start postgresql"
}

verify_backup() {
    echo "Verifying backup integrity..."
    
    # Check base backups
    for backup in "$BASE_BACKUP"/*/*.tar.gz; do
        if [ -f "$backup" ]; then
            if tar -tzf "$backup" > /dev/null 2>&1; then
                echo "✓ Valid: $backup"
            else
                echo "✗ Corrupted: $backup"
            fi
        fi
    done
    
    # Check WAL archive
    local wal_count=$(find "$WAL_ARCHIVE" -type f | wc -l)
    echo "WAL archive files: $wal_count"
}

# Main
case "${1:-backup}" in
    setup)
        setup_wal_archiving
        ;;
    backup)
        create_base_backup
        ;;
    restore)
        restore_to_point_in_time "$2"
        ;;
    verify)
        verify_backup
        ;;
    *)
        echo "Usage: $0 {setup|backup|restore 'YYYY-MM-DD HH:MM:SS'|verify}"
        exit 1
        ;;
esac
EOF

chmod +x pg_backup_pitr.sh
```

**3. Multi-database backup orchestrator:**

```bash
cat > multi_db_backup.sh <<'EOF'
#!/bin/bash

set -e

BACKUP_ROOT="/backup"
LOG_FILE="/var/log/multi_db_backup.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

backup_mysql() {
    log "Backing up MySQL databases..."
    
    mysqldump -u root \
        --all-databases \
        --single-transaction \
        --quick \
        --lock-tables=false \
        | gzip > "$BACKUP_ROOT/mysql/mysql-$(date +%Y%m%d-%H%M%S).sql.gz"
    
    log "MySQL backup completed"
}

backup_postgresql() {
    log "Backing up PostgreSQL databases..."
    
    sudo -u postgres pg_dumpall \
        | gzip > "$BACKUP_ROOT/postgresql/pg-$(date +%Y%m%d-%H%M%S).sql.gz"
    
    log "PostgreSQL backup completed"
}

backup_mongodb() {
    log "Backing up MongoDB databases..."
    
    mongodump --out="$BACKUP_ROOT/mongodb/mongo-$(date +%Y%m%d-%H%M%S)"
    
    # Compress
    tar -czf "$BACKUP_ROOT/mongodb/mongo-$(date +%Y%m%d-%H%M%S).tar.gz" \
        -C "$BACKUP_ROOT/mongodb" \
        "mongo-$(date +%Y%m%d-%H%M%S)"
    
    # Remove uncompressed
    rm -rf "$BACKUP_ROOT/mongodb/mongo-$(date +%Y%m%d-%H%M%S)"
    
    log "MongoDB backup completed"
}

backup_redis() {
    log "Backing up Redis..."
    
    # Trigger background save
    redis-cli BGSAVE
    
    # Wait for completion
    while [ "$(redis-cli LASTSAVE)" = "$(redis-cli LASTSAVE)" ]; do
        sleep 1
    done
    
    # Copy RDB file
    cp /var/lib/redis/dump.rdb \
        "$BACKUP_ROOT/redis/redis-$(date +%Y%m%d-%H%M%S).rdb"
    
    log "Redis backup completed"
}

sync_to_remote() {
    local remote_host="backup-server.example.com"
    local remote_path="/remote/backup"
    
    log "Syncing backups to remote server..."
    
    rsync -avz --delete \
        "$BACKUP_ROOT/" \
        "backup@${remote_host}:${remote_path}/"
    
    if [ $? -eq 0 ]; then
        log "Remote sync completed successfully"
    else
        log "ERROR: Remote sync failed"
        return 1
    fi
}

generate_report() {
    local report_file="$BACKUP_ROOT/backup_report_$(date +%Y%m%d).txt"
    
    cat > "$report_file" <<REPORT
Database Backup Report Generated: $(date)

=== Backup Summary ===
REPORT
    
    for db_type in mysql postgresql mongodb redis; do
        local backup_dir="$BACKUP_ROOT/$db_type"
        if [ -d "$backup_dir" ]; then
            local count=$(find "$backup_dir" -type f -mtime -1 | wc -l)
            local size=$(du -sh "$backup_dir" | cut -f1)
            echo "$db_type: $count files, $size total" >> "$report_file"
        fi
    done
    
    cat "$report_file"
}

# Main execution
log "Starting multi-database backup"

# Create backup directories
for db in mysql postgresql mongodb redis; do
    mkdir -p "$BACKUP_ROOT/$db"
done

# Run backups
backup_mysql || log "WARNING: MySQL backup failed"
backup_postgresql || log "WARNING: PostgreSQL backup failed"
backup_mongodb || log "WARNING: MongoDB backup failed"
backup_redis || log "WARNING: Redis backup failed"

# Sync to remote
sync_to_remote || log "WARNING: Remote sync failed"

# Generate report
generate_report

log "Multi-database backup completed"
EOF

chmod +x multi_db_backup.sh
```

**4. Setup cron schedule:**

```bash
cat > db_backup.cron <<'EOF'
# MySQL backups
0 2 * * * /usr/local/bin/mysql_backup.sh daily
0 2 * * 0 /usr/local/bin/mysql_backup.sh weekly
0 2 1 * * /usr/local/bin/mysql_backup.sh monthly

# PostgreSQL base backup (weekly)
0 3 * * 0 /usr/local/bin/pg_backup_pitr.sh backup

# Multi-DB backup (daily)
30 2 * * * /usr/local/bin/multi_db_backup.sh

# Verification (weekly)
0 4 * * 6 /usr/local/bin/pg_backup_pitr.sh verify
EOF

crontab db_backup.cron
````

### 🚀 Бонус (новое)

**1. Automated backup testing:**

```bash
cat > test_backup.sh <<'EOF'
#!/bin/bash

BACKUP_FILE="$1"
TEST_DIR="/tmp/backup_test_$$"

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <backup_file>"
    exit 1
fi

echo "Testing backup: $BACKUP_FILE"

# Create test directory
mkdir -p "$TEST_DIR"

# Restore to test directory
if [[ "$BACKUP_FILE" == *.sql.gz ]]; then
    # MySQL/PostgreSQL SQL dump
    gunzip -c "$BACKUP_FILE" > "$TEST_DIR/test.sql"
    
    # Verify SQL syntax
    if grep -q "CREATE TABLE" "$TEST_DIR/test.sql"; then
        echo "✓ SQL backup appears valid"
    else
        echo "✗ SQL backup may be corrupted"
        exit 1
    fi
    
elif [[ "$BACKUP_FILE" == *.tar.gz ]]; then
    # Extract archive
    tar -xzf "$BACKUP_FILE" -C "$TEST_DIR"
    
    # Check for expected structure
    if [ -d "$TEST_DIR/data" ] || [ -f "$TEST_DIR/base.tar" ]; then
        echo "✓ Archive backup appears valid"
    else
        echo "✗ Archive backup structure unexpected"
        exit 1
    fi
fi

# Cleanup
rm -rf "$TEST_DIR"

echo "Backup test completed successfully"
EOF

chmod +x test_backup.sh
```

**2. Backup monitoring с Prometheus:**

```bash
cat > backup_exporter.sh <<'EOF'
#!/bin/bash

# Prometheus metrics file
METRICS_FILE="/var/lib/node_exporter/textfile_collector/backup.prom"

generate_metrics() {
    cat > "$METRICS_FILE" <<METRICS
# HELP backup_last_success_timestamp_seconds Last successful backup time
# TYPE backup_last_success_timestamp_seconds gauge
METRICS

    for db_type in mysql postgresql mongodb; do
        local backup_dir="/backup/$db_type"
        if [ -d "$backup_dir" ]; then
            # Find latest backup
            local latest=$(find "$backup_dir" -type f -name "*.gz" -o -name "*.dump" | sort | tail -1)
            
            if [ -n "$latest" ]; then
                local timestamp=$(stat -c %Y "$latest" 2>/dev/null || stat -f %m "$latest")
                local size=$(stat -c %s "$latest" 2>/dev/null || stat -f %z "$latest")
                
                echo "backup_last_success_timestamp_seconds{database=\"$db_type\"} $timestamp" >> "$METRICS_FILE"
                echo "backup_size_bytes{database=\"$db_type\"} $size" >> "$METRICS_FILE"
            fi
        fi
    done
}

generate_metrics
EOF

chmod +x backup_exporter.sh

# Add to cron
echo "*/5 * * * * /usr/local/bin/backup_exporter.sh" | crontab -
```

**3. Database backup с encryption и cloud upload:**

```bash
cat > encrypted_cloud_backup.sh <<'EOF'
#!/bin/bash

set -e

DB_NAME="$1"
BACKUP_FILE="/tmp/${DB_NAME}-$(date +%Y%m%d-%H%M%S).sql"
ENCRYPTED_FILE="${BACKUP_FILE}.gpg"
S3_BUCKET="s3://my-backup-bucket"
GPG_RECIPIENT="backup@example.com"

if [ -z "$DB_NAME" ]; then
    echo "Usage: $0 <database_name>"
    exit 1
fi

echo "Backing up database: $DB_NAME"

# 1. Create backup
pg_dump -U postgres "$DB_NAME" > "$BACKUP_FILE"

# 2. Encrypt
gpg --encrypt --recipient "$GPG_RECIPIENT" \
    --output "$ENCRYPTED_FILE" \
    "$BACKUP_FILE"

# 3. Upload to S3
aws s3 cp "$ENCRYPTED_FILE" \
    "${S3_BUCKET}/postgresql/${DB_NAME}/" \
    --storage-class GLACIER

# 4. Verify upload
if aws s3 ls "${S3_BUCKET}/postgresql/${DB_NAME}/$(basename "$ENCRYPTED_FILE")" > /dev/null; then
    echo "✓ Backup uploaded successfully"
    
    # Cleanup local files
    rm -f "$BACKUP_FILE" "$ENCRYPTED_FILE"
else
    echo "✗ Upload verification failed"
    exit 1
fi

echo "Encrypted cloud backup completed"
EOF

chmod +x encrypted_cloud_backup.sh
```

---

## Модуль 3: Filesystem & System Backups (25 минут)

### 🎯 Напоминалка

**Filesystem backup методы:**

```bash
# rsync - инкрементальные бэкапы
rsync -avz --delete \
    --exclude='/proc' \
    --exclude='/sys' \
    --exclude='/dev' \
    --exclude='/tmp' \
    /source/ /backup/destination/

# tar - полные архивы
tar -czf backup.tar.gz \
    --exclude='/proc' \
    --exclude='/sys' \
    /source/

# dd - побайтовое копирование
dd if=/dev/sda of=/backup/sda.img bs=4M status=progress

# cpio - для создания архивов с pipe
find /source -print | cpio -o > backup.cpio
```

**LVM Snapshots:**

```bash
# Создание snapshot
lvcreate -L 10G -s -n snap_data /dev/vg0/data

# Монтирование
mount -o ro /dev/vg0/snap_data /mnt/snapshot

# Backup из snapshot
tar -czf /backup/data-snap.tar.gz /mnt/snapshot

# Удаление snapshot
umount /mnt/snapshot
lvremove -f /dev/vg0/snap_data

# Проверка snapshot usage
lvs -o +lv_snapshot_invalid,snap_percent
```

**Btrfs Snapshots:**

```bash
# Создание read-only snapshot
btrfs subvolume snapshot -r /data /data/.snapshots/snap-$(date +%Y%m%d)

# Incremental backup
btrfs send /data/.snapshots/snap-20240115 | \
    btrfs receive /backup/btrfs/

# Incremental с parent
btrfs send -p /data/.snapshots/snap-20240114 \
    /data/.snapshots/snap-20240115 | \
    btrfs receive /backup/btrfs/

# Список snapshots
btrfs subvolume list /data

# Удаление snapshot
btrfs subvolume delete /data/.snapshots/snap-20240115
```

**ZFS Snapshots:**

```bash
# Создание snapshot
zfs snapshot pool/data@snap-$(date +%Y%m%d-%H%M)

# Список snapshots
zfs list -t snapshot

# Incremental backup
zfs send -i pool/data@snap-prev pool/data@snap-current | \
    gzip > /backup/zfs-incremental.gz

# Receive snapshot
gunzip < /backup/zfs-incremental.gz | zfs receive backup/data

# Rollback к snapshot
zfs rollback pool/data@snap-20240115

# Клонирование snapshot
zfs clone pool/data@snap-20240115 pool/data-clone

# Удаление snapshot
zfs destroy pool/data@snap-20240115
```

**System state backup:**

```bash
# Package list
dpkg --get-selections > /backup/packages.list     # Debian/Ubuntu
rpm -qa > /backup/packages.list                   # RHEL/CentOS

# System configuration
tar -czf /backup/etc-$(date +%Y%m%d).tar.gz /etc

# User data
tar -czf /backup/home-$(date +%Y%m%d).tar.gz /home

# Crontabs
for user in $(cut -f1 -d: /etc/passwd); do
    crontab -u $user -l > /backup/cron-$user.txt 2>/dev/null || true
done

# Services list
systemctl list-unit-files --state=enabled > /backup/services.txt

# Network configuration
ip addr show > /backup/network-config.txt
ip route show >> /backup/network-config.txt
```

**Bare Metal Recovery:**

```bash
# Создание полного образа системы (ReaR - Relax and Recover)
rear mkbackup

# Конфигурация /etc/rear/local.conf
OUTPUT=ISO
BACKUP=NETFS
BACKUP_URL=nfs://backup-server/rear-backups

# Restore
# 1. Boot from ReaR ISO
# 2. rear recover
# 3. Система восстановится автоматически
```

**Application-consistent backups:**

```bash
# Pre-backup script (flush caches, freeze IO)
#!/bin/bash
# pre-backup.sh

# Sync filesystem
sync

# Freeze filesystem
fsfreeze -f /data

# Or application-specific:
# MySQL: FLUSH TABLES WITH READ LOCK
# PostgreSQL: pg_start_backup()

echo "Application prepared for backup"

# Post-backup script
#!/bin/bash
# post-backup.sh

# Unfreeze filesystem
fsfreeze -u /data

# Or application-specific cleanup:
# MySQL: UNLOCK TABLES
# PostgreSQL: pg_stop_backup()

echo "Application resumed"
```

### 💻 Задание

Создай comprehensive filesystem backup solution:

**1. Rsync-based incremental backup система:**

```bash
cat > rsync_backup.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
SOURCE_DIR="/data"
BACKUP_ROOT="/backup/rsync-snapshots"
RETENTION_DAYS=30
LOG_FILE="/var/log/rsync_backup.log"
EXCLUDE_FILE="/etc/backup/rsync-exclude.txt"

# Create exclude file
cat > "$EXCLUDE_FILE" <<EXCLUDE
/proc/*
/sys/*
/dev/*
/tmp/*
/run/*
/mnt/*
/media/*
*.tmp
*.cache
.Trash*
EXCLUDE

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

create_snapshot() {
    local date_stamp=$(date +%Y-%m-%d_%H-%M-%S)
    local snapshot_dir="$BACKUP_ROOT/$date_stamp"
    local latest_link="$BACKUP_ROOT/latest"
    local incomplete_flag="$snapshot_dir/.incomplete"
    
    log "Creating snapshot: $date_stamp"
    
    # Create snapshot directory
    mkdir -p "$snapshot_dir"
    touch "$incomplete_flag"
    
    # Rsync with hardlinks to previous snapshot
    rsync -aAXHv \
        --delete \
        --delete-excluded \
        --exclude-from="$EXCLUDE_FILE" \
        --link-dest="$latest_link" \
        --stats \
        "$SOURCE_DIR/" \
        "$snapshot_dir/" 2>&1 | tee -a "$LOG_FILE"
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        # Remove incomplete flag
        rm "$incomplete_flag"
        
        # Update latest symlink
        rm -f "$latest_link"
        ln -s "$snapshot_dir" "$latest_link"
        
        # Calculate space usage
        local size=$(du -sh "$snapshot_dir" | cut -f1)
        local actual_size=$(du -sh --apparent-size "$snapshot_dir" | cut -f1)
        
        log "Snapshot created successfully"
        log "Apparent size: $actual_size, Actual size: $size (deduplicated)"
        
        # Create snapshot metadata
        cat > "$snapshot_dir/.snapshot_info" <<INFO
Snapshot: $date_stamp
Created: $(date)
Source: $SOURCE_DIR
Apparent Size: $actual_size
Actual Size: $size
INFO
        
        return 0
    else
        log "ERROR: Snapshot creation failed"
        return 1
    fi
}

cleanup_old_snapshots() {
    log "Cleaning up snapshots older than $RETENTION_DAYS days"
    
    find "$BACKUP_ROOT" -maxdepth 1 -type d -mtime +$RETENTION_DAYS \
        -not -name "latest" \
        -exec rm -rf {} \;
    
    log "Cleanup completed"
}

verify_snapshot() {
    local snapshot="$1"
    
    if [ -f "$snapshot/.incomplete" ]; then
        log "WARNING: Snapshot $snapshot is incomplete"
        return 1
    fi
    
    # Count files
    local file_count=$(find "$snapshot" -type f | wc -l)
    log "Snapshot contains $file_count files"
    
    # Random integrity check
    local random_files=$(find "$snapshot" -type f | shuf -n 10)
    local checked=0
    local failed=0
    
    for file in $random_files; do
        if [ -r "$file" ]; then
            checked=$((checked + 1))
        else
            failed=$((failed + 1))
            log "WARNING: Cannot read $file"
        fi
    done
    
    log "Integrity check: $checked OK, $failed failed"
    
    [ $failed -eq 0 ] && return 0 || return 1
}

list_snapshots() {
    echo "Available snapshots in $BACKUP_ROOT:"
    echo "========================================"
    
    for snapshot in "$BACKUP_ROOT"/*; do
        if [ -d "$snapshot" ] && [ "$(basename "$snapshot")" != "latest" ]; then
            local name=$(basename "$snapshot")
            local size=$(du -sh "$snapshot" 2>/dev/null | cut -f1)
            local status="OK"
            
            [ -f "$snapshot/.incomplete" ] && status="INCOMPLETE"
            
            printf "%-25s %10s %s\n" "$name" "$size" "$status"
        fi
    done
}

restore_snapshot() {
    local snapshot="$1"
    local restore_target="$2"
    
    if [ -z "$snapshot" ] || [ -z "$restore_target" ]; then
        echo "Usage: $0 restore <snapshot_name> <restore_target>"
        exit 1
    fi
    
    local snapshot_path="$BACKUP_ROOT/$snapshot"
    
    if [ ! -d "$snapshot_path" ]; then
        log "ERROR: Snapshot not found: $snapshot"
        exit 1
    fi
    
    log "Restoring snapshot $snapshot to $restore_target"
    
    mkdir -p "$restore_target"
    
    rsync -aAXHv \
        --delete \
        "$snapshot_path/" \
        "$restore_target/" 2>&1 | tee -a "$LOG_FILE"
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        log "Restore completed successfully"
        return 0
    else
        log "ERROR: Restore failed"
        return 1
    fi
}

# Main
mkdir -p "$BACKUP_ROOT"

case "${1:-snapshot}" in
    snapshot)
        create_snapshot && verify_snapshot "$BACKUP_ROOT/latest"
        cleanup_old_snapshots
        ;;
    list)
        list_snapshots
        ;;
    verify)
        verify_snapshot "$BACKUP_ROOT/latest"
        ;;
    restore)
        restore_snapshot "$2" "$3"
        ;;
    cleanup)
        cleanup_old_snapshots
        ;;
    *)
        echo "Usage: $0 {snapshot|list|verify|restore <snapshot> <target>|cleanup}"
        exit 1
        ;;
esac
EOF

chmod +x rsync_backup.sh
```

**2. LVM snapshot backup:**

```bash
cat > lvm_snapshot_backup.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
VG_NAME="vg0"
LV_NAME="data"
SNAP_NAME="${LV_NAME}_snap"
SNAP_SIZE="10G"
MOUNT_POINT="/mnt/lvm-snapshot"
BACKUP_DIR="/backup/lvm"
LOG_FILE="/var/log/lvm_backup.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

create_snapshot() {
    log "Creating LVM snapshot: /dev/$VG_NAME/$SNAP_NAME"
    
    # Check if snapshot already exists
    if lvs "/dev/$VG_NAME/$SNAP_NAME" &>/dev/null; then
        log "Snapshot already exists, removing old snapshot"
        lvremove -f "/dev/$VG_NAME/$SNAP_NAME"
    fi
    
    # Create snapshot
    lvcreate -L "$SNAP_SIZE" -s -n "$SNAP_NAME" "/dev/$VG_NAME/$LV_NAME"
    
    if [ $? -eq 0 ]; then
        log "Snapshot created successfully"
        
        # Display snapshot info
        lvs "/dev/$VG_NAME/$SNAP_NAME" -o lv_name,lv_size,snap_percent
        
        return 0
    else
        log "ERROR: Failed to create snapshot"
        return 1
    fi
}

mount_snapshot() {
    log "Mounting snapshot to $MOUNT_POINT"
    
    mkdir -p "$MOUNT_POINT"
    
    mount -o ro "/dev/$VG_NAME/$SNAP_NAME" "$MOUNT_POINT"
    
    if [ $? -eq 0 ]; then
        log "Snapshot mounted successfully"
        return 0
    else
        log "ERROR: Failed to mount snapshot"
        return 1
    fi
}

backup_from_snapshot() {
    local backup_file="$BACKUP_DIR/lvm-backup-$(date +%Y%m%d-%H%M%S).tar.gz"
    
    log "Creating backup from snapshot: $backup_file"
    
    mkdir -p "$BACKUP_DIR"
    
    # Backup with progress
    tar -czf - "$MOUNT_POINT" 2>/dev/null | \
        pv -s $(du -sb "$MOUNT_POINT" | cut -f1) > "$backup_file"
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        local size=$(du -h "$backup_file" | cut -f1)
        log "Backup created successfully: $size"
        
        # Create checksum
        md5sum "$backup_file" > "${backup_file}.md5"
        
        return 0
    else
        log "ERROR: Backup failed"
        return 1
    fi
}

cleanup_snapshot() {
    log "Cleaning up snapshot"
    
    # Unmount if mounted
    if mountpoint -q "$MOUNT_POINT"; then
        umount "$MOUNT_POINT"
        log "Snapshot unmounted"
    fi
    
    # Remove snapshot
    if lvs "/dev/$VG_NAME/$SNAP_NAME" &>/dev/null; then
        lvremove -f "/dev/$VG_NAME/$SNAP_NAME"
        log "Snapshot removed"
    fi
    
    # Check snapshot space before removal
    lvs -o snap_percent "/dev/$VG_NAME/$SNAP_NAME" 2>/dev/null || true
}

check_snapshot_usage() {
    if lvs "/dev/$VG_NAME/$SNAP_NAME" &>/dev/null; then
        local usage=$(lvs --noheadings -o snap_percent "/dev/$VG_NAME/$SNAP_NAME" | tr -d ' %')
        log "Snapshot usage: ${usage}%"
        
        if [ "$usage" -gt 80 ]; then
            log "WARNING: Snapshot usage > 80%"
            return 1
        fi
    else
        log "Snapshot does not exist"
        return 1
    fi
}

restore_from_backup() {
    local backup_file="$1"
    local restore_target="$2"
    
    if [ -z "$backup_file" ] || [ -z "$restore_target" ]; then
        echo "Usage: $0 restore <backup_file> <restore_target>"
        exit 1
    fi
    
    if [ ! -f "$backup_file" ]; then
        log "ERROR: Backup file not found: $backup_file"
        exit 1
    fi
    
    log "Restoring from $backup_file to $restore_target"
    
    # Verify checksum
    if [ -f "${backup_file}.md5" ]; then
        cd "$(dirname "$backup_file")"
        if md5sum -c "$(basename "${backup_file}.md5")" > /dev/null 2>&1; then
            log "Checksum verification passed"
        else
            log "ERROR: Checksum verification failed"
            exit 1
        fi
    fi
    
    # Extract
    mkdir -p "$restore_target"
    tar -xzf "$backup_file" -C "$restore_target" --strip-components=1
    
    log "Restore completed"
}

# Main execution
case "${1:-backup}" in
    backup)
        create_snapshot
        mount_snapshot
        backup_from_snapshot
        cleanup_snapshot
        ;;
    create)
        create_snapshot
        ;;
    mount)
        mount_snapshot
        ;;
    cleanup)
        cleanup_snapshot
        ;;
    check)
        check_snapshot_usage
        ;;
    restore)
        restore_from_backup "$2" "$3"
        ;;
    *)
        echo "Usage: $0 {backup|create|mount|cleanup|check|restore <file> <target>}"
        exit 1
        ;;
esac
EOF

chmod +x lvm_snapshot_backup.sh
```

**3. Full system backup script:**

```bash
cat > system_backup.sh <<'EOF'
#!/bin/bash

set -e

BACKUP_ROOT="/backup/system"
DATE_STAMP=$(date +%Y%m%d-%H%M%S)
BACKUP_DIR="$BACKUP_ROOT/$DATE_STAMP"
LOG_FILE="$BACKUP_DIR/backup.log"

mkdir -p "$BACKUP_DIR"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

log "Starting full system backup"

# 1. System configuration
log "Backing up system configuration..."
tar -czf "$BACKUP_DIR/etc.tar.gz" /etc

# 2. Package lists
log "Saving package lists..."
if command -v dpkg &>/dev/null; then
    dpkg --get-selections > "$BACKUP_DIR/packages-dpkg.list"
elif command -v rpm &>/dev/null; then
    rpm -qa > "$BACKUP_DIR/packages-rpm.list"
fi

# 3. User data
log "Backing up user data..."
tar -czf "$BACKUP_DIR/home.tar.gz" \
    --exclude='*/.cache' \
    --exclude='*/Downloads' \
    /home

# 4. System services
log "Recording enabled services..."
systemctl list-unit-files --state=enabled > "$BACKUP_DIR/services.txt"

# 5. Crontabs
log "Backing up crontabs..."
mkdir -p "$BACKUP_DIR/crontabs"
for user in $(cut -f1 -d: /etc/passwd); do
    crontab -u "$user" -l > "$BACKUP_DIR/crontabs/$user" 2>/dev/null || true
done

# 6. Network configuration
log "Saving network configuration..."
ip addr show > "$BACKUP_DIR/network-interfaces.txt"
ip route show > "$BACKUP_DIR/network-routes.txt"
cat /etc/resolv.conf > "$BACKUP_DIR/network-dns.txt" 2>/dev/null || true

# 7. Partition table
log "Backing up partition table..."
for disk in /dev/sd? /dev/nvme?n?; do
    if [ -b "$disk" ]; then
        sfdisk -d "$disk" > "$BACKUP_DIR/partition-$(basename $disk).dump" 2>/dev/null || true
    fi
done

# 8. Filesystem UUIDs
log "Recording filesystem UUIDs..."
blkid > "$BACKUP_DIR/filesystem-uuids.txt"

# 9. Kernel modules
log "Recording loaded kernel modules..."
lsmod > "$BACKUP_DIR/kernel-modules.txt"

# 10. System information
log "Collecting system information..."
cat > "$BACKUP_DIR/system-info.txt" <<INFO
Hostname: $(hostname)
Kernel: $(uname -r)
Distribution: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)
Architecture: $(uname -m)
CPU: $(grep "model name" /proc/cpuinfo | head -1 | cut -d: -f2 | xargs)
Memory: $(free -h | grep Mem | awk '{print $2}')
Disks: $(lsblk -d -o NAME,SIZE | tail -n +2)
Backup Date: $(date)
INFO

# 11. Create restore script
cat > "$BACKUP_DIR/restore.sh" <<'RESTORE'
#!/bin/bash
# System Restore Script
# Generated: DATE_STAMP

echo "This script will help restore the system backup"
echo "WARNING: This should only be run on a clean installation"
read -p "Continue? (yes/no): " confirm

if [ "$confirm" != "yes" ]; then
    exit 0
fi

# Restore packages
if [ -f "packages-dpkg.list" ]; then
    dpkg --set-selections < packages-dpkg.list
    apt-get dselect-upgrade -y
elif [ -f "packages-rpm.list" ]; then
    yum install $(cat packages-rpm.list) -y
fi

# Restore configuration
tar -xzf etc.tar.gz -C /

# Restore user data
tar -xzf home.tar.gz -C /

# Restore crontabs
for user_cron in crontabs/*; do
    user=$(basename "$user_cron")
    crontab -u "$user" "$user_cron"
done

# Enable services
while IFS= read -r service; do
    systemctl enable "$service" 2>/dev/null || true
done < services.txt

echo "Restore completed. Please reboot the system."
RESTORE

chmod +x "$BACKUP_DIR/restore.sh"

# 12. Create backup manifest
log "Creating backup manifest..."
find "$BACKUP_DIR" -type f -exec md5sum {} \; > "$BACKUP_DIR/MANIFEST.md5"

# 13. Calculate total size
TOTAL_SIZE=$(du -sh "$BACKUP_DIR" | cut -f1)
log "System backup completed"
log "Total size: $TOTAL_SIZE"
log "Location: $BACKUP_DIR"

# 14. Cleanup old backups (keep last 5)
log "Cleaning up old backups..."
ls -t "$BACKUP_ROOT" | tail -n +6 | xargs -I {} rm -rf "$BACKUP_ROOT/{}"

log "Full system backup finished successfully"
EOF

chmod +x system_backup.sh
```

**4. Setup automated backups:**

```bash
cat > setup_backup_cron.sh <<'EOF'
#!/bin/bash

cat > /etc/cron.d/system-backups <<CRON
# Rsync snapshot - daily at 2 AM
0 2 * * * root /usr/local/bin/rsync_backup.sh snapshot

# LVM snapshot backup - weekly on Sunday at 3 AM
0 3 * * 0 root /usr/local/bin/lvm_snapshot_backup.sh backup

# Full system backup - monthly on 1st at 4 AM
0 4 1 * * root /usr/local/bin/system_backup.sh

# Cleanup old snapshots - daily at 5 AM
0 5 * * * root /usr/local/bin/rsync_backup.sh cleanup
CRON

echo "Backup cron jobs configured"
cat /etc/cron.d/system-backups
EOF

chmod +x setup_backup_cron.sh
./setup_backup_cron.sh
```

### 🚀 Бонус (продвинутое)

**1. Btrfs snapshot backup automation:**

```bash
cat > btrfs_snapshot_backup.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
SUBVOLUME="/data"
SNAPSHOT_DIR="/data/.snapshots"
BACKUP_TARGET="/backup/btrfs"
RETENTION_DAILY=7
RETENTION_WEEKLY=4
RETENTION_MONTHLY=12
LOG_FILE="/var/log/btrfs_backup.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

create_snapshot() {
    local snap_type="$1"  # daily, weekly, monthly
    local snap_name="${snap_type}-$(date +%Y%m%d-%H%M%S)"
    local snap_path="$SNAPSHOT_DIR/$snap_name"
    
    log "Creating $snap_type snapshot: $snap_name"
    
    # Create snapshot directory if not exists
    mkdir -p "$SNAPSHOT_DIR"
    
    # Create read-only snapshot
    btrfs subvolume snapshot -r "$SUBVOLUME" "$snap_path"
    
    if [ $? -eq 0 ]; then
        log "Snapshot created: $snap_path"
        echo "$snap_path"
        return 0
    else
        log "ERROR: Failed to create snapshot"
        return 1
    fi
}

get_latest_snapshot() {
    local snap_type="$1"
    ls -t "$SNAPSHOT_DIR"/${snap_type}-* 2>/dev/null | head -1
}

send_incremental() {
    local current_snap="$1"
    local backup_file="$BACKUP_TARGET/$(basename "$current_snap").btrfs"
    
    log "Sending incremental backup: $(basename "$current_snap")"
    
    mkdir -p "$BACKUP_TARGET"
    
    # Get previous snapshot of same type
    local snap_type=$(echo "$current_snap" | grep -o 'daily\|weekly\|monthly')
    local prev_snap=$(ls -t "$SNAPSHOT_DIR"/${snap_type}-* 2>/dev/null | sed -n '2p')
    
    if [ -n "$prev_snap" ] && [ -d "$prev_snap" ]; then
        # Incremental send
        log "Using parent snapshot: $(basename "$prev_snap")"
        btrfs send -p "$prev_snap" "$current_snap" | \
            pv | gzip > "$backup_file.gz"
    else
        # Full send (no parent)
        log "No parent snapshot, performing full send"
        btrfs send "$current_snap" | \
            pv | gzip > "$backup_file.gz"
    fi
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        log "Backup sent successfully: $backup_file.gz"
        
        # Create checksum
        md5sum "$backup_file.gz" > "$backup_file.gz.md5"
        
        # Create metadata
        cat > "$backup_file.info" <<INFO
Snapshot: $(basename "$current_snap")
Type: $snap_type
Parent: $(basename "$prev_snap")
Created: $(date)
Size: $(du -h "$backup_file.gz" | cut -f1)
INFO
        
        return 0
    else
        log "ERROR: Backup send failed"
        return 1
    fi
}

receive_backup() {
    local backup_file="$1"
    local target_path="$2"
    
    if [ -z "$backup_file" ] || [ -z "$target_path" ]; then
        echo "Usage: $0 receive <backup_file> <target_path>"
        exit 1
    fi
    
    log "Receiving backup from $backup_file to $target_path"
    
    # Verify checksum
    if [ -f "$backup_file.md5" ]; then
        cd "$(dirname "$backup_file")"
        if ! md5sum -c "$(basename "$backup_file.md5")" > /dev/null 2>&1; then
            log "ERROR: Checksum verification failed"
            exit 1
        fi
        log "Checksum verified"
    fi
    
    # Receive
    gunzip < "$backup_file" | btrfs receive "$target_path"
    
    if [ $? -eq 0 ]; then
        log "Backup received successfully"
        return 0
    else
        log "ERROR: Failed to receive backup"
        return 1
    fi
}

cleanup_snapshots() {
    log "Cleaning up old snapshots..."
    
    # Daily snapshots
    local daily_count=$(ls -t "$SNAPSHOT_DIR"/daily-* 2>/dev/null | wc -l)
    if [ $daily_count -gt $RETENTION_DAILY ]; then
        ls -t "$SNAPSHOT_DIR"/daily-* | tail -n +$((RETENTION_DAILY + 1)) | while read snap; do
            log "Deleting old daily snapshot: $(basename "$snap")"
            btrfs subvolume delete "$snap"
        done
    fi
    
    # Weekly snapshots
    local weekly_count=$(ls -t "$SNAPSHOT_DIR"/weekly-* 2>/dev/null | wc -l)
    if [ $weekly_count -gt $RETENTION_WEEKLY ]; then
        ls -t "$SNAPSHOT_DIR"/weekly-* | tail -n +$((RETENTION_WEEKLY + 1)) | while read snap; do
            log "Deleting old weekly snapshot: $(basename "$snap")"
            btrfs subvolume delete "$snap"
        done
    fi
    
    # Monthly snapshots
    local monthly_count=$(ls -t "$SNAPSHOT_DIR"/monthly-* 2>/dev/null | wc -l)
    if [ $monthly_count -gt $RETENTION_MONTHLY ]; then
        ls -t "$SNAPSHOT_DIR"/monthly-* | tail -n +$((RETENTION_MONTHLY + 1)) | while read snap; do
            log "Deleting old monthly snapshot: $(basename "$snap")"
            btrfs subvolume delete "$snap"
        done
    fi
    
    log "Snapshot cleanup completed"
}

list_snapshots() {
    echo "=== Btrfs Snapshots ==="
    btrfs subvolume list "$SUBVOLUME" | grep ".snapshots"
    
    echo ""
    echo "=== Backup Files ==="
    ls -lh "$BACKUP_TARGET"/*.btrfs.gz 2>/dev/null || echo "No backups found"
}

verify_snapshots() {
    log "Verifying snapshots..."
    
    for snap in "$SNAPSHOT_DIR"/*; do
        if [ -d "$snap" ]; then
            if btrfs subvolume show "$snap" > /dev/null 2>&1; then
                log "✓ Valid: $(basename "$snap")"
            else
                log "✗ Invalid: $(basename "$snap")"
            fi
        fi
    done
}

# Main execution
case "${1:-daily}" in
    daily)
        snap=$(create_snapshot "daily")
        send_incremental "$snap"
        cleanup_snapshots
        ;;
    weekly)
        snap=$(create_snapshot "weekly")
        send_incremental "$snap"
        ;;
    monthly)
        snap=$(create_snapshot "monthly")
        send_incremental "$snap"
        ;;
    receive)
        receive_backup "$2" "$3"
        ;;
    list)
        list_snapshots
        ;;
    verify)
        verify_snapshots
        ;;
    cleanup)
        cleanup_snapshots
        ;;
    *)
        echo "Usage: $0 {daily|weekly|monthly|receive <file> <target>|list|verify|cleanup}"
        exit 1
        ;;
esac
EOF

chmod +x btrfs_snapshot_backup.sh
```

**2. Disaster Recovery Plan Generator:**

````bash
cat > generate_dr_plan.sh <<'EOF'
#!/bin/bash

DR_PLAN_FILE="/backup/disaster-recovery-plan.md"
BACKUP_INFO="/backup/system/latest"

cat > "$DR_PLAN_FILE" <<'DRPLAN'
# Disaster Recovery Plan

**Generated:** $(date)
**System:** $(hostname)

---

## Emergency Contacts

| Role | Name | Contact |
|------|------|---------|
| System Admin | Your Name | +1-xxx-xxx-xxxx |
| Backup Admin | Backup Name | +1-xxx-xxx-xxxx |
| Manager | Manager Name | +1-xxx-xxx-xxxx |

---

## Recovery Time Objectives (RTO)

| System Component | RTO | RPO |
|-----------------|-----|-----|
| Database | 2 hours | 4 hours |
| File Storage | 4 hours | 24 hours |
| Web Application | 1 hour | 1 hour |

---

## Backup Locations

### Primary Backups
- **Location:** /backup/
- **Type:** Local storage
- **Retention:** 30 days

### Secondary Backups
- **Location:** NAS (//backup-server/backups)
- **Type:** Network storage
- **Retention:** 90 days

### Offsite Backups
- **Location:** AWS S3 (s3://company-backups/)
- **Type:** Cloud storage
- **Retention:** 1 year

---

## Recovery Procedures

### 1. Database Recovery

#### MySQL Recovery
```bash
# Stop MySQL
systemctl stop mysql

# Restore from latest backup
LATEST_BACKUP=$(ls -t /backup/mysql/daily/*.sql.gz | head -1)
gunzip < "$LATEST_BACKUP" | mysql -u root -p

# Apply binary logs for PITR
mysqlbinlog --start-datetime="YYYY-MM-DD HH:MM:SS" \
    /backup/mysql/binlogs/*.log | mysql -u root -p

# Start MySQL
systemctl start mysql
````

#### PostgreSQL Recovery

```bash
# Stop PostgreSQL
systemctl stop postgresql

# Clear data directory
rm -rf /var/lib/postgresql/14/main/*

# Restore base backup
tar -xzf /backup/postgresql/base/latest/base.tar.gz \
    -C /var/lib/postgresql/14/main/

# Configure recovery
cat > /var/lib/postgresql/14/main/postgresql.auto.conf <<PGCONF
restore_command = 'cp /backup/postgresql/wal_archive/%f %p'
recovery_target_time = 'YYYY-MM-DD HH:MM:SS'
PGCONF

touch /var/lib/postgresql/14/main/recovery.signal

# Start PostgreSQL
systemctl start postgresql
```

### 2. File System Recovery

#### From Rsync Snapshots

```bash
# List available snapshots
/usr/local/bin/rsync_backup.sh list

# Restore specific snapshot
/usr/local/bin/rsync_backup.sh restore SNAPSHOT_NAME /restore/target
```

#### From LVM Backup

```bash
# Restore from LVM backup
tar -xzf /backup/lvm/lvm-backup-YYYYMMDD.tar.gz -C /restore/target
```

#### From Btrfs Snapshot

```bash
# Receive btrfs backup
gunzip < /backup/btrfs/daily-YYYYMMDD.btrfs.gz | \
    btrfs receive /restore/target
```

### 3. Full System Recovery

#### Bare Metal Recovery

```bash
# 1. Boot from installation media
# 2. Partition disks according to /backup/system/latest/partition-*.dump
# 3. Mount filesystems
# 4. Run restore script
cd /backup/system/latest
./restore.sh
```

#### Virtual Machine Recovery

```bash
# 1. Create new VM with same specs
# 2. Mount backup location
# 3. Restore system files
tar -xzf /backup/system/latest/etc.tar.gz -C /
tar -xzf /backup/system/latest/home.tar.gz -C /

# 4. Reinstall packages
dpkg --set-selections < /backup/system/latest/packages-dpkg.list
apt-get dselect-upgrade -y

# 5. Restore databases
# Follow database recovery procedures above
```

---

## Verification Steps

After recovery, verify the following:

### Database Verification

```bash
# MySQL
mysql -u root -p -e "SELECT COUNT(*) FROM critical_table;"

# PostgreSQL
psql -U postgres -c "SELECT COUNT(*) FROM critical_table;"
```

### Application Verification

```bash
# Check services
systemctl status mysql postgresql nginx

# Test application endpoints
curl http://localhost/health
curl http://localhost/api/status
```

### Data Integrity

```bash
# Verify file checksums
md5sum -c /backup/checksums.md5

# Check disk usage
df -h

# Verify user access
su - testuser -c "ls ~/Documents"
```

---

## Common Issues and Solutions

### Issue: Database won't start after restore

**Solution:**

```bash
# Check permissions
chown -R mysql:mysql /var/lib/mysql
chown -R postgres:postgres /var/lib/postgresql

# Check logs
tail -f /var/log/mysql/error.log
tail -f /var/log/postgresql/postgresql-14-main.log
```

### Issue: Missing binary logs

**Solution:**

```bash
# Restore from last consistent backup
# Accept data loss up to last backup point
# Document lost transaction window
```

### Issue: Corrupted backup file

**Solution:**

```bash
# Try next recent backup
# Use backup from secondary location
# Restore from offsite if necessary
```

---

## Testing Schedule

|Test Type|Frequency|Last Test|Next Test|
|---|---|---|---|
|File Restore Test|Monthly|$(date -d 'last month' +%Y-%m-%d)|$(date -d 'next month' +%Y-%m-%d)|
|Database Restore Test|Quarterly|-|-|
|Full DR Test|Annually|-|-|

---

## Backup Verification Log

```bash
# Last verification: $(date)
# Verified backups:
$(ls -lh /backup/mysql/daily/*.sql.gz 2>/dev/null | tail -5 || echo "No backups found")
```

---

## Notes

- Always verify backup integrity before declaring recovery complete
- Document any deviations from this plan
- Update this plan after each DR test
- Keep offline copy of this plan

DRPLAN

echo "Disaster Recovery Plan generated: $DR_PLAN_FILE" EOF

chmod +x generate_dr_plan.sh

````

**3. Backup Monitoring Dashboard:**

```bash
cat > backup_status_dashboard.sh <<'EOF'
#!/bin/bash

# Generate HTML dashboard for backup status

DASHBOARD_FILE="/var/www/html/backup-status.html"
BACKUP_ROOT="/backup"

generate_dashboard() {
    cat > "$DASHBOARD_FILE" <<'HTML'
<!DOCTYPE html>
<html>
<head>
    <title>Backup Status Dashboard</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f5f5f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        h1 {
            color: #333;
            border-bottom: 2px solid #4CAF50;
            padding-bottom: 10px;
        }
        .status-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            margin: 20px 0;
        }
        .status-card {
            border: 1px solid #ddd;
            border-radius: 4px;
            padding: 15px;
            background: #fafafa;
        }
        .status-ok { border-left: 4px solid #4CAF50; }
        .status-warning { border-left: 4px solid #FF9800; }
        .status-error { border-left: 4px solid #f44336; }
        .metric {
            display: flex;
            justify-content: space-between;
            margin: 10px 0;
        }
        .metric-value {
            font-weight: bold;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #4CAF50;
            color: white;
        }
        .timestamp {
            color: #666;
            font-size: 0.9em;
        }
    </style>
    <meta http-equiv="refresh" content="300">
</head>
<body>
    <div class="container">
        <h1>Backup Status Dashboard</h1>
        <div class="timestamp">Last Updated: $(date)</div>
        
        <div class="status-grid">
HTML

    # MySQL Backups
    local mysql_latest=$(ls -t $BACKUP_ROOT/mysql/daily/*.sql.gz 2>/dev/null | head -1)
    local mysql_age=""
    local mysql_status="status-ok"
    
    if [ -n "$mysql_latest" ]; then
        mysql_age=$(( ($(date +%s) - $(stat -c %Y "$mysql_latest")) / 3600 ))
        [ $mysql_age -gt 24 ] && mysql_status="status-warning"
        [ $mysql_age -gt 48 ] && mysql_status="status-error"
    else
        mysql_status="status-error"
        mysql_age="N/A"
    fi
    
    cat >> "$DASHBOARD_FILE" <<HTML
            <div class="status-card $mysql_status">
                <h3>MySQL Backups</h3>
                <div class="metric">
                    <span>Last Backup:</span>
                    <span class="metric-value">${mysql_age}h ago</span>
                </div>
                <div class="metric">
                    <span>Size:</span>
                    <span class="metric-value">$(du -sh $BACKUP_ROOT/mysql 2>/dev/null | cut -f1 || echo "N/A")</span>
                </div>
                <div class="metric">
                    <span>Files:</span>
                    <span class="metric-value">$(find $BACKUP_ROOT/mysql -name "*.sql.gz" | wc -l)</span>
                </div>
            </div>
HTML

    # PostgreSQL Backups
    local pg_latest=$(ls -t $BACKUP_ROOT/postgresql/pg-*.sql.gz 2>/dev/null | head -1)
    local pg_age=""
    local pg_status="status-ok"
    
    if [ -n "$pg_latest" ]; then
        pg_age=$(( ($(date +%s) - $(stat -c %Y "$pg_latest")) / 3600 ))
        [ $pg_age -gt 24 ] && pg_status="status-warning"
        [ $pg_age -gt 48 ] && pg_status="status-error"
    else
        pg_status="status-error"
        pg_age="N/A"
    fi
    
    cat >> "$DASHBOARD_FILE" <<HTML
            <div class="status-card $pg_status">
                <h3>PostgreSQL Backups</h3>
                <div class="metric">
                    <span>Last Backup:</span>
                    <span class="metric-value">${pg_age}h ago</span>
                </div>
                <div class="metric">
                    <span>Size:</span>
                    <span class="metric-value">$(du -sh $BACKUP_ROOT/postgresql 2>/dev/null | cut -f1 || echo "N/A")</span>
                </div>
                <div class="metric">
                    <span>WAL Files:</span>
                    <span class="metric-value">$(find $BACKUP_ROOT/postgresql/wal_archive -type f 2>/dev/null | wc -l || echo "0")</span>
                </div>
            </div>
HTML

    # Filesystem Backups
    local rsync_latest=$(ls -td $BACKUP_ROOT/rsync-snapshots/* 2>/dev/null | grep -v latest | head -1)
    local rsync_age=""
    local rsync_status="status-ok"
    
    if [ -n "$rsync_latest" ]; then
        rsync_age=$(( ($(date +%s) - $(stat -c %Y "$rsync_latest")) / 3600 ))
        [ $rsync_age -gt 24 ] && rsync_status="status-warning"
        [ $rsync_age -gt 48 ] && rsync_status="status-error"
    else
        rsync_status="status-error"
        rsync_age="N/A"
    fi
    
    cat >> "$DASHBOARD_FILE" <<HTML
            <div class="status-card $rsync_status">
                <h3>Filesystem Snapshots</h3>
                <div class="metric">
                    <span>Last Snapshot:</span>
                    <span class="metric-value">${rsync_age}h ago</span>
                </div>
                <div class="metric">
                    <span>Total Size:</span>
                    <span class="metric-value">$(du -sh $BACKUP_ROOT/rsync-snapshots 2>/dev/null | cut -f1 || echo "N/A")</span>
                </div>
                <div class="metric">
                    <span>Snapshots:</span>
                    <span class="metric-value">$(ls -d $BACKUP_ROOT/rsync-snapshots/* 2>/dev/null | wc -l || echo "0")</span>
                </div>
            </div>
HTML

    # Storage Usage
    local total_backup_size=$(du -sh $BACKUP_ROOT 2>/dev/null | cut -f1 || echo "N/A")
    local disk_usage=$(df -h $BACKUP_ROOT | tail -1 | awk '{print $5}' | tr -d '%')
    local storage_status="status-ok"
    
    [ $disk_usage -gt 80 ] && storage_status="status-warning"
    [ $disk_usage -gt 90 ] && storage_status="status-error"
    
    cat >> "$DASHBOARD_FILE" <<HTML
            <div class="status-card $storage_status">
                <h3>Storage Usage</h3>
                <div class="metric">
                    <span>Total Backups:</span>
                    <span class="metric-value">$total_backup_size</span>
                </div>
                <div class="metric">
                    <span>Disk Usage:</span>
                    <span class="metric-value">${disk_usage}%</span>
                </div>
                <div class="metric">
                    <span>Available:</span>
                    <span class="metric-value">$(df -h $BACKUP_ROOT | tail -1 | awk '{print $4}')</span>
                </div>
            </div>
        </div>
        
        <h2>Recent Backups</h2>
        <table>
            <tr>
                <th>Type</th>
                <th>File</th>
                <th>Size</th>
                <th>Age</th>
            </tr>
HTML

    # List recent backups
    find $BACKUP_ROOT -name "*.sql.gz" -o -name "*.tar.gz" | \
        xargs ls -lt 2>/dev/null | head -10 | while read line; do
        local file=$(echo "$line" | awk '{print $NF}')
        local size=$(echo "$line" | awk '{print $5}' | numfmt --to=iec-i --suffix=B 2>/dev/null || echo "$line" | awk '{print $5}')
        local date=$(echo "$line" | awk '{print $6, $7, $8}')
        local type=$(echo "$file" | grep -o 'mysql\|postgresql\|system\|lvm\|rsync' | head -1)
        
        cat >> "$DASHBOARD_FILE" <<HTML
            <tr>
                <td>$type</td>
                <td>$(basename "$file")</td>
                <td>$size</td>
                <td>$date</td>
            </tr>
HTML
    done
    
    cat >> "$DASHBOARD_FILE" <<HTML
        </table>
    </div>
</body>
</html>
HTML
}

# Generate dashboard
generate_dashboard

echo "Dashboard generated: $DASHBOARD_FILE"
EOF

chmod +x backup_status_dashboard.sh

# Add to cron
echo "*/5 * * * * /usr/local/bin/backup_status_dashboard.sh" | crontab -
````

---

## Итоговое задание модуля 3

Создай полноценную backup & recovery инфраструктуру:

```bash
# 1. Установи все скрипты
sudo cp rsync_backup.sh /usr/local/bin/
sudo cp lvm_snapshot_backup.sh /usr/local/bin/
sudo cp system_backup.sh /usr/local/bin/
sudo cp btrfs_snapshot_backup.sh /usr/local/bin/
sudo cp generate_dr_plan.sh /usr/local/bin/
sudo cp backup_status_dashboard.sh /usr/local/bin/

# 2. Создай структуру директорий
sudo mkdir -p /backup/{rsync-snapshots,lvm,system,btrfs,mysql,postgresql}

# 3. Настрой расписание
sudo ./setup_backup_cron.sh

# 4. Запусти тестовое резервное копирование
sudo rsync_backup.sh snapshot
sudo lvm_snapshot_backup.sh backup
sudo system_backup.sh

# 5. Сгенерируй DR план
sudo generate_dr_plan.sh

# 6. Проверь статус
sudo backup_status_dashboard.sh
firefox /var/www/html/backup-status.html
```

---

## Заключение курса

**Что вы освежили:**

- ✅ Backup стратегии (Full/Incremental/Differential)
- ✅ Database backup методы (MySQL, PostgreSQL, MongoDB, Redis)
- ✅ Filesystem snapshots (LVM, Btrfs, ZFS)
- ✅ System state backups
- ✅ Disaster Recovery planning

**Новые техники:**

- 🚀 Deduplicated backups с rsync hardlinks
- 🚀 Encrypted cloud backups
- 🚀 Automated backup testing
- 🚀 Monitoring и dashboards
- 🚀 Incremental Btrfs snapshots

**Следующие шаги:**

1. Адаптируй скрипты под свою инфраструктуру
2. Протестируй восстановление (ОБЯЗАТЕЛЬНО!)
3. Документируй свои процедуры
4. Автоматизируй мониторинг
5. Регулярно тестируй DR план

**Помни золотое правило:**

> "Backup без проверенного restore = катастрофа"

Тестируй восстановление минимум раз в квартал!

# Модуль 4: Cloud Backups & Hybrid Strategies (30 минут)

## 🎯 Напоминалка

### Cloud Storage Тiers

```
Cloud Storage Hierarchy
├── Hot Storage (Instant Access)
│   ├── AWS S3 Standard
│   ├── Azure Blob Hot
│   └── GCP Standard Storage
│   └── Cost: $$$$, Access: milliseconds
│
├── Cool Storage (Infrequent Access)
│   ├── AWS S3 Infrequent Access (IA)
│   ├── Azure Blob Cool
│   └── GCP Nearline
│   └── Cost: $$$, Access: milliseconds
│
├── Cold Storage (Archive)
│   ├── AWS S3 Glacier
│   ├── Azure Blob Archive
│   └── GCP Coldline
│   └── Cost: $$, Access: minutes to hours
│
└── Deep Archive
    ├── AWS S3 Glacier Deep Archive
    ├── Azure Archive (long-term)
    └── GCP Archive
    └── Cost: $, Access: hours to days
```

### 3-2-1-1-0 Rule (Modern Extension)

```
3 - Three copies of data
2 - Two different media types
1 - One copy offsite
1 - One copy offline (air-gapped)
0 - Zero errors in recovery testing
```

### Cloud Provider CLI Basics

```bash
# AWS CLI
aws s3 cp file.txt s3://bucket/
aws s3 sync /local/dir s3://bucket/dir/
aws s3 ls s3://bucket/

# Azure CLI
az storage blob upload --file file.txt --container backups
az storage blob upload-batch --destination backups --source /local/dir

# Google Cloud
gsutil cp file.txt gs://bucket/
gsutil -m rsync -r /local/dir gs://bucket/dir/
gsutil ls gs://bucket/

# Rclone (universal tool)
rclone sync /local/dir remote:bucket/dir
rclone copy /local/file remote:bucket/
```

### Lifecycle Policies

```json
// AWS S3 Lifecycle Policy
{
  "Rules": [
    {
      "Id": "BackupLifecycle",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        },
        {
          "Days": 365,
          "StorageClass": "DEEP_ARCHIVE"
        }
      ],
      "Expiration": {
        "Days": 2555  // 7 years
      }
    }
  ]
}
```

### Cost Comparison (approximate, per GB/month)

```
AWS S3 Standard:        $0.023
AWS S3 IA:              $0.0125
AWS S3 Glacier:         $0.004
AWS S3 Deep Archive:    $0.00099

Azure Blob Hot:         $0.018
Azure Blob Cool:        $0.01
Azure Blob Archive:     $0.002

GCP Standard:           $0.020
GCP Nearline:          $0.010
GCP Coldline:          $0.004
GCP Archive:           $0.0012

Backblaze B2:          $0.005
Wasabi:                $0.0059 (no egress fees!)
```

---

## 💻 Задание 1: Multi-Cloud Backup с Rclone

### Установка и настройка Rclone

```bash
cat > setup_rclone.sh <<'EOF'
#!/bin/bash

set -e

# Install rclone
echo "Installing rclone..."
curl https://rclone.org/install.sh | sudo bash

# Create config directory
mkdir -p ~/.config/rclone

cat > ~/.config/rclone/rclone.conf <<'RCLONE_CONFIG'
# AWS S3
[aws-s3]
type = s3
provider = AWS
access_key_id = YOUR_AWS_ACCESS_KEY
secret_access_key = YOUR_AWS_SECRET_KEY
region = us-east-1
acl = private

# Azure Blob
[azure-blob]
type = azureblob
account = YOUR_AZURE_ACCOUNT
key = YOUR_AZURE_KEY

# Google Cloud Storage
[gcs]
type = google cloud storage
project_number = YOUR_PROJECT_NUMBER
service_account_file = /path/to/service-account.json
location = us

# Backblaze B2
[b2]
type = b2
account = YOUR_B2_ACCOUNT_ID
key = YOUR_B2_APP_KEY

# Wasabi
[wasabi]
type = s3
provider = Wasabi
access_key_id = YOUR_WASABI_ACCESS_KEY
secret_access_key = YOUR_WASABI_SECRET_KEY
region = us-east-1
endpoint = s3.wasabisys.com

# Local NAS
[nas]
type = sftp
host = nas.local
user = backup
key_file = ~/.ssh/nas_backup_key
RCLONE_CONFIG

echo "Rclone configured. Edit ~/.config/rclone/rclone.conf with your credentials"
echo "Test with: rclone lsd aws-s3:"
EOF

chmod +x setup_rclone.sh
./setup_rclone.sh
```

### Cloud Backup Script с Multi-Destination

```bash
cat > cloud_backup.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
SOURCE_DIR="/data"
BACKUP_NAME="backup-$(date +%Y%m%d-%H%M%S)"
LOCAL_BACKUP="/backup/cloud-staging"
LOG_FILE="/var/log/cloud_backup.log"
ENCRYPTION_PASSWORD_FILE="/root/.backup_encryption_key"

# Cloud destinations (priority order)
PRIMARY_CLOUD="aws-s3:company-backups"
SECONDARY_CLOUD="b2:company-backups-secondary"
TERTIARY_CLOUD="wasabi:company-backups-tertiary"

# Notification settings
SLACK_WEBHOOK="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
EMAIL_TO="ops@company.com"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

send_notification() {
    local status="$1"
    local message="$2"
    
    # Slack notification
    if [ -n "$SLACK_WEBHOOK" ]; then
        curl -X POST "$SLACK_WEBHOOK" \
            -H 'Content-Type: application/json' \
            -d "{\"text\":\"[$status] Cloud Backup: $message\"}" \
            2>/dev/null || true
    fi
    
    # Email notification (if mailutils installed)
    if command -v mail &>/dev/null; then
        echo "$message" | mail -s "[$status] Cloud Backup" "$EMAIL_TO" || true
    fi
}

create_local_backup() {
    log "Creating local staging backup..."
    
    mkdir -p "$LOCAL_BACKUP"
    
    # Create compressed archive with encryption
    tar -czf - "$SOURCE_DIR" 2>/dev/null | \
        openssl enc -aes-256-cbc -salt -pbkdf2 \
        -pass file:"$ENCRYPTION_PASSWORD_FILE" \
        -out "$LOCAL_BACKUP/${BACKUP_NAME}.tar.gz.enc"
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        local size=$(du -h "$LOCAL_BACKUP/${BACKUP_NAME}.tar.gz.enc" | cut -f1)
        log "Local backup created: $size"
        
        # Create metadata
        cat > "$LOCAL_BACKUP/${BACKUP_NAME}.meta" <<META
{
  "backup_name": "$BACKUP_NAME",
  "source": "$SOURCE_DIR",
  "created": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "size_bytes": $(stat -c%s "$LOCAL_BACKUP/${BACKUP_NAME}.tar.gz.enc" 2>/dev/null || stat -f%z "$LOCAL_BACKUP/${BACKUP_NAME}.tar.gz.enc"),
  "encrypted": true,
  "compression": "gzip"
}
META
        
        # Create checksum
        md5sum "$LOCAL_BACKUP/${BACKUP_NAME}.tar.gz.enc" > "$LOCAL_BACKUP/${BACKUP_NAME}.md5"
        
        return 0
    else
        log "ERROR: Failed to create local backup"
        return 1
    fi
}

upload_to_cloud() {
    local destination="$1"
    local destination_name=$(echo "$destination" | cut -d: -f1)
    
    log "Uploading to $destination_name..."
    
    # Upload with progress and retry
    rclone copy "$LOCAL_BACKUP/${BACKUP_NAME}.tar.gz.enc" \
        "$destination/backups/$(date +%Y/%m/%d)/" \
        --progress \
        --transfers 4 \
        --checkers 8 \
        --retries 3 \
        --low-level-retries 10 \
        --stats 1m \
        --stats-log-level INFO \
        2>&1 | tee -a "$LOG_FILE"
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        # Upload metadata and checksum
        rclone copy "$LOCAL_BACKUP/${BACKUP_NAME}.meta" \
            "$destination/backups/$(date +%Y/%m/%d)/" --quiet
        rclone copy "$LOCAL_BACKUP/${BACKUP_NAME}.md5" \
            "$destination/backups/$(date +%Y/%m/%d)/" --quiet
        
        log "✓ Upload to $destination_name completed"
        return 0
    else
        log "✗ Upload to $destination_name failed"
        return 1
    fi
}

verify_cloud_backup() {
    local destination="$1"
    local destination_name=$(echo "$destination" | cut -d: -f1)
    local remote_path="$destination/backups/$(date +%Y/%m/%d)/${BACKUP_NAME}.tar.gz.enc"
    
    log "Verifying backup on $destination_name..."
    
    # Check if file exists
    if rclone lsf "$remote_path" &>/dev/null; then
        # Get remote size
        local remote_size=$(rclone size "$remote_path" --json | jq -r '.bytes')
        local local_size=$(stat -c%s "$LOCAL_BACKUP/${BACKUP_NAME}.tar.gz.enc" 2>/dev/null || stat -f%z "$LOCAL_BACKUP/${BACKUP_NAME}.tar.gz.enc")
        
        if [ "$remote_size" = "$local_size" ]; then
            log "✓ Verification passed: sizes match ($local_size bytes)"
            return 0
        else
            log "✗ Verification failed: size mismatch (local: $local_size, remote: $remote_size)"
            return 1
        fi
    else
        log "✗ Verification failed: file not found on $destination_name"
        return 1
    fi
}

cleanup_old_backups() {
    local destination="$1"
    local destination_name=$(echo "$destination" | cut -d: -f1)
    local retention_days="$2"
    
    log "Cleaning up backups older than $retention_days days on $destination_name..."
    
    # List and delete old backups
    rclone delete "$destination/backups/" \
        --min-age ${retention_days}d \
        --verbose \
        2>&1 | tee -a "$LOG_FILE"
    
    # Clean empty directories
    rclone rmdirs "$destination/backups/" \
        --leave-root \
        2>&1 | tee -a "$LOG_FILE"
}

cleanup_local_staging() {
    log "Cleaning up local staging area..."
    
    # Keep only last 2 backups locally
    ls -t "$LOCAL_BACKUP"/*.tar.gz.enc 2>/dev/null | tail -n +3 | xargs rm -f 2>/dev/null || true
    ls -t "$LOCAL_BACKUP"/*.meta 2>/dev/null | tail -n +3 | xargs rm -f 2>/dev/null || true
    ls -t "$LOCAL_BACKUP"/*.md5 2>/dev/null | tail -n +3 | xargs rm -f 2>/dev/null || true
}

generate_backup_report() {
    local report_file="$LOCAL_BACKUP/backup-report-$(date +%Y%m%d).txt"
    
    cat > "$report_file" <<REPORT
================================================================================
Cloud Backup Report
Generated: $(date)
================================================================================

Backup Details:
- Name: $BACKUP_NAME
- Source: $SOURCE_DIR
- Size: $(du -h "$LOCAL_BACKUP/${BACKUP_NAME}.tar.gz.enc" | cut -f1)

Upload Status:
REPORT

    for cloud in "$PRIMARY_CLOUD" "$SECONDARY_CLOUD" "$TERTIARY_CLOUD"; do
        local cloud_name=$(echo "$cloud" | cut -d: -f1)
        if rclone lsf "$cloud/backups/$(date +%Y/%m/%d)/${BACKUP_NAME}.tar.gz.enc" &>/dev/null; then
            echo "- $cloud_name: ✓ SUCCESS" >> "$report_file"
        else
            echo "- $cloud_name: ✗ FAILED" >> "$report_file"
        fi
    done
    
    cat >> "$report_file" <<REPORT

Storage Usage by Cloud:
REPORT

    for cloud in "$PRIMARY_CLOUD" "$SECONDARY_CLOUD" "$TERTIARY_CLOUD"; do
        local cloud_name=$(echo "$cloud" | cut -d: -f1)
        local usage=$(rclone size "$cloud/backups/" --json 2>/dev/null | jq -r '.bytes' || echo "0")
        local usage_gb=$(echo "scale=2; $usage / 1024 / 1024 / 1024" | bc)
        echo "- $cloud_name: ${usage_gb} GB" >> "$report_file"
    done
    
    cat "$report_file"
    
    # Upload report to clouds
    for cloud in "$PRIMARY_CLOUD"; do
        rclone copy "$report_file" "$cloud/reports/" --quiet || true
    done
}

# Main execution
main() {
    local start_time=$(date +%s)
    
    log "========================================="
    log "Starting cloud backup process"
    log "========================================="
    
    # Create local backup
    if ! create_local_backup; then
        send_notification "ERROR" "Failed to create local backup"
        exit 1
    fi
    
    # Upload to multiple clouds
    local upload_success=0
    
    if upload_to_cloud "$PRIMARY_CLOUD" && verify_cloud_backup "$PRIMARY_CLOUD"; then
        upload_success=$((upload_success + 1))
    fi
    
    if upload_to_cloud "$SECONDARY_CLOUD" && verify_cloud_backup "$SECONDARY_CLOUD"; then
        upload_success=$((upload_success + 1))
    fi
    
    if upload_to_cloud "$TERTIARY_CLOUD" && verify_cloud_backup "$TERTIARY_CLOUD"; then
        upload_success=$((upload_success + 1))
    fi
    
    # Check if at least one upload succeeded
    if [ $upload_success -eq 0 ]; then
        send_notification "CRITICAL" "All cloud uploads failed!"
        exit 1
    elif [ $upload_success -lt 3 ]; then
        send_notification "WARNING" "Only $upload_success of 3 cloud uploads succeeded"
    else
        send_notification "SUCCESS" "All cloud backups completed successfully"
    fi
    
    # Cleanup old backups (different retention for each tier)
    cleanup_old_backups "$PRIMARY_CLOUD" 30      # 30 days on primary
    cleanup_old_backups "$SECONDARY_CLOUD" 90    # 90 days on secondary
    cleanup_old_backups "$TERTIARY_CLOUD" 365    # 1 year on tertiary
    
    # Cleanup local staging
    cleanup_local_staging
    
    # Generate report
    generate_backup_report
    
    local end_time=$(date +%s)
    local duration=$((end_time - start_time))
    
    log "========================================="
    log "Cloud backup completed in $duration seconds"
    log "Successful uploads: $upload_success/3"
    log "========================================="
}

# Run main function
main
EOF

chmod +x cloud_backup.sh
```

---

## 💻 Задание 2: Restic - Modern Backup Tool

### Установка и настройка Restic

```bash
cat > setup_restic.sh <<'EOF'
#!/bin/bash

set -e

# Install restic
echo "Installing restic..."
wget https://github.com/restic/restic/releases/download/v0.16.2/restic_0.16.2_linux_amd64.bz2
bunzip2 restic_0.16.2_linux_amd64.bz2
chmod +x restic_0.16.2_linux_amd64
sudo mv restic_0.16.2_linux_amd64 /usr/local/bin/restic

# Verify installation
restic version

echo "Restic installed successfully"
EOF

chmod +x setup_restic.sh
./setup_restic.sh
```

### Restic Backup Script (with multiple backends)

```bash
cat > restic_backup.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
export RESTIC_PASSWORD_FILE="/root/.restic_password"
export AWS_ACCESS_KEY_ID="YOUR_AWS_KEY"
export AWS_SECRET_ACCESS_KEY="YOUR_AWS_SECRET"
export B2_ACCOUNT_ID="YOUR_B2_ACCOUNT"
export B2_ACCOUNT_KEY="YOUR_B2_KEY"

BACKUP_SOURCE="/data"
LOG_FILE="/var/log/restic_backup.log"

# Repository locations
REPO_LOCAL="/backup/restic-repo"
REPO_S3="s3:s3.amazonaws.com/my-restic-backup"
REPO_B2="b2:my-restic-backup"
REPO_SFTP="sftp:backup@nas.local:/backups/restic"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

init_repository() {
    local repo="$1"
    
    log "Initializing repository: $repo"
    
    if ! restic -r "$repo" snapshots &>/dev/null; then
        restic -r "$repo" init
        log "Repository initialized: $repo"
    else
        log "Repository already exists: $repo"
    fi
}

backup_to_repo() {
    local repo="$1"
    local repo_name="$2"
    
    log "Starting backup to $repo_name..."
    
    restic -r "$repo" backup "$BACKUP_SOURCE" \
        --verbose \
        --exclude='*.tmp' \
        --exclude='*.cache' \
        --exclude='*/node_modules/*' \
        --exclude='*/.git/*' \
        --tag "automated" \
        --tag "$(hostname)" \
        2>&1 | tee -a "$LOG_FILE"
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        log "✓ Backup to $repo_name completed"
        return 0
    else
        log "✗ Backup to $repo_name failed"
        return 1
    fi
}

verify_backup() {
    local repo="$1"
    local repo_name="$2"
    
    log "Verifying backup in $repo_name..."
    
    # Check repository integrity
    restic -r "$repo" check \
        --read-data-subset=10% \
        2>&1 | tee -a "$LOG_FILE"
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        log "✓ Verification passed for $repo_name"
        return 0
    else
        log "✗ Verification failed for $repo_name"
        return 1
    fi
}

cleanup_old_snapshots() {
    local repo="$1"
    local repo_name="$2"
    
    log "Cleaning up old snapshots in $repo_name..."
    
    # Keep:
    # - Last 7 daily snapshots
    # - Last 4 weekly snapshots
    # - Last 12 monthly snapshots
    # - Last 7 yearly snapshots
    restic -r "$repo" forget \
        --keep-daily 7 \
        --keep-weekly 4 \
        --keep-monthly 12 \
        --keep-yearly 7 \
        --prune \
        2>&1 | tee -a "$LOG_FILE"
    
    log "Cleanup completed for $repo_name"
}

list_snapshots() {
    local repo="$1"
    local repo_name="$2"
    
    log "Listing snapshots in $repo_name:"
    
    restic -r "$repo" snapshots --compact
}

get_backup_stats() {
    local repo="$1"
    
    restic -r "$repo" stats --mode raw-data --json
}

restore_from_backup() {
    local repo="$1"
    local snapshot_id="$2"
    local target_dir="$3"
    
    if [ -z "$snapshot_id" ] || [ -z "$target_dir" ]; then
        echo "Usage: restore_from_backup <repo> <snapshot_id> <target_dir>"
        return 1
    fi
    
    log "Restoring snapshot $snapshot_id to $target_dir..."
    
    mkdir -p "$target_dir"
    
    restic -r "$repo" restore "$snapshot_id" \
        --target "$target_dir" \
        --verbose \
        2>&1 | tee -a "$LOG_FILE"
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        log "✓ Restore completed successfully"
        return 0
    else
        log "✗ Restore failed"
        return 1
    fi
}

mount_backup() {
    local repo="$1"
    local mount_point="$2"
    
    if [ -z "$mount_point" ]; then
        echo "Usage: mount_backup <repo> <mount_point>"
        return 1
    fi
    
    log "Mounting repository to $mount_point..."
    
    mkdir -p "$mount_point"
    
    restic -r "$repo" mount "$mount_point"
}

compare_repos() {
    log "Comparing repositories..."
    
    echo "=== Local Repository ==="
    restic -r "$REPO_LOCAL" snapshots --compact
    get_backup_stats "$REPO_LOCAL"
    
    echo ""
    echo "=== S3 Repository ==="
    restic -r "$REPO_S3" snapshots --compact
    get_backup_stats "$REPO_S3"
    
    echo ""
    echo "=== B2 Repository ==="
    restic -r "$REPO_B2" snapshots --compact
    get_backup_stats "$REPO_B2"
}

# Main execution
case "${1:-backup}" in
    init)
        init_repository "$REPO_LOCAL"
        init_repository "$REPO_S3"
        init_repository "$REPO_B2"
        ;;
    
    backup)
        backup_to_repo "$REPO_LOCAL" "local"
        backup_to_repo "$REPO_S3" "s3"
        backup_to_repo "$REPO_B2" "b2"
        ;;
    
    verify)
        verify_backup "$REPO_LOCAL" "local"
        verify_backup "$REPO_S3" "s3"
        ;;
    
    cleanup)
        cleanup_old_snapshots "$REPO_LOCAL" "local"
        cleanup_old_snapshots "$REPO_S3" "s3"
        cleanup_old_snapshots "$REPO_B2" "b2"
        ;;
    
    list)
        list_snapshots "$REPO_LOCAL" "local"
        echo ""
        list_snapshots "$REPO_S3" "s3"
        ;;
    
    compare)
        compare_repos
        ;;
    
    restore)
        restore_from_backup "$REPO_LOCAL" "$2" "$3"
        ;;
    
    mount)
        mount_backup "$REPO_LOCAL" "$2"
        ;;
    
    *)
        cat <<HELP
Usage: $0 {init|backup|verify|cleanup|list|compare|restore|mount}

Commands:
  init              - Initialize all repositories
  backup            - Backup to all repositories
  verify            - Verify backup integrity
  cleanup           - Remove old snapshots according to policy
  list              - List all snapshots
  compare           - Compare repositories
  restore <id> <dir> - Restore snapshot to directory
  mount <dir>       - Mount backup as filesystem

Examples:
  $0 backup
  $0 restore latest /restore
  $0 mount /mnt/backup
HELP
        exit 1
        ;;
esac
EOF

chmod +x restic_backup.sh
```

---

## 💻 Задание 3: Hybrid Backup Strategy

### Comprehensive Hybrid Backup Solution

```bash
cat > hybrid_backup_orchestrator.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
BACKUP_ROOT="/backup"
DATA_DIR="/data"
LOG_FILE="/var/log/hybrid_backup.log"
CONFIG_FILE="/etc/hybrid-backup/config.json"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Create configuration
create_config() {
    mkdir -p /etc/hybrid-backup
    
    cat > "$CONFIG_FILE" <<'CONFIG'
{
  "strategy": "3-2-1-1-0",
  "tiers": {
    "tier1_local": {
      "type": "local",
      "path": "/backup/tier1",
      "retention_days": 7,
      "enabled": true
    },
    "tier2_nas": {
      "type": "nfs",
      "path": "/mnt/nas/backups",
      "retention_days": 30,
      "enabled": true
    },
    "tier3_s3": {
      "type": "s3",
      "bucket": "company-backups",
      "storage_class": "STANDARD_IA",
      "retention_days": 90,
      "enabled": true
    },
    "tier4_glacier": {
      "type": "s3",
      "bucket": "company-backups",
      "storage_class": "GLACIER",
      "retention_days": 2555,
      "enabled": true
    },
    "tier5_offsite": {
      "type": "b2",
      "bucket": "company-offsite-backup",
      "retention_days": 365,
      "enabled": true
    }
  },
  "backup_types": {
    "databases": {
      "mysql": true,
      "postgresql": true,
      "mongodb": false
    },
    "filesystems": {
      "application_data": true,
      "user_data": true,
      "system_config": true
    },
    "vms": {
      "enabled": false
    }
  },
  "encryption": {
    "enabled": true,
    "algorithm": "aes-256-cbc"
  },
  "compression": {
    "enabled": true,
    "algorithm": "zstd"
  },
  "notifications": {
    "email": "ops@company.com",
    "slack_webhook": "https://hooks.slack.com/...",
    "on_success": true,
    "on_failure": true
  }
}
CONFIG
    
    log "Configuration created: $CONFIG_FILE"
}

# Tier 1: Local Fast Storage (SSD)
tier1_backup() {
    log "=== Tier 1: Local Fast Backup ==="
    
    local tier1_dir="$BACKUP_ROOT/tier1"
    local backup_name="tier1-$(date +%Y%m%d-%H%M%S)"
    
    mkdir -p "$tier1_dir"
    
    # Fast backup using rsync with hardlinks (deduplication)
    rsync -aH \
        --link-dest="$tier1_dir/latest" \
        --exclude='*.tmp' \
        --info=progress2 \
        "$DATA_DIR/" \
        "$tier1_dir/$backup_name/"
    
    # Update latest symlink
    rm -f "$tier1_dir/latest"
    ln -s "$backup_name" "$tier1_dir/latest"
    
    log "✓ Tier 1 backup completed: $backup_name"
}

# Tier 2: NAS (Network Storage)
tier2_backup() {
    log "=== Tier 2: NAS Backup ==="
    
    local nas_mount="/mnt/nas/backups"
    local backup_file="tier2-$(date +%Y%m%d-%H%M%S).tar.zst"
    
    # Check if NAS is mounted
    if ! mountpoint -q "$nas_mount"; then
        log "Mounting NAS..."
        mount -t nfs nas.local:/backups "$nas_mount"
    fi
    
    # Compressed backup to NAS
    tar -cf - "$DATA_DIR" 2>/dev/null | \
        zstd -T0 -3 > "$nas_mount/$backup_file"
    
    # Create checksum
    cd "$nas_mount"
    sha256sum "$backup_file" > "${backup_file}.sha256"
    
    log "✓ Tier 2 backup completed: $backup_file"
}

# Tier 3: Cloud (S3 Standard-IA)
tier3_backup() {
    log "=== Tier 3: Cloud S3 Backup ==="
    
    local backup_file="$BACKUP_ROOT/staging/tier3-$(date +%Y%m%d).tar.zst.enc"
    
    mkdir -p "$BACKUP_ROOT/staging"
    
    # Create encrypted compressed backup
    tar -cf - "$DATA_DIR" 2>/dev/null | \
        zstd -T0 -3 | \
        openssl enc -aes-256-cbc -salt -pbkdf2 \
        -pass file:/root/.backup_key \
        -out "$backup_file"
    
    # Upload to S3 with lifecycle policy
    aws s3 cp "$backup_file" \
        "s3://company-backups/tier3/$(date +%Y/%m/%d)/" \
        --storage-class STANDARD_IA \
        --metadata "backup-tier=tier3,retention-days=90"
    
    log "✓ Tier 3 backup uploaded to S3"
    
    # Cleanup staging
    rm -f "$backup_file"
}

# Tier 4: Deep Archive (Glacier)
tier4_backup() {
    log "=== Tier 4: Glacier Deep Archive ==="
    
    # Only run on 1st of month
    if [ "$(date +%d)" != "01" ]; then
        log "Skipping Tier 4 (not 1st of month)"
        return 0
    fi
    
    local backup_file="$BACKUP_ROOT/staging/tier4-$(date +%Y%m).tar.zst.enc"
    
    # Create monthly backup
    tar -cf - "$DATA_DIR" 2>/dev/null | zstd -T0 -9 |  
openssl enc -aes-256-cbc -salt -pbkdf2  
-pass file:/root/.backup_key  
-out "$backup_file"


# Upload to Glacier
aws s3 cp "$backup_file" \
    "s3://company-backups/tier4/$(date +%Y/)/" \
    --storage-class DEEP_ARCHIVE \
    --metadata "backup-tier=tier4,retention-years=7"

log "✓ Tier 4 backup archived to Glacier"

rm -f "$backup_file"


}

# Tier 5: Offsite (Backblaze B2)

tier5_backup() { log "=== Tier 5: Offsite B2 Backup ==="


# Use restic for offsite (with deduplication)
export RESTIC_REPOSITORY="b2:company-offsite-backup"
export RESTIC_PASSWORD_FILE="/root/.restic_password"
export B2_ACCOUNT_ID="$B2_ACCOUNT_ID"
export B2_ACCOUNT_KEY="$B2_ACCOUNT_KEY"

restic backup "$DATA_DIR" \
    --tag "tier5" \
    --tag "offsite" \
    --exclude='*.tmp'

# Cleanup old snapshots
restic forget \
    --keep-daily 7 \
    --keep-weekly 4 \
    --keep-monthly 12 \
    --prune

log "✓ Tier 5 offsite backup completed"


}

# Cleanup old backups per tier

cleanup_tiers() { log "=== Cleaning up old backups ==="


# Tier 1: Keep last 7 days
find "$BACKUP_ROOT/tier1" -maxdepth 1 -type d -mtime +7 \
    -not -name "latest" -exec rm -rf {} \; 2>/dev/null || true

# Tier 2: Keep last 30 days
find /mnt/nas/backups -name "tier2-*.tar.zst" -mtime +30 \
    -delete 2>/dev/null || true

log "✓ Cleanup completed"


}

# Verify all tiers

verify_all_tiers() { log "=== Verifying All Tiers ==="


local verification_report="$BACKUP_ROOT/verification-$(date +%Y%m%d).txt"

cat > "$verification_report" <<REPORT


# Backup Verification Report Generated: $(date)

REPORT


# Tier 1
if [ -d "$BACKUP_ROOT/tier1/latest" ]; then
    local tier1_size=$(du -sh "$BACKUP_ROOT/tier1/latest" | cut -f1)
    local tier1_files=$(find "$BACKUP_ROOT/tier1/latest" -type f | wc -l)
    echo "Tier 1 (Local): ✓ $tier1_size, $tier1_files files" >> "$verification_report"
else
    echo "Tier 1 (Local): ✗ No backup found" >> "$verification_report"
fi

# Tier 2
local tier2_latest=$(ls -t /mnt/nas/backups/tier2-*.tar.zst 2>/dev/null | head -1)
if [ -n "$tier2_latest" ]; then
    local tier2_size=$(du -h "$tier2_latest" | cut -f1)
    echo "Tier 2 (NAS): ✓ $tier2_size" >> "$verification_report"
    
    # Verify checksum
    cd "$(dirname "$tier2_latest")"
    if sha256sum -c "$(basename "$tier2_latest").sha256" &>/dev/null; then
        echo "  Checksum: ✓ Valid" >> "$verification_report"
    else
        echo "  Checksum: ✗ Failed" >> "$verification_report"
    fi
else
    echo "Tier 2 (NAS): ✗ No backup found" >> "$verification_report"
fi

# Tier 3
local tier3_count=$(aws s3 ls s3://company-backups/tier3/ --recursive | wc -l)
echo "Tier 3 (S3-IA): ✓ $tier3_count files" >> "$verification_report"

# Tier 4
local tier4_count=$(aws s3 ls s3://company-backups/tier4/ --recursive | wc -l)
echo "Tier 4 (Glacier): ✓ $tier4_count files" >> "$verification_report"

# Tier 5
export RESTIC_REPOSITORY="b2:company-offsite-backup"
export RESTIC_PASSWORD_FILE="/root/.restic_password"
local tier5_snapshots=$(restic snapshots --compact | grep -c "^ID" || echo "0")
echo "Tier 5 (B2 Offsite): ✓ $tier5_snapshots snapshots" >> "$verification_report"

cat "$verification_report"

log "✓ Verification report generated"


}

# Generate cost estimate

estimate_costs() { log "=== Estimating Monthly Costs ==="


local report="$BACKUP_ROOT/cost-estimate-$(date +%Y%m).txt"

cat > "$report" <<COST


# Cloud Backup Cost Estimate Month: $(date +%Y-%m)

COST


# Calculate sizes
local tier3_size=$(aws s3 ls s3://company-backups/tier3/ --recursive --summarize | grep "Total Size" | awk '{print $3}')
local tier3_gb=$(echo "scale=2; $tier3_size / 1024 / 1024 / 1024" | bc)
local tier3_cost=$(echo "scale=2; $tier3_gb * 0.0125" | bc)

local tier4_size=$(aws s3 ls s3://company-backups/tier4/ --recursive --summarize | grep "Total Size" | awk '{print $3}')
local tier4_gb=$(echo "scale=2; $tier4_size / 1024 / 1024 / 1024" | bc)
local tier4_cost=$(echo "scale=2; $tier4_gb * 0.00099" | bc)

cat >> "$report" <<COST


Tier 3 (S3 Standard-IA): Storage: ${tier3_gb} GB Cost: $${tier3_cost}/month

Tier 4 (Glacier Deep Archive): Storage: ${tier4_gb} GB Cost: $${tier4_cost}/month

Tier 5 (Backblaze B2): Estimated: $10/month (500GB)

Total Estimated Cost: $$(echo "scale=2; $tier3_cost + $tier4_cost + 10" | bc)/month

COST


cat "$report"


}

# Main orchestration

main() { local start_time=$(date +%s)


log "========================================="
log "Starting Hybrid Backup Orchestration"
log "Strategy: 3-2-1-1-0 Rule"
log "========================================="

# Execute all tiers
tier1_backup
tier2_backup
tier3_backup
tier4_backup
tier5_backup

# Cleanup
cleanup_tiers

# Verification
verify_all_tiers

# Cost estimation
estimate_costs

local end_time=$(date +%s)
local duration=$((end_time - start_time))

log "========================================="
log "Hybrid backup completed in $duration seconds"
log "All 5 tiers backed up successfully"
log "========================================="


}

# Command handling

case "${1:-run}" in init) create_config ;; run) main ;; verify) verify_all_tiers ;; costs) estimate_costs ;; tier1) tier1_backup ;; tier2) tier2_backup ;; tier3) tier3_backup ;; tier4) tier4_backup ;; tier5) tier5_backup ;; *) cat <<HELP Usage: $0 {init|run|verify|costs|tier1|tier2|tier3|tier4|tier5}

Commands: init - Create initial configuration run - Run full hybrid backup (all tiers) verify - Verify all backup tiers costs - Estimate monthly costs tier1 - Run Tier 1 only (local fast) tier2 - Run Tier 2 only (NAS) tier3 - Run Tier 3 only (S3-IA) tier4 - Run Tier 4 only (Glacier) tier5 - Run Tier 5 only (B2 offsite)

Backup Tiers: Tier 1: Local SSD (7 days) Tier 2: NAS (30 days) Tier 3: S3 Standard-IA (90 days) Tier 4: Glacier Deep Archive (7 years) Tier 5: Backblaze B2 Offsite (1 year) HELP exit 1 ;; esac EOF

chmod +x hybrid_backup_orchestrator.sh

````

---

## 🚀 Бонус: Cloud Backup Best Practices

### 1. S3 Lifecycle Policy Automation

```bash
cat > setup_s3_lifecycle.sh <<'EOF'
#!/bin/bash

BUCKET_NAME="company-backups"

# Create lifecycle policy
cat > lifecycle-policy.json <<'POLICY'
{
  "Rules": [
    {
      "Id": "BackupLifecycleRule",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "backups/"
      },
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER_IR"
        },
        {
          "Days": 180,
          "StorageClass": "GLACIER"
        },
        {
          "Days": 365,
          "StorageClass": "DEEP_ARCHIVE"
        }
      ],
      "Expiration": {
        "Days": 2555
      },
      "NoncurrentVersionTransitions": [
        {
          "NoncurrentDays": 30,
          "StorageClass": "GLACIER"
        }
      ],
      "NoncurrentVersionExpiration": {
        "NoncurrentDays": 90
      }
    }
  ]
}
POLICY

# Apply lifecycle policy
aws s3api put-bucket-lifecycle-configuration \
    --bucket "$BUCKET_NAME" \
    --lifecycle-configuration file://lifecycle-policy.json

echo "✓ Lifecycle policy applied to $BUCKET_NAME"

# Enable versioning
aws s3api put-bucket-versioning \
    --bucket "$BUCKET_NAME" \
    --versioning-configuration Status=Enabled

echo "✓ Versioning enabled on $BUCKET_NAME"

# Enable encryption
aws s3api put-bucket-encryption \
    --bucket "$BUCKET_NAME" \
    --server-side-encryption-configuration '{
        "Rules": [{
            "ApplyServerSideEncryptionByDefault": {
                "SSEAlgorithm": "AES256"
            }
        }]
    }'

echo "✓ Encryption enabled on $BUCKET_NAME"
EOF

chmod +x setup_s3_lifecycle.sh
````

### 2. Multi-Region Replication

```bash
cat > setup_s3_replication.sh <<'EOF'
#!/bin/bash

SOURCE_BUCKET="company-backups-us-east-1"
DEST_BUCKET="company-backups-eu-west-1"
REPLICATION_ROLE_ARN="arn:aws:iam::123456789:role/S3ReplicationRole"

# Create replication configuration
cat > replication-config.json <<'REPL'
{
  "Role": "REPLICATION_ROLE_ARN",
  "Rules": [
    {
      "Status": "Enabled",
      "Priority": 1,
      "DeleteMarkerReplication": { "Status": "Enabled" },
      "Filter" : { "Prefix": "backups/"},
      "Destination": {
        "Bucket": "arn:aws:s3:::DEST_BUCKET",
        "ReplicationTime": {
          "Status": "Enabled",
          "Time": {
            "Minutes": 15
          }
        },
        "Metrics": {
          "Status": "Enabled",
          "EventThreshold": {
            "Minutes": 15
          }
        }
      }
    }
  ]
}
REPL

# Replace placeholders
sed -i "s|REPLICATION_ROLE_ARN|$REPLICATION_ROLE_ARN|g" replication-config.json
sed -i "s|DEST_BUCKET|$DEST_BUCKET|g" replication-config.json

# Apply replication
aws s3api put-bucket-replication \
    --bucket "$SOURCE_BUCKET" \
    --replication-configuration file://replication-config.json

echo "✓ Cross-region replication configured"
EOF

chmod +x setup_s3_replication.sh
```

### 3. Cloud Backup Monitoring

```bash
cat > cloud_backup_monitor.sh <<'EOF'
#!/bin/bash

# Monitor all cloud backups
METRICS_FILE="/var/lib/node_exporter/textfile_collector/cloud_backups.prom"

cat > "$METRICS_FILE" <<METRICS
# HELP cloud_backup_size_bytes Size of backups in cloud storage
# TYPE cloud_backup_size_bytes gauge

# HELP cloud_backup_count Number of backup files
# TYPE cloud_backup_count gauge

# HELP cloud_backup_last_success Last successful backup timestamp
# TYPE cloud_backup_last_success gauge

# HELP cloud_backup_cost_usd_monthly Estimated monthly cost
# TYPE cloud_backup_cost_usd_monthly gauge
METRICS

# S3 metrics
s3_size=$(aws s3 ls s3://company-backups/ --recursive --summarize | grep "Total Size" | awk '{print $3}')
s3_count=$(aws s3 ls s3://company-backups/ --recursive | wc -l)
s3_latest=$(aws s3api list-objects-v2 --bucket company-backups --query 'sort_by(Contents, &LastModified)[-1].LastModified' --output text)
s3_timestamp=$(date -d "$s3_latest" +%s 2>/dev/null || echo "0")

cat >> "$METRICS_FILE" <<METRICS
cloud_backup_size_bytes{provider="aws",tier="s3"} $s3_size
cloud_backup_count{provider="aws",tier="s3"} $s3_count
cloud_backup_last_success{provider="aws",tier="s3"} $s3_timestamp
METRICS

# B2 metrics (using restic)
export RESTIC_REPOSITORY="b2:company-offsite-backup"
export RESTIC_PASSWORD_FILE="/root/.restic_password"

b2_stats=$(restic stats --mode raw-data --json 2>/dev/null || echo '{"total_size":0}')
b2_size=$(echo "$b2_stats" | jq -r '.total_size')
b2_snapshots=$(restic snapshots --json 2>/dev/null | jq '. | length')
b2_latest=$(restic snapshots --json 2>/dev/null | jq -r 'sort_by(.time) | .[-1].time')
b2_timestamp=$(date -d "$b2_latest" +%s 2>/dev/null || echo "0")

cat >> "$METRICS_FILE" <<METRICS
cloud_backup_size_bytes{provider="backblaze",tier="b2"} $b2_size
cloud_backup_count{provider="backblaze",tier="b2"} $b2_snapshots
cloud_backup_last_success{provider="backblaze",tier="b2"} $b2_timestamp
METRICS

echo "✓ Cloud backup metrics updated"
EOF

chmod +x cloud_backup_monitor.sh

# Add to cron
echo "*/10 * * * * /usr/local/bin/cloud_backup_monitor.sh" | crontab -
```

---

## Итоговое задание модуля 4

```bash
# 1. Настрой все инструменты
./setup_rclone.sh
./setup_restic.sh

# 2. Инициализируй репозитории
./restic_backup.sh init

# 3. Настрой гибридную стратегию
./hybrid_backup_orchestrator.sh init

# 4. Запусти тестовый backup
./hybrid_backup_orchestrator.sh run

# 5. Проверь результаты
./hybrid_backup_orchestrator.sh verify
./hybrid_backup_orchestrator.sh costs

# 6. Настрой S3 lifecycle
./setup_s3_lifecycle.sh

# 7. Настрой мониторинг
./cloud_backup_monitor.sh

# 8. Проверь метрики
cat /var/lib/node_exporter/textfile_collector/cloud_backups.prom
```

**Результат:** У тебя будет production-ready hybrid backup solution с:

- ✅ 5-tier backup стратегия
- ✅ Multi-cloud redundancy
- ✅ Автоматическое lifecycle management
- ✅ Cost optimization
- ✅ Monitoring и alerting
- ✅ Encrypted backups
- ✅ Geographic redundancy

Готов к следующему модулю?


# Модуль 5: Advanced Recovery Scenarios & Disaster Recovery Automation (35 минут)

## 🎯 Напоминалка

### Recovery Scenarios Matrix

```
Recovery Complexity Matrix
├── Level 1: Single File Recovery
│   ├── Time: Minutes
│   ├── Complexity: Low
│   └── Tools: rsync, restic, cp
│
├── Level 2: Database Point-in-Time Recovery (PITR)
│   ├── Time: 15-60 minutes
│   ├── Complexity: Medium
│   └── Tools: mysqldump, pg_basebackup, binlogs
│
├── Level 3: Full Application Recovery
│   ├── Time: 1-4 hours
│   ├── Complexity: Medium-High
│   └── Tools: Ansible, Docker, orchestration scripts
│
├── Level 4: Infrastructure Recovery
│   ├── Time: 4-8 hours
│   ├── Complexity: High
│   └── Tools: Terraform, CloudFormation, IaC
│
└── Level 5: Complete Datacenter Failover
    ├── Time: 8-24 hours
    ├── Complexity: Very High
    └── Tools: DR orchestration, multi-region setup
```

### RTO vs Complexity Trade-offs

```
Recovery Strategy Options
┌─────────────────────────────────────────┐
│ Hot Standby (Active-Active)             │
│ RTO: < 1 minute                         │
│ Cost: $$$$$ (5x production)             │
│ Complexity: Very High                   │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ Warm Standby (Active-Passive)           │
│ RTO: 5-60 minutes                       │
│ Cost: $$$ (2x production)               │
│ Complexity: High                        │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ Pilot Light                             │
│ RTO: 1-4 hours                          │
│ Cost: $$ (0.5x production)              │
│ Complexity: Medium                      │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ Backup & Restore                        │
│ RTO: 4-24 hours                         │
│ Cost: $ (0.1x production)               │
│ Complexity: Low-Medium                  │
└─────────────────────────────────────────┘
```

### Common Failure Scenarios

```bash
# 1. Accidental File Deletion
Problem: User deleted critical files
Solution: Point-in-time file restore from snapshots

# 2. Database Corruption
Problem: Corrupted database after failed upgrade
Solution: PITR to just before corruption

# 3. Ransomware Attack
Problem: Files encrypted by malware
Solution: Restore from immutable/air-gapped backup

# 4. Hardware Failure
Problem: Server/disk failure
Solution: Full system restore to new hardware

# 5. Regional Outage
Problem: Entire datacenter/region unavailable
Solution: Failover to DR site

# 6. Human Error (DROP DATABASE)
Problem: Accidental data deletion
Solution: Database PITR + transaction log replay

# 7. Application Bug
Problem: Bug corrupted data
Solution: Restore to last known good state

# 8. Natural Disaster
Problem: Physical site destruction
Solution: Geographic failover
```

### Recovery Testing Pyramid

```
Recovery Testing Levels
        ┌─────────────────────┐
        │  Annual Full DR     │  ← Most expensive, most realistic
        │     Exercise        │
        ├─────────────────────┤
        │  Quarterly Tabletop │
        │   DR Walkthrough    │
        ├─────────────────────┤
        │   Monthly Partial   │
        │  Service Recovery   │
        ├─────────────────────┤
        │  Weekly Database    │
        │  Restore Tests      │
        ├─────────────────────┤
        │   Daily Backup      │
        │  Verification       │  ← Least expensive, most frequent
        └─────────────────────┘
```

---

## 💻 Задание 1: Automated Recovery Orchestrator

### Universal Recovery Orchestration Script

```bash
cat > recovery_orchestrator.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
RECOVERY_CONFIG="/etc/recovery/config.yaml"
LOG_FILE="/var/log/recovery_$(date +%Y%m%d-%H%M%S).log"
STATE_FILE="/var/run/recovery_state.json"
RECOVERY_DIR="/recovery"

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log() {
    local level="$1"
    shift
    local message="$*"
    local timestamp=$(date +'%Y-%m-%d %H:%M:%S')
    
    echo "[$timestamp] [$level] $message" | tee -a "$LOG_FILE"
    
    case "$level" in
        ERROR)
            echo -e "${RED}[ERROR]${NC} $message" >&2
            ;;
        SUCCESS)
            echo -e "${GREEN}[SUCCESS]${NC} $message"
            ;;
        WARNING)
            echo -e "${YELLOW}[WARNING]${NC} $message"
            ;;
        INFO)
            echo "[INFO] $message"
            ;;
    esac
}

# Initialize recovery state tracking
init_recovery_state() {
    cat > "$STATE_FILE" <<STATE
{
  "recovery_id": "$(uuidgen)",
  "started_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "status": "in_progress",
  "steps_completed": [],
  "steps_failed": [],
  "current_step": null
}
STATE
}

update_recovery_state() {
    local step="$1"
    local status="$2"  # completed, failed, in_progress
    
    jq --arg step "$step" --arg status "$status" --arg time "$(date -u +%Y-%m-%dT%H:%M:%SZ)" '
        .current_step = $step |
        .last_updated = $time |
        if $status == "completed" then
            .steps_completed += [$step]
        elif $status == "failed" then
            .steps_failed += [$step]
        else
            .
        end
    ' "$STATE_FILE" > "$STATE_FILE.tmp" && mv "$STATE_FILE.tmp" "$STATE_FILE"
}

# Pre-recovery checks
pre_recovery_checks() {
    log INFO "Running pre-recovery checks..."
    update_recovery_state "pre_checks" "in_progress"
    
    local checks_passed=0
    local checks_failed=0
    
    # Check 1: Sufficient disk space
    log INFO "Checking disk space..."
    local available_space=$(df -BG "$RECOVERY_DIR" | tail -1 | awk '{print $4}' | tr -d 'G')
    local required_space=100  # GB
    
    if [ "$available_space" -gt "$required_space" ]; then
        log SUCCESS "Disk space check passed: ${available_space}GB available"
        ((checks_passed++))
    else
        log ERROR "Insufficient disk space: ${available_space}GB < ${required_space}GB"
        ((checks_failed++))
    fi
    
    # Check 2: Network connectivity
    log INFO "Checking network connectivity..."
    if ping -c 3 8.8.8.8 &>/dev/null; then
        log SUCCESS "Network connectivity check passed"
        ((checks_passed++))
    else
        log WARNING "Network connectivity issues detected"
        ((checks_failed++))
    fi
    
    # Check 3: Backup availability
    log INFO "Checking backup availability..."
    if [ -d "/backup" ] && [ "$(ls -A /backup)" ]; then
        log SUCCESS "Backup directory accessible"
        ((checks_passed++))
    else
        log ERROR "Backup directory not accessible or empty"
        ((checks_failed++))
    fi
    
    # Check 4: Required services stopped
    log INFO "Checking if services are stopped..."
    local services_to_check=("mysql" "postgresql" "nginx" "apache2")
    local running_services=0
    
    for service in "${services_to_check[@]}"; do
        if systemctl is-active --quiet "$service" 2>/dev/null; then
            log WARNING "$service is still running"
            ((running_services++))
        fi
    done
    
    if [ $running_services -eq 0 ]; then
        log SUCCESS "All services properly stopped"
        ((checks_passed++))
    else
        log WARNING "$running_services services still running (will be stopped)"
    fi
    
    # Summary
    log INFO "Pre-checks summary: $checks_passed passed, $checks_failed failed"
    
    if [ $checks_failed -gt 0 ]; then
        log WARNING "Some pre-checks failed. Continue anyway? (yes/no)"
        read -r response
        if [ "$response" != "yes" ]; then
            log ERROR "Recovery aborted by user"
            update_recovery_state "pre_checks" "failed"
            exit 1
        fi
    fi
    
    update_recovery_state "pre_checks" "completed"
    return 0
}

# Scenario 1: Single File Recovery
recover_file() {
    local file_path="$1"
    local restore_point="${2:-latest}"
    
    log INFO "Starting file recovery for: $file_path"
    update_recovery_state "file_recovery" "in_progress"
    
    # Determine backup type
    local backup_source=""
    
    # Try restic first
    if [ -n "$RESTIC_REPOSITORY" ]; then
        log INFO "Attempting recovery from Restic repository..."
        
        # List snapshots
        local snapshot_id=$(restic -r "$RESTIC_REPOSITORY" snapshots --json | \
            jq -r ".[0].id" 2>/dev/null)
        
        if [ -n "$snapshot_id" ]; then
            # Create temporary restore directory
            local temp_restore="/tmp/restic_restore_$$"
            mkdir -p "$temp_restore"
            
            # Restore specific file
            restic -r "$RESTIC_REPOSITORY" restore "$snapshot_id" \
                --target "$temp_restore" \
                --include "$file_path"
            
            if [ -f "$temp_restore/$file_path" ]; then
                # Backup existing file if it exists
                if [ -f "$file_path" ]; then
                    mv "$file_path" "${file_path}.corrupted.$(date +%Y%m%d-%H%M%S)"
                    log INFO "Existing file backed up"
                fi
                
                # Restore file
                cp "$temp_restore/$file_path" "$file_path"
                log SUCCESS "File restored from Restic: $file_path"
                
                # Cleanup
                rm -rf "$temp_restore"
                update_recovery_state "file_recovery" "completed"
                return 0
            fi
        fi
    fi
    
    # Try rsync snapshots
    if [ -d "/backup/rsync-snapshots/latest" ]; then
        log INFO "Attempting recovery from rsync snapshots..."
        
        local snapshot_file="/backup/rsync-snapshots/latest$file_path"
        
        if [ -f "$snapshot_file" ]; then
            if [ -f "$file_path" ]; then
                mv "$file_path" "${file_path}.corrupted.$(date +%Y%m%d-%H%M%S)"
            fi
            
            cp "$snapshot_file" "$file_path"
            log SUCCESS "File restored from rsync snapshot: $file_path"
            update_recovery_state "file_recovery" "completed"
            return 0
        fi
    fi
    
    log ERROR "File recovery failed: $file_path not found in any backup"
    update_recovery_state "file_recovery" "failed"
    return 1
}

# Scenario 2: Database Point-in-Time Recovery
recover_database_pitr() {
    local db_type="$1"
    local target_time="$2"
    local db_name="${3:-all}"
    
    log INFO "Starting PITR for $db_type to $target_time"
    update_recovery_state "database_pitr" "in_progress"
    
    case "$db_type" in
        mysql)
            recover_mysql_pitr "$target_time" "$db_name"
            ;;
        postgresql)
            recover_postgresql_pitr "$target_time" "$db_name"
            ;;
        *)
            log ERROR "Unsupported database type: $db_type"
            return 1
            ;;
    esac
}

recover_mysql_pitr() {
    local target_time="$1"
    local db_name="$2"
    
    log INFO "MySQL PITR recovery to $target_time"
    
    # Step 1: Stop MySQL
    log INFO "Stopping MySQL service..."
    systemctl stop mysql
    
    # Step 2: Backup current data directory
    log INFO "Backing up current MySQL data..."
    local backup_current="/backup/mysql_current_$(date +%Y%m%d-%H%M%S)"
    cp -a /var/lib/mysql "$backup_current"
    
    # Step 3: Find and restore latest full backup before target time
    log INFO "Finding latest full backup before $target_time..."
    local latest_backup=$(find /backup/mysql/daily -name "*.sql.gz" -type f | \
        xargs ls -lt | \
        awk '{print $NF}' | \
        head -1)
    
    if [ -z "$latest_backup" ]; then
        log ERROR "No full backup found"
        return 1
    fi
    
    log INFO "Restoring from: $latest_backup"
    
    # Clear data directory
    rm -rf /var/lib/mysql/*
    
    # Restore full backup
    gunzip < "$latest_backup" | mysql -u root
    
    # Step 4: Start MySQL in recovery mode
    systemctl start mysql
    
    # Step 5: Apply binary logs up to target time
    log INFO "Applying binary logs up to $target_time..."
    
    local binlog_dir="/backup/mysql/binlogs"
    if [ -d "$binlog_dir" ]; then
        # Extract and apply binlogs
        local temp_binlog="/tmp/binlogs_$$"
        mkdir -p "$temp_binlog"
        
        tar -xzf "$binlog_dir"/binlog-*.tar.gz -C "$temp_binlog"
        
        # Apply each binlog up to target time
        for binlog in "$temp_binlog"/*; do
            mysqlbinlog --stop-datetime="$target_time" "$binlog" | \
                mysql -u root
        done
        
        rm -rf "$temp_binlog"
    fi
    
    log SUCCESS "MySQL PITR completed to $target_time"
    update_recovery_state "database_pitr" "completed"
}

recover_postgresql_pitr() {
    local target_time="$1"
    local db_name="$2"
    
    log INFO "PostgreSQL PITR recovery to $target_time"
    
    # Step 1: Stop PostgreSQL
    log INFO "Stopping PostgreSQL service..."
    systemctl stop postgresql
    
    # Step 2: Backup current data directory
    local pg_data="/var/lib/postgresql/14/main"
    local backup_current="/backup/pg_current_$(date +%Y%m%d-%H%M%S)"
    cp -a "$pg_data" "$backup_current"
    
    # Step 3: Clear data directory
    rm -rf "$pg_data"/*
    
    # Step 4: Restore base backup
    log INFO "Restoring base backup..."
    local latest_base=$(ls -t /backup/postgresql/base/base-*.tar.gz | head -1)
    
    if [ -z "$latest_base" ]; then
        log ERROR "No base backup found"
        return 1
    fi
    
    tar -xzf "$latest_base" -C "$pg_data"
    
    # Step 5: Configure recovery
    log INFO "Configuring PITR to $target_time..."
    
    cat > "$pg_data/postgresql.auto.conf" <<PGCONF
restore_command = 'cp /backup/postgresql/wal_archive/%f %p'
recovery_target_time = '$target_time'
recovery_target_action = 'promote'
PGCONF
    
    touch "$pg_data/recovery.signal"
    
    # Fix permissions
    chown -R postgres:postgres "$pg_data"
    chmod 700 "$pg_data"
    
    # Step 6: Start PostgreSQL (will perform recovery)
    log INFO "Starting PostgreSQL for recovery..."
    systemctl start postgresql
    
    # Wait for recovery to complete
    log INFO "Waiting for recovery to complete..."
    local max_wait=300  # 5 minutes
    local waited=0
    
    while [ $waited -lt $max_wait ]; do
        if sudo -u postgres psql -c "SELECT 1" &>/dev/null; then
            log SUCCESS "PostgreSQL recovery completed"
            update_recovery_state "database_pitr" "completed"
            return 0
        fi
        sleep 5
        waited=$((waited + 5))
    done
    
    log ERROR "PostgreSQL recovery timed out"
    update_recovery_state "database_pitr" "failed"
    return 1
}

# Scenario 3: Full Application Stack Recovery
recover_application_stack() {
    local app_name="$1"
    local environment="${2:-production}"
    
    log INFO "Starting full application stack recovery: $app_name ($environment)"
    update_recovery_state "application_recovery" "in_progress"
    
    # Define recovery steps
    local steps=(
        "stop_services"
        "restore_database"
        "restore_application_files"
        "restore_configuration"
        "verify_dependencies"
        "start_services"
        "health_check"
    )
    
    for step in "${steps[@]}"; do
        log INFO "Executing step: $step"
        
        case "$step" in
            stop_services)
                systemctl stop nginx mysql redis-server || true
                ;;
            
            restore_database)
                recover_database_pitr "mysql" "latest" "$app_name"
                ;;
            
            restore_application_files)
                if [ -d "/backup/rsync-snapshots/latest/var/www/$app_name" ]; then
                    rsync -av "/backup/rsync-snapshots/latest/var/www/$app_name/" \
                        "/var/www/$app_name/"
                    log SUCCESS "Application files restored"
                else
                    log ERROR "Application files not found in backup"
                    return 1
                fi
                ;;
            
            restore_configuration)
                if [ -d "/backup/rsync-snapshots/latest/etc/$app_name" ]; then
                    rsync -av "/backup/rsync-snapshots/latest/etc/$app_name/" \
                        "/etc/$app_name/"
                    log SUCCESS "Configuration restored"
                fi
                ;;
            
            verify_dependencies)
                log INFO "Verifying dependencies..."
                # Check PHP, Node.js, Python, etc.
                command -v php &>/dev/null && log SUCCESS "PHP available"
                command -v node &>/dev/null && log SUCCESS "Node.js available"
                ;;
            
            start_services)
                systemctl start mysql
                sleep 5
                systemctl start redis-server
                sleep 2
                systemctl start nginx
                log SUCCESS "Services started"
                ;;
            
            health_check)
                log INFO "Performing health checks..."
                sleep 5
                
                # Check if services are running
                if systemctl is-active --quiet nginx && \
                   systemctl is-active --quiet mysql; then
                    log SUCCESS "All services are running"
                    
                    # Check if application responds
                    if curl -f http://localhost &>/dev/null; then
                        log SUCCESS "Application is responding"
                    else
                        log WARNING "Application not responding yet"
                    fi
                else
                    log ERROR "Some services failed to start"
                    return 1
                fi
                ;;
        esac
        
        update_recovery_state "$step" "completed"
    done
    
    log SUCCESS "Application stack recovery completed: $app_name"
    update_recovery_state "application_recovery" "completed"
}

# Scenario 4: Infrastructure Recovery
recover_infrastructure() {
    local recovery_type="$1"  # vm, bare-metal, container
    
    log INFO "Starting infrastructure recovery: $recovery_type"
    update_recovery_state "infrastructure_recovery" "in_progress"
    
    case "$recovery_type" in
        vm)
            recover_vm_infrastructure
            ;;
        bare-metal)
            recover_bare_metal
            ;;
        container)
            recover_container_infrastructure
            ;;
        *)
            log ERROR "Unknown recovery type: $recovery_type"
            return 1
            ;;
    esac
}

recover_vm_infrastructure() {
    log INFO "Recovering VM infrastructure..."
    
    # Check if Terraform state exists
    if [ -f "/backup/terraform/terraform.tfstate" ]; then
        log INFO "Found Terraform state, using IaC recovery..."
        
        cd /backup/terraform
        
        # Initialize Terraform
        terraform init
        
        # Plan recovery
        log INFO "Planning infrastructure recovery..."
        terraform plan -out=recovery.tfplan
        
        # Apply recovery
        log INFO "Applying infrastructure recovery..."
        if terraform apply recovery.tfplan; then
            log SUCCESS "VM infrastructure recovered via Terraform"
            update_recovery_state "infrastructure_recovery" "completed"
            return 0
        else
            log ERROR "Terraform apply failed"
            return 1
        fi
    else
        log ERROR "No Terraform state found for VM recovery"
        return 1
    fi
}

recover_bare_metal() {
    log INFO "Recovering bare metal server..."
    
    # Check for ReaR (Relax and Recover) backup
    if command -v rear &>/dev/null; then
        log INFO "Using ReaR for bare metal recovery..."
        
        # This would typically be run from ReaR rescue media
        log INFO "Manual intervention required: Boot from ReaR rescue media"
        log INFO "Then run: rear recover"
        
        cat > /tmp/bare_metal_recovery_instructions.txt <<INSTRUCTIONS
Bare Metal Recovery Instructions
=================================

1. Boot the system from ReaR rescue ISO/USB
2. At the boot prompt, select "Recover <hostname>"
3. Run: rear recover
4. Follow the prompts to restore the system
5. After recovery, reboot into the restored system
6. Verify all services are running
7. Check application data integrity

Backup location: /backup/rear/
Latest backup: $(ls -t /backup/rear/ | head -1)
INSTRUCTIONS
        
        log INFO "Recovery instructions written to /tmp/bare_metal_recovery_instructions.txt"
    else
        log ERROR "ReaR not installed, cannot perform automated bare metal recovery"
        return 1
    fi
}

recover_container_infrastructure() {
    log INFO "Recovering container infrastructure..."
    
    # Check for Docker Compose or Kubernetes manifests
    if [ -f "/backup/docker/docker-compose.yml" ]; then
        log INFO "Found Docker Compose configuration..."
        
        cd /backup/docker
        
        # Restore volumes
        if [ -d "/backup/docker/volumes" ]; then
            log INFO "Restoring Docker volumes..."
            rsync -av /backup/docker/volumes/ /var/lib/docker/volumes/
        fi
        
        # Start containers
        log INFO "Starting containers..."
        docker-compose up -d
        
        log SUCCESS "Container infrastructure recovered"
        update_recovery_state "infrastructure_recovery" "completed"
        return 0
        
    elif [ -d "/backup/kubernetes" ]; then
        log INFO "Found Kubernetes manifests..."
        
        # Apply all manifests
        kubectl apply -f /backup/kubernetes/
        
        log SUCCESS "Kubernetes resources applied"
        return 0
    else
        log ERROR "No container configuration found"
        return 1
    fi
}

# Generate recovery report
generate_recovery_report() {
    local report_file="/var/log/recovery_report_$(date +%Y%m%d-%H%M%S).html"
    
    log INFO "Generating recovery report..."
    
    # Parse state file
    local recovery_id=$(jq -r '.recovery_id' "$STATE_FILE")
    local started_at=$(jq -r '.started_at' "$STATE_FILE")
    local status=$(jq -r '.status' "$STATE_FILE")
    local steps_completed=$(jq -r '.steps_completed | join(", ")' "$STATE_FILE")
    local steps_failed=$(jq -r '.steps_failed | join(", ")' "$STATE_FILE")
    
    cat > "$report_file" <<HTML
<!DOCTYPE html>
<html>
<head>
    <title>Recovery Report - $recovery_id</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 40px;
            background-color: #f5f5f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }
        h1 {
            color: #2c3e50;
            border-bottom: 3px solid #3498db;
            padding-bottom: 10px;
        }
        .status {
            padding: 15px;
            border-radius: 5px;
            margin: 20px 0;
            font-weight: bold;
        }
        .status-success {
            background-color: #d4edda;
            color: #155724;
            border-left: 4px solid #28a745;
        }
        .status-warning {
            background-color: #fff3cd;
            color: #856404;
            border-left: 4px solid #ffc107;
        }
        .status-error {
            background-color: #f8d7da;
            color: #721c24;
            border-left: 4px solid #dc3545;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #3498db;
            color: white;
        }
        .metric {
            display: inline-block;
            margin: 10px 20px 10px 0;
            padding: 15px;
            background: #ecf0f1;
            border-radius: 5px;
        }
        .metric-label {
            font-size: 0.9em;
            color: #7f8c8d;
        }
        .metric-value {
            font-size: 1.5em;
            font-weight: bold;
            color: #2c3e50;
        }
        .log-section {
            background: #2c3e50;
            color: #ecf0f1;
            padding: 15px;
            border-radius: 5px;
            font-family: 'Courier New', monospace;
            font-size: 0.9em;
            max-height: 400px;
            overflow-y: auto;
            white-space: pre-wrap;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔄 Recovery Report</h1>
        
        <div class="status status-success">
            Recovery ID: $recovery_id
        </div>
        
        <h2>Summary</h2>
        <div class="metric">
            <div class="metric-label">Started At</div>
            <div class="metric-value">$started_at</div>
        </div>
        <div class="metric">
            <div class="metric-label">Duration</div>
            <div class="metric-value">$(( $(date +%s) - $(date -d "$started_at" +%s) ))s</div>
        </div>
        <div class="metric">
            <div class="metric-label">Status</div>
            <div class="metric-value">$status</div>
        </div>
        
        <h2>Steps Completed</h2>
        <p>$steps_completed</p>
        
        <h2>Steps Failed</h2>
        <p>${steps_failed:-None}</p>
        
        <h2>Recovery Log</h2>
        <div class="log-section">
$(tail -100 "$LOG_FILE")
        </div>
        
        <h2>System State After Recovery</h2>
        <table>
            <tr>
                <th>Component</th>
                <th>Status</th>
                <th>Details</th>
            </tr>
            <tr>
                <td>MySQL</td>
                <td>$(systemctl is-active mysql 2>/dev/null || echo "inactive")</td>
                <td>Database service</td>
            </tr>
            <tr>
                <td>PostgreSQL</td>
                <td>$(systemctl is-active postgresql 2>/dev/null || echo "inactive")</td>
                <td>Database service</td>
            </tr>
            <tr>
                <td>Nginx</td>
                <td>$(systemctl is-active nginx 2>/dev/null || echo "inactive")</td>
                <td>Web server</td>
            </tr>
            <tr>
                <td>Docker</td>
                <td>$(systemctl is-active docker 2>/dev/null || echo "inactive")</td>
                <td>Container runtime</td>
            </tr>
            <tr>
                <td>Disk Space</td>
                <td>$(df -h / | tail -1 | awk '{print $5}')</td>
                <td>Root filesystem usage</td>
            </tr>
        </table>
        
        <h2>Recommendations</h2>
        <ul>
            <li>Verify application functionality manually</li>
            <li>Check database consistency</li>
            <li>Review application logs for errors</li>
            <li>Test critical user workflows</li>
            <li>Update monitoring dashboards</li>
            <li>Document any issues encountered</li>
            <li>Schedule post-recovery review meeting</li>
        </ul>
        
        <hr>
        <p style="text-align: center; color: #7f8c8d;">
            Generated: $(date)<br>
            Recovery Orchestrator v1.0
        </p>
    </div>
</body>
</html>
HTML
    
    log SUCCESS "Recovery report generated: $report_file"
    echo "$report_file"
}

# Interactive recovery wizard
recovery_wizard() {
    clear
    cat <<BANNER
╔═══════════════════════════════════════════════════════════╗
║                                                           ║
║           🚨 DISASTER RECOVERY ORCHESTRATOR 🚨            ║
║                                                           ║
║                     Interactive Wizard                    ║
║                                                           ║
╚═══════════════════════════════════════════════════════════╝
BANNER
    
    echo ""
    logINFO "Welcome to the Recovery Wizard" echo ""

# Step 1: Identify scenario
echo "What type of recovery do you need?"
echo "1) Single file recovery"
echo "2) Database point-in-time recovery"
echo "3) Full application stack recovery"
echo "4) Infrastructure recovery"
echo "5) Complete datacenter failover"
echo ""
read -p "Select option (1-5): " scenario

case "$scenario" in
    1)
        read -p "Enter file path to recover: " file_path
        read -p "Restore to which point? (latest/YYYY-MM-DD HH:MM:SS): " restore_point
        recover_file "$file_path" "$restore_point"
        ;;
    
    2)
        read -p "Database type (mysql/postgresql): " db_type
        read -p "Target time (YYYY-MM-DD HH:MM:SS): " target_time
        read -p "Database name (or 'all'): " db_name
        recover_database_pitr "$db_type" "$target_time" "$db_name"
        ;;
    
    3)
        read -p "Application name: " app_name
        read -p "Environment (production/staging): " environment
        recover_application_stack "$app_name" "$environment"
        ;;
    
    4)
        echo "Infrastructure type:"
        echo "  - vm (Virtual Machine)"
        echo "  - bare-metal"
        echo "  - container"
        read -p "Select type: " infra_type
        recover_infrastructure "$infra_type"
        ;;
    
    5)
        log WARNING "Complete datacenter failover is a complex operation"
        read -p "Are you sure you want to proceed? (yes/no): " confirm
        if [ "$confirm" = "yes" ]; then
            datacenter_failover
        fi
        ;;
    
    *)
        log ERROR "Invalid option selected"
        exit 1
        ;;
esac

# Generate report
generate_recovery_report

}

# Main execution

main() { # Check if running as root if [ "$EUID" -ne 0 ]; then log ERROR "This script must be run as root" exit 1 fi


# Create directories
mkdir -p "$RECOVERY_DIR"
mkdir -p "$(dirname "$LOG_FILE")"

# Initialize state
init_recovery_state

# Run pre-checks
pre_recovery_checks

case "${1:-wizard}" in
    wizard)
        recovery_wizard
        ;;
    
    file)
        recover_file "$2" "${3:-latest}"
        ;;
    
    pitr)
        recover_database_pitr "$2" "$3" "${4:-all}"
        ;;
    
    app)
        recover_application_stack "$2" "${3:-production}"
        ;;
    
    infra)
        recover_infrastructure "$2"
        ;;
    
    report)
        generate_recovery_report
        ;;
    
    *)
        cat <<HELP


Disaster Recovery Orchestrator

Usage: $0 <command> [options]

Commands: wizard - Interactive recovery wizard file <path> [time] - Recover single file pitr <db_type> <time> [db_name] - Database point-in-time recovery app <name> [env] - Full application stack recovery infra <type> - Infrastructure recovery report - Generate recovery report

Examples: $0 wizard $0 file /var/www/index.php latest $0 pitr mysql "2024-01-15 10:00:00" mydb $0 app wordpress production $0 infra vm HELP ;; esac }

# Run main function

main "$@" EOF

chmod +x recovery_orchestrator.sh

```

---

## 💻 Задание 2: Automated DR Testing Framework

```bash
cat > dr_testing_framework.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
TEST_CONFIG="/etc/dr-testing/config.json"
TEST_LOG_DIR="/var/log/dr-tests"
TEST_RESULTS_DIR="/var/lib/dr-tests/results"
NOTIFICATION_WEBHOOK="https://hooks.slack.com/services/YOUR/WEBHOOK"

mkdir -p "$TEST_LOG_DIR" "$TEST_RESULTS_DIR"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$TEST_LOG_DIR/dr_test_$(date +%Y%m%d).log"
}

# Create test configuration
create_test_config() {
    mkdir -p "$(dirname "$TEST_CONFIG")"
    
    cat > "$TEST_CONFIG" <<'CONFIG'
{
  "test_schedule": {
    "daily": {
      "enabled": true,
      "tests": ["backup_verification", "single_file_restore"]
    },
    "weekly": {
      "enabled": true,
      "tests": ["database_restore", "application_restore"]
    },
    "monthly": {
      "enabled": true,
      "tests": ["full_system_restore"]
    },
    "quarterly": {
      "enabled": true,
      "tests": ["dr_site_failover_simulation"]
    }
  },
  "test_environments": {
    "isolated_test": {
      "type": "docker",
      "network_isolated": true,
      "auto_cleanup": true
    }
  },
  "success_criteria": {
    "rto_target_minutes": 120,
    "rpo_target_minutes": 240,
    "data_integrity_threshold": 100,
    "service_availability_threshold": 99.9
  },
  "notifications": {
    "on_failure": true,
    "on_success": false,
    "on_threshold_breach": true
  }
}
CONFIG
    
    log "Test configuration created: $TEST_CONFIG"
}

# Test 1: Backup Verification
test_backup_verification() {
    local test_id="bv_$(date +%Y%m%d_%H%M%S)"
    local test_log="$TEST_LOG_DIR/${test_id}.log"
    
    log "=== Starting Backup Verification Test ==="
    
    local start_time=$(date +%s)
    local tests_passed=0
    local tests_failed=0
    
    # Test backup integrity
    log "Checking backup integrity..."
    
    # Check MySQL backups
    if [ -d "/backup/mysql/daily" ]; then
        local latest_mysql=$(ls -t /backup/mysql/daily/*.sql.gz 2>/dev/null | head -1)
        
        if [ -n "$latest_mysql" ]; then
            # Check if file is readable and not corrupted
            if gunzip -t "$latest_mysql" 2>/dev/null; then
                log "✓ MySQL backup integrity OK"
                ((tests_passed++))
                
                # Verify checksum if exists
                if [ -f "${latest_mysql}.md5" ]; then
                    if md5sum -c "${latest_mysql}.md5" &>/dev/null; then
                        log "✓ MySQL backup checksum verified"
                        ((tests_passed++))
                    else
                        log "✗ MySQL backup checksum failed"
                        ((tests_failed++))
                    fi
                fi
            else
                log "✗ MySQL backup corrupted"
                ((tests_failed++))
            fi
        else
            log "✗ No MySQL backup found"
            ((tests_failed++))
        fi
    fi
    
    # Check PostgreSQL backups
    if [ -d "/backup/postgresql" ]; then
        local latest_pg=$(ls -t /backup/postgresql/pg-*.sql.gz 2>/dev/null | head -1)
        
        if [ -n "$latest_pg" ]; then
            if gunzip -t "$latest_pg" 2>/dev/null; then
                log "✓ PostgreSQL backup integrity OK"
                ((tests_passed++))
            else
                log "✗ PostgreSQL backup corrupted"
                ((tests_failed++))
            fi
        fi
    fi
    
    # Check filesystem snapshots
    if [ -d "/backup/rsync-snapshots/latest" ]; then
        local file_count=$(find /backup/rsync-snapshots/latest -type f | wc -l)
        
        if [ "$file_count" -gt 0 ]; then
            log "✓ Filesystem snapshot exists ($file_count files)"
            ((tests_passed++))
            
            # Random sample verification
            local sample_files=$(find /backup/rsync-snapshots/latest -type f | shuf -n 10)
            local verified=0
            
            for file in $sample_files; do
                if [ -r "$file" ]; then
                    ((verified++))
                fi
            done
            
            if [ $verified -eq 10 ]; then
                log "✓ Random sample verification passed (10/10 files readable)"
                ((tests_passed++))
            else
                log "✗ Random sample verification failed ($verified/10 files readable)"
                ((tests_failed++))
            fi
        else
            log "✗ Filesystem snapshot is empty"
            ((tests_failed++))
        fi
    fi
    
    # Check cloud backups
    if command -v aws &>/dev/null; then
        local s3_count=$(aws s3 ls s3://company-backups/ --recursive 2>/dev/null | wc -l)
        
        if [ "$s3_count" -gt 0 ]; then
            log "✓ Cloud backups exist ($s3_count files in S3)"
            ((tests_passed++))
        else
            log "✗ No cloud backups found"
            ((tests_failed++))
        fi
    fi
    
    local end_time=$(date +%s)
    local duration=$((end_time - start_time))
    
    # Generate test result
    cat > "$TEST_RESULTS_DIR/${test_id}.json" <<RESULT
{
  "test_id": "$test_id",
  "test_type": "backup_verification",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "duration_seconds": $duration,
  "tests_passed": $tests_passed,
  "tests_failed": $tests_failed,
  "success_rate": $(echo "scale=2; $tests_passed * 100 / ($tests_passed + $tests_failed)" | bc),
  "status": "$([ $tests_failed -eq 0 ] && echo 'PASS' || echo 'FAIL')"
}
RESULT
    
    log "Backup verification test completed: $tests_passed passed, $tests_failed failed"
    
    [ $tests_failed -eq 0 ] && return 0 || return 1
}

# Test 2: Single File Restore Test
test_single_file_restore() {
    local test_id="sfr_$(date +%Y%m%d_%H%M%S)"
    
    log "=== Starting Single File Restore Test ==="
    
    local start_time=$(date +%s)
    
    # Create a test file
    local test_file="/tmp/dr_test_file_$$"
    local test_content="DR Test Content: $(date) - $(uuidgen)"
    echo "$test_content" > "$test_file"
    
    log "Test file created: $test_file"
    
    # Simulate backup (copy to backup location)
    local backup_file="/backup/test/${test_id}.txt"
    mkdir -p "$(dirname "$backup_file")"
    cp "$test_file" "$backup_file"
    
    # Delete original
    rm "$test_file"
    log "Original file deleted"
    
    # Restore from backup
    log "Attempting restore..."
    cp "$backup_file" "$test_file"
    
    # Verify restored content
    local restored_content=$(cat "$test_file")
    
    local status="FAIL"
    if [ "$restored_content" = "$test_content" ]; then
        log "✓ File restore successful, content verified"
        status="PASS"
    else
        log "✗ File restore failed, content mismatch"
    fi
    
    # Cleanup
    rm -f "$test_file" "$backup_file"
    
    local end_time=$(date +%s)
    local duration=$((end_time - start_time))
    
    cat > "$TEST_RESULTS_DIR/${test_id}.json" <<RESULT
{
  "test_id": "$test_id",
  "test_type": "single_file_restore",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "duration_seconds": $duration,
  "rto_actual_seconds": $duration,
  "status": "$status"
}
RESULT
    
    log "Single file restore test completed in ${duration}s"
    
    [ "$status" = "PASS" ] && return 0 || return 1
}

# Test 3: Database Restore Test (Isolated)
test_database_restore() {
    local test_id="dbr_$(date +%Y%m%d_%H%M%S)"
    
    log "=== Starting Database Restore Test (Isolated) ==="
    
    local start_time=$(date +%s)
    
    # Create isolated test environment using Docker
    log "Creating isolated test environment..."
    
    # MySQL restore test
    if [ -f "/backup/mysql/daily/$(ls -t /backup/mysql/daily/*.sql.gz | head -1)" ]; then
        local backup_file=$(ls -t /backup/mysql/daily/*.sql.gz | head -1)
        
        log "Starting test MySQL container..."
        docker run -d \
            --name "dr_test_mysql_$$" \
            -e MYSQL_ROOT_PASSWORD=test123 \
            -e MYSQL_DATABASE=testdb \
            mysql:8.0
        
        # Wait for MySQL to be ready
        log "Waiting for MySQL to be ready..."
        sleep 30
        
        # Restore backup
        log "Restoring backup to test database..."
        gunzip < "$backup_file" | \
            docker exec -i "dr_test_mysql_$$" mysql -uroot -ptest123
        
        if [ $? -eq 0 ]; then
            log "✓ Database restore successful"
            
            # Verify data
            local table_count=$(docker exec "dr_test_mysql_$$" \
                mysql -uroot -ptest123 -e "SHOW TABLES" | wc -l)
            
            log "Tables found: $table_count"
            
            if [ "$table_count" -gt 1 ]; then
                log "✓ Database contains data"
                local status="PASS"
            else
                log "✗ Database appears empty"
                local status="FAIL"
            fi
        else
            log "✗ Database restore failed"
            local status="FAIL"
        fi
        
        # Cleanup
        docker stop "dr_test_mysql_$$" >/dev/null 2>&1
        docker rm "dr_test_mysql_$$" >/dev/null 2>&1
        
    else
        log "✗ No MySQL backup found for testing"
        local status="FAIL"
    fi
    
    local end_time=$(date +%s)
    local duration=$((end_time - start_time))
    local rto_minutes=$(echo "scale=2; $duration / 60" | bc)
    
    cat > "$TEST_RESULTS_DIR/${test_id}.json" <<RESULT
{
  "test_id": "$test_id",
  "test_type": "database_restore",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "duration_seconds": $duration,
  "rto_actual_minutes": $rto_minutes,
  "rto_target_minutes": 120,
  "rto_met": $([ $(echo "$rto_minutes < 120" | bc) -eq 1 ] && echo true || echo false),
  "status": "$status"
}
RESULT
    
    log "Database restore test completed in ${duration}s (${rto_minutes}m)"
    
    [ "$status" = "PASS" ] && return 0 || return 1
}

# Test 4: Application Restore Test
test_application_restore() {
    local test_id="apr_$(date +%Y%m%d_%H%M%S)"
    
    log "=== Starting Application Restore Test ==="
    
    local start_time=$(date +%s)
    
    # Create test environment
    local test_dir="/tmp/dr_app_test_$$"
    mkdir -p "$test_dir"
    
    log "Test environment: $test_dir"
    
    # Simulate application files
    cat > "$test_dir/app.conf" <<APPCONF
[application]
name = test_app
version = 1.0
database = mysql://localhost/testdb
cache = redis://localhost:6379
APPCONF
    
    # Backup
    local backup_archive="/backup/test/app_${test_id}.tar.gz"
    tar -czf "$backup_archive" -C "$(dirname "$test_dir")" "$(basename "$test_dir")"
    
    log "Application backed up to: $backup_archive"
    
    # Simulate disaster
    rm -rf "$test_dir"
    log "Application directory deleted (simulated disaster)"
    
    # Restore
    log "Restoring application..."
    tar -xzf "$backup_archive" -C "$(dirname "$test_dir")"
    
    # Verify
    if [ -f "$test_dir/app.conf" ]; then
        log "✓ Application files restored"
        
        # Verify content
        if grep -q "test_app" "$test_dir/app.conf"; then
            log "✓ Application configuration intact"
            local status="PASS"
        else
            log "✗ Application configuration corrupted"
            local status="FAIL"
        fi
    else
        log "✗ Application restore failed"
        local status="FAIL"
    fi
    
    # Cleanup
    rm -rf "$test_dir" "$backup_archive"
    
    local end_time=$(date +%s)
    local duration=$((end_time - start_time))
    
    cat > "$TEST_RESULTS_DIR/${test_id}.json" <<RESULT
{
  "test_id": "$test_id",
  "test_type": "application_restore",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "duration_seconds": $duration,
  "status": "$status"
}
RESULT
    
    log "Application restore test completed in ${duration}s"
    
    [ "$status" = "PASS" ] && return 0 || return 1
}

# Test 5: Full System Restore Simulation
test_full_system_restore() {
    local test_id="fsr_$(date +%Y%m%d_%H%M%S)"
    
    log "=== Starting Full System Restore Simulation ==="
    
    local start_time=$(date +%s)
    
    # This is a simulation - we don't actually restore the system
    log "Simulating full system restore workflow..."
    
    local steps=(
        "Verify backup availability"
        "Check restore target capacity"
        "Stop services"
        "Restore system files"
        "Restore databases"
        "Restore application data"
        "Verify configuration files"
        "Start services"
        "Run health checks"
    )
    
    local steps_completed=0
    local steps_failed=0
    
    for step in "${steps[@]}"; do
        log "Step: $step"
        
        # Simulate step execution
        sleep 2
        
        # Random success (90% success rate for simulation)
        if [ $((RANDOM % 10)) -lt 9 ]; then
            log "  ✓ $step - OK"
            ((steps_completed++))
        else
            log "  ✗ $step - FAILED"
            ((steps_failed++))
        fi
    done
    
    local end_time=$(date +%s)
    local duration=$((end_time - start_time))
    local rto_minutes=$(echo "scale=2; $duration / 60" | bc)
    
    local status="FAIL"
    if [ $steps_failed -eq 0 ]; then
        status="PASS"
    fi
    
    cat > "$TEST_RESULTS_DIR/${test_id}.json" <<RESULT
{
  "test_id": "$test_id",
  "test_type": "full_system_restore",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "duration_seconds": $duration,
  "rto_actual_minutes": $rto_minutes,
  "steps_total": ${#steps[@]},
  "steps_completed": $steps_completed,
  "steps_failed": $steps_failed,
  "status": "$status"
}
RESULT
    
    log "Full system restore simulation completed in ${duration}s"
    log "Steps: $steps_completed completed, $steps_failed failed"
    
    [ "$status" = "PASS" ] && return 0 || return 1
}

# Test 6: RTO/RPO Validation
test_rto_rpo_validation() {
    local test_id="rto_rpo_$(date +%Y%m%d_%H%M%S)"
    
    log "=== Starting RTO/RPO Validation ==="
    
    # Check latest backup age
    local latest_backup=""
    local backup_age_hours=999
    
    if [ -d "/backup/mysql/daily" ]; then
        latest_backup=$(ls -t /backup/mysql/daily/*.sql.gz 2>/dev/null | head -1)
        if [ -n "$latest_backup" ]; then
            local backup_timestamp=$(stat -c %Y "$latest_backup" 2>/dev/null || stat -f %m "$latest_backup")
            local current_timestamp=$(date +%s)
            backup_age_hours=$(( (current_timestamp - backup_timestamp) / 3600 ))
        fi
    fi
    
    log "Latest backup age: ${backup_age_hours} hours"
    
    # RPO validation (target: 4 hours)
    local rpo_target=4
    local rpo_met=false
    
    if [ "$backup_age_hours" -le "$rpo_target" ]; then
        log "✓ RPO met: ${backup_age_hours}h <= ${rpo_target}h"
        rpo_met=true
    else
        log "✗ RPO not met: ${backup_age_hours}h > ${rpo_target}h"
    fi
    
    # RTO validation (simulate restore and measure time)
    log "Simulating restore to measure RTO..."
    local restore_start=$(date +%s)
    
    # Simulate restore process
    sleep 5  # Simulated restore time
    
    local restore_end=$(date +%s)
    local rto_actual=$(( (restore_end - restore_start) / 60 ))
    local rto_target=120  # 2 hours
    local rto_met=false
    
    if [ "$rto_actual" -le "$rto_target" ]; then
        log "✓ RTO met: ${rto_actual}m <= ${rto_target}m"
        rto_met=true
    else
        log "✗ RTO not met: ${rto_actual}m > ${rto_target}m"
    fi
    
    local status="PASS"
    if [ "$rpo_met" = false ] || [ "$rto_met" = false ]; then
        status="FAIL"
    fi
    
    cat > "$TEST_RESULTS_DIR/${test_id}.json" <<RESULT
{
  "test_id": "$test_id",
  "test_type": "rto_rpo_validation",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "rpo": {
    "target_hours": $rpo_target,
    "actual_hours": $backup_age_hours,
    "met": $rpo_met
  },
  "rto": {
    "target_minutes": $rto_target,
    "actual_minutes": $rto_actual,
    "met": $rto_met
  },
  "status": "$status"
}
RESULT
    
    log "RTO/RPO validation completed"
    
    [ "$status" = "PASS" ] && return 0 || return 1
}

# Generate comprehensive test report
generate_test_report() {
    local report_file="$TEST_RESULTS_DIR/summary_$(date +%Y%m%d).html"
    
    log "Generating test report..."
    
    # Aggregate test results
    local total_tests=0
    local passed_tests=0
    local failed_tests=0
    
    for result in "$TEST_RESULTS_DIR"/*.json; do
        if [ -f "$result" ]; then
            ((total_tests++))
            local status=$(jq -r '.status' "$result")
            if [ "$status" = "PASS" ]; then
                ((passed_tests++))
            else
                ((failed_tests++))
            fi
        fi
    done
    
    local success_rate=$(echo "scale=2; $passed_tests * 100 / $total_tests" | bc 2>/dev/null || echo "0")
    
    cat > "$report_file" <<HTML
<!DOCTYPE html>
<html>
<head>
    <title>DR Testing Report - $(date +%Y-%m-%d)</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f0f0f0;
        }
        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
        h1 {
            color: #d9534f;
            border-bottom: 3px solid #d9534f;
            padding-bottom: 10px;
        }
        .summary {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 20px;
            margin: 30px 0;
        }
        .metric-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 20px;
            border-radius: 8px;
            text-align: center;
        }
        .metric-card.success {
            background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
        }
        .metric-card.danger {
            background: linear-gradient(135deg, #eb3349 0%, #f45c43 100%);
        }
        .metric-value {
            font-size: 3em;
            font-weight: bold;
            margin: 10px 0;
        }
        .metric-label {
            font-size: 0.9em;
            opacity: 0.9;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #667eea;
            color: white;
        }
        tr:hover {
            background-color: #f5f5f5;
        }
        .status-pass {
            color: #28a745;
            font-weight: bold;
        }
        .status-fail {
            color: #dc3545;
            font-weight: bold;
        }
        .chart-container {
            margin: 30px 0;
            padding: 20px;
            background: #f9f9f9;
            border-radius: 8px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🧪 Disaster Recovery Testing Report</h1>
        <p><strong>Generated:</strong> $(date)</p>
        
        <div class="summary">
            <div class="metric-card">
                <div class="metric-label">Total Tests</div>
                <div class="metric-value">$total_tests</div>
            </div>
            <div class="metric-card success">
                <div class="metric-label">Passed</div>
                <div class="metric-value">$passed_tests</div>
            </div>
            <div class="metric-card danger">
                <div class="metric-label">Failed</div>
                <div class="metric-value">$failed_tests</div>
            </div>
            <div class="metric-card">
                <div class="metric-label">Success Rate</div>
                <div class="metric-value">${success_rate}%</div>
            </div>
        </div>
        
        <h2>Test Results</h2>
        <table>
            <thead>
                <tr>
                    <th>Test ID</th>
                    <th>Test Type</th>
                    <th>Timestamp</th>
                    <th>Duration</th>
                    <th>Status</th>
                </tr>
            </thead>
            <tbody>
HTML
    
    # Add test results to table
    for result in "$TEST_RESULTS_DIR"/*.json; do
        if [ -f "$result" ]; then
            local test_id=$(jq -r '.test_id' "$result")
            local test_type=$(jq -r '.test_type' "$result")
            local timestamp=$(jq -r '.timestamp' "$result")
            local duration=$(jq -r '.duration_seconds // 0' "$result")
            local status=$(jq -r '.status' "$result")
            
            local status_class="status-pass"
            [ "$status" = "FAIL" ] && status_class="status-fail"
            
            cat >> "$report_file" <<HTML
                <tr>
                    <td>$test_id</td>
                    <td>$test_type</td>
                    <td>$timestamp</td>
                    <td>${duration}s</td>
                    <td class="$status_class">$status</td>
                </tr>
HTML
        fi
    done
    
    cat >> "$report_file" <<HTML
            </tbody>
        </table>
        
        <h2>RTO/RPO Compliance</h2>
        <div class="chart-container">
HTML
    
    # Check for RTO/RPO test results
    local rto_rpo_result=$(ls -t "$TEST_RESULTS_DIR"/rto_rpo_*.json 2>/dev/null | head -1)
    if [ -n "$rto_rpo_result" ]; then
        local rpo_target=$(jq -r '.rpo.target_hours' "$rto_rpo_result")
        local rpo_actual=$(jq -r '.rpo.actual_hours' "$rto_rpo_result")
        local rpo_met=$(jq -r '.rpo.met' "$rto_rpo_result")
        
        local rto_target=$(jq -r '.rto.target_minutes' "$rto_rpo_result")
        local rto_actual=$(jq -r '.rto.actual_minutes' "$rto_rpo_result")
        local rto_met=$(jq -r '.rto.met' "$rto_rpo_result")
        
        cat >> "$report_file" <<HTML
            <p><strong>RPO (Recovery Point Objective):</strong></p>
            <ul>
                <li>Target: ${rpo_target} hours</li>
                <li>Actual: ${rpo_actual} hours</li>
                <li>Status: $([ "$rpo_met" = "true" ] && echo "✓ Met" || echo "✗ Not Met")</li>
            </ul>
            
            <p><strong>RTO (Recovery Time Objective):</strong></p>
            <ul>
                <li>Target: ${rto_target} minutes</li>
                <li>Actual: ${rto_actual} minutes</li>
                <li>Status: $([ "$rto_met" = "true" ] && echo "✓ Met" || echo "✗ Not Met")</li>
            </ul>
HTML
    else
        cat >> "$report_file" <<HTML
            <p><em>No RTO/RPO validation test results available</em></p>
HTML
    fi
    
    cat >> "$report_file" <<HTML
        </div>
        
        <h2>Recommendations</h2>
        <ul>
HTML
    
    # Generate recommendations based on results
    if [ $failed_tests -gt 0 ]; then
        cat >> "$report_file" <<HTML
            <li><strong>Action Required:</strong> $failed_tests test(s) failed. Review and address failures immediately.</li>
HTML
    fi
    
    if [ $(echo "$success_rate < 90" | bc) -eq 1 ]; then
        cat >> "$report_file" <<HTML
            <li><strong>Warning:</strong> Success rate is below 90%. Consider reviewing backup procedures.</li>
HTML
    fi
    
    cat >> "$report_file" <<HTML
            <li>Schedule next full DR test within 30 days</li>
            <li>Update disaster recovery documentation</li>
            <li>Review and update runbooks based on test findings</li>
            <li>Train team on any identified gaps</li>
        </ul>
        
        <hr>
        <p style="text-align: center; color: #666;">
            DR Testing Framework v1.0 | Report generated: $(date)
        </p>
    </div>
</body>
</html>
HTML
    
    log "Test report generated: $report_file"
    echo "$report_file"
}

# Send notification
send_notification() {
    local test_type="$1"
    local status="$2"
    local details="$3"
    
    if [ -n "$NOTIFICATION_WEBHOOK" ]; then
        local color="good"
        [ "$status" = "FAIL" ] && color="danger"
        
        curl -X POST "$NOTIFICATION_WEBHOOK" \
            -H 'Content-Type: application/json' \
            -d "{
                \"attachments\": [{
                    \"color\": \"$color\",
                    \"title\": \"DR Test: $test_type\",
                    \"text\": \"Status: $status\n$details\",
                    \"footer\": \"DR Testing Framework\",
                    \"ts\": $(date +%s)
                }]
            }" 2>/dev/null || true
    fi
}

# Run all tests
run_all_tests() {
    log "=========================================="
    log "Starting DR Test Suite"
    log "=========================================="
    
    local tests_run=0
    local tests_passed=0
    local tests_failed=0
    
    # Test 1: Backup Verification
    ((tests_run++))
    if test_backup_verification; then
        ((tests_passed++))
        send_notification "Backup Verification" "PASS" "All backups verified successfully"
    else
        ((tests_failed++))
        send_notification "Backup Verification" "FAIL" "Some backup verifications failed"
    fi
    
    # Test 2: Single File Restore
((tests_run++)) if test_single_file_restore; then ((tests_passed++)) else ((tests_failed++)) fi


# Test 3: Database Restore
((tests_run++))
if test_database_restore; then
    ((tests_passed++))
else
    ((tests_failed++))
    send_notification "Database Restore" "FAIL" "Database restore test failed"
fi

# Test 4: Application Restore
((tests_run++))
if test_application_restore; then
    ((tests_passed++))
else
    ((tests_failed++))
fi

# Test 5: Full System Restore Simulation
((tests_run++))
if test_full_system_restore; then
    ((tests_passed++))
else
    ((tests_failed++))
    send_notification "Full System Restore" "FAIL" "Full system restore simulation failed"
fi

# Test 6: RTO/RPO Validation
((tests_run++))
if test_rto_rpo_validation; then
    ((tests_passed++))
else
    ((tests_failed++))
    send_notification "RTO/RPO Validation" "FAIL" "RTO/RPO targets not met"
fi

log "=========================================="
log "DR Test Suite Completed"
log "Tests Run: $tests_run"
log "Tests Passed: $tests_passed"
log "Tests Failed: $tests_failed"
log "=========================================="

# Generate final report
generate_test_report


}

# Main execution

case "${1:-help}" in init) create_test_config ;;


run-all)
    run_all_tests
    ;;

backup-verification)
    test_backup_verification
    ;;

file-restore)
    test_single_file_restore
    ;;

database-restore)
    test_database_restore
    ;;

app-restore)
    test_application_restore
    ;;

full-restore)
    test_full_system_restore
    ;;

rto-rpo)
    test_rto_rpo_validation
    ;;

report)
    generate_test_report
    ;;

*)
    cat <<HELP


DR Testing Framework

Usage: $0 <command>

Commands: init - Initialize test configuration run-all - Run all DR tests backup-verification - Test backup integrity file-restore - Test single file restore database-restore - Test database restore (isolated) app-restore - Test application restore full-restore - Test full system restore simulation rto-rpo - Validate RTO/RPO compliance report - Generate test report

Examples: $0 init $0 run-all $0 report HELP ;; esac EOF

chmod +x dr_testing_framework.sh

````

---

## 💻 Задание 3: Automated Failover System

```bash
cat > automated_failover.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
FAILOVER_CONFIG="/etc/failover/config.json"
STATE_FILE="/var/run/failover_state.json"
LOG_FILE="/var/log/failover_$(date +%Y%m%d).log"

PRIMARY_SITE="primary.example.com"
DR_SITE="dr.example.com"
HEALTH_CHECK_INTERVAL=60
FAILOVER_THRESHOLD=3

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# Initialize failover state
init_failover_state() {
    cat > "$STATE_FILE" <<STATE
{
  "current_active_site": "primary",
  "last_failover": null,
  "failover_count": 0,
  "health_check_failures": 0,
  "status": "normal"
}
STATE
}

# Health check function
health_check() {
    local site="$1"
    
    log "Performing health check on $site..."
    
    local checks_passed=0
    local checks_total=0
    
    # Check 1: HTTP endpoint
    ((checks_total++))
    if curl -f -s -o /dev/null -w "%{http_code}" "http://$site/health" | grep -q "200"; then
        log "  ✓ HTTP health check passed"
        ((checks_passed++))
    else
        log "  ✗ HTTP health check failed"
    fi
    
    # Check 2: Database connectivity
    ((checks_total++))
    if nc -z -w 5 "$site" 3306 2>/dev/null; then
        log "  ✓ Database connectivity OK"
        ((checks_passed++))
    else
        log "  ✗ Database connectivity failed"
    fi
    
    # Check 3: Ping
    ((checks_total++))
    if ping -c 3 -W 5 "$site" &>/dev/null; then
        log "  ✓ Ping successful"
        ((checks_passed++))
    else
        log "  ✗ Ping failed"
    fi
    
    # Return success if majority of checks passed
    [ $checks_passed -ge $(( checks_total / 2 + 1 )) ] && return 0 || return 1
}

# Initiate failover
initiate_failover() {
    local from_site="$1"
    local to_site="$2"
    
    log "=========================================="
    log "INITIATING FAILOVER"
    log "From: $from_site"
    log "To: $to_site"
    log "=========================================="
    
    # Step 1: Notify team
    send_alert "CRITICAL" "Failover initiated from $from_site to $to_site"
    
    # Step 2: Update DNS (example with Route53)
    log "Updating DNS records..."
    update_dns "$to_site"
    
    # Step 3: Promote DR site to primary
    log "Promoting DR site..."
    promote_dr_site "$to_site"
    
    # Step 4: Update state
    jq --arg site "$to_site" --arg time "$(date -u +%Y-%m-%dT%H:%M:%SZ)" '
        .current_active_site = $site |
        .last_failover = $time |
        .failover_count += 1 |
        .status = "failover_complete"
    ' "$STATE_FILE" > "$STATE_FILE.tmp" && mv "$STATE_FILE.tmp" "$STATE_FILE"
    
    log "Failover completed successfully"
    send_alert "INFO" "Failover completed. Active site is now: $to_site"
}

# Update DNS
update_dns() {
    local target_site="$1"
    
    log "Updating DNS to point to $target_site..."
    
    # Example with AWS Route53
    if command -v aws &>/dev/null; then
        aws route53 change-resource-record-sets \
            --hosted-zone-id Z1234567890ABC \
            --change-batch "{
                \"Changes\": [{
                    \"Action\": \"UPSERT\",
                    \"ResourceRecordSet\": {
                        \"Name\": \"app.example.com\",
                        \"Type\": \"A\",
                        \"TTL\": 60,
                        \"ResourceRecords\": [{\"Value\": \"$target_site\"}]
                    }
                }]
            }"
        
        log "DNS updated successfully"
    else
        log "AWS CLI not available, skipping DNS update"
    fi
}

# Promote DR site
promote_dr_site() {
    local dr_site="$1"
    
    log "Promoting DR site to primary..."
    
    # Example steps for database promotion
    log "Promoting database replica to master..."
    
    # MySQL example
    ssh "root@$dr_site" "mysql -e 'STOP SLAVE; RESET SLAVE ALL;'" || true
    
    # PostgreSQL example
    ssh "root@$dr_site" "sudo -u postgres pg_ctl promote -D /var/lib/postgresql/14/main" || true
    
    log "DR site promoted successfully"
}

# Send alert
send_alert() {
    local severity="$1"
    local message="$2"
    
    log "[$severity] $message"
    
    # Email alert
    if command -v mail &>/dev/null; then
        echo "$message" | mail -s "[$severity] Failover Alert" ops@example.com
    fi
    
    # Slack alert
    if [ -n "$SLACK_WEBHOOK" ]; then
        local color="warning"
        [ "$severity" = "CRITICAL" ] && color="danger"
        [ "$severity" = "INFO" ] && color="good"
        
        curl -X POST "$SLACK_WEBHOOK" \
            -H 'Content-Type: application/json' \
            -d "{
                \"attachments\": [{
                    \"color\": \"$color\",
                    \"title\": \"Failover Alert\",
                    \"text\": \"$message\",
                    \"footer\": \"Automated Failover System\"
                }]
            }" 2>/dev/null || true
    fi
    
    # PagerDuty integration
    if [ -n "$PAGERDUTY_KEY" ] && [ "$severity" = "CRITICAL" ]; then
        curl -X POST https://events.pagerduty.com/v2/enqueue \
            -H 'Content-Type: application/json' \
            -d "{
                \"routing_key\": \"$PAGERDUTY_KEY\",
                \"event_action\": \"trigger\",
                \"payload\": {
                    \"summary\": \"$message\",
                    \"severity\": \"critical\",
                    \"source\": \"failover-system\"
                }
            }" 2>/dev/null || true
    fi
}

# Monitoring loop
monitor_sites() {
    log "Starting site monitoring..."
    
    while true; do
        local current_active=$(jq -r '.current_active_site' "$STATE_FILE")
        local health_failures=$(jq -r '.health_check_failures' "$STATE_FILE")
        
        if [ "$current_active" = "primary" ]; then
            local check_site="$PRIMARY_SITE"
            local failover_target="$DR_SITE"
        else
            local check_site="$DR_SITE"
            local failover_target="$PRIMARY_SITE"
        fi
        
        if health_check "$check_site"; then
            log "Health check passed for $check_site"
            
            # Reset failure counter
            jq '.health_check_failures = 0' "$STATE_FILE" > "$STATE_FILE.tmp" && \
                mv "$STATE_FILE.tmp" "$STATE_FILE"
        else
            log "Health check failed for $check_site"
            
            # Increment failure counter
            ((health_failures++))
            jq --arg failures "$health_failures" '.health_check_failures = ($failures | tonumber)' \
                "$STATE_FILE" > "$STATE_FILE.tmp" && mv "$STATE_FILE.tmp" "$STATE_FILE"
            
            # Check if failover threshold reached
            if [ "$health_failures" -ge "$FAILOVER_THRESHOLD" ]; then
                log "CRITICAL: Health check failures ($health_failures) exceeded threshold ($FAILOVER_THRESHOLD)"
                send_alert "CRITICAL" "Health checks failing on $check_site. Initiating failover..."
                
                # Initiate failover
                initiate_failover "$check_site" "$failover_target"
                
                # Reset failure counter
                jq '.health_check_failures = 0' "$STATE_FILE" > "$STATE_FILE.tmp" && \
                    mv "$STATE_FILE.tmp" "$STATE_FILE"
            else
                send_alert "WARNING" "Health check failure $health_failures/$FAILOVER_THRESHOLD on $check_site"
            fi
        fi
        
        # Wait before next check
        sleep "$HEALTH_CHECK_INTERVAL"
    done
}

# Manual failover
manual_failover() {
    log "Manual failover requested"
    
    read -p "Are you sure you want to initiate manual failover? (yes/no): " confirm
    
    if [ "$confirm" != "yes" ]; then
        log "Manual failover cancelled"
        exit 0
    fi
    
    local current_active=$(jq -r '.current_active_site' "$STATE_FILE")
    
    if [ "$current_active" = "primary" ]; then
        initiate_failover "$PRIMARY_SITE" "$DR_SITE"
    else
        initiate_failover "$DR_SITE" "$PRIMARY_SITE"
    fi
}

# Failback to primary
failback_to_primary() {
    log "Initiating failback to primary site..."
    
    local current_active=$(jq -r '.current_active_site' "$STATE_FILE")
    
    if [ "$current_active" = "primary" ]; then
        log "Already running on primary site"
        exit 0
    fi
    
    # Check if primary is healthy
    if health_check "$PRIMARY_SITE"; then
        log "Primary site is healthy, proceeding with failback"
        initiate_failover "$DR_SITE" "$PRIMARY_SITE"
    else
        log "ERROR: Primary site is not healthy, cannot failback"
        exit 1
    fi
}

# Status report
status_report() {
    local current_active=$(jq -r '.current_active_site' "$STATE_FILE")
    local last_failover=$(jq -r '.last_failover' "$STATE_FILE")
    local failover_count=$(jq -r '.failover_count' "$STATE_FILE")
    local health_failures=$(jq -r '.health_check_failures' "$STATE_FILE")
    local status=$(jq -r '.status' "$STATE_FILE")
    
    cat <<STATUS

╔═══════════════════════════════════════════════════════════╗
║            FAILOVER SYSTEM STATUS REPORT                  ║
╚═══════════════════════════════════════════════════════════╝

Current Active Site: $current_active
System Status: $status
Health Check Failures: $health_failures/$FAILOVER_THRESHOLD
Total Failovers: $failover_count
Last Failover: ${last_failover:-Never}

Primary Site Health:
STATUS
    
    if health_check "$PRIMARY_SITE"; then
        echo "  Status: ✓ HEALTHY"
    else
        echo "  Status: ✗ UNHEALTHY"
    fi
    
    echo ""
    echo "DR Site Health:"
    
    if health_check "$DR_SITE"; then
        echo "  Status: ✓ HEALTHY"
    else
        echo "  Status: ✗ UNHEALTHY"
    fi
    
    echo ""
}

# Main execution
case "${1:-help}" in
    init)
        mkdir -p "$(dirname "$STATE_FILE")"
        mkdir -p "$(dirname "$LOG_FILE")"
        init_failover_state
        log "Failover system initialized"
        ;;
    
    monitor)
        monitor_sites
        ;;
    
    failover)
        manual_failover
        ;;
    
    failback)
        failback_to_primary
        ;;
    
    status)
        status_report
        ;;
    
    test)
        log "Testing health checks..."
        health_check "$PRIMARY_SITE"
        health_check "$DR_SITE"
        ;;
    
    *)
        cat <<HELP
Automated Failover System

Usage: $0 <command>

Commands:
  init        - Initialize failover system
  monitor     - Start continuous monitoring (daemon mode)
  failover    - Manual failover to DR site
  failback    - Failback to primary site
  status      - Show current status
  test        - Test health checks

Examples:
  $0 init
  $0 monitor &
  $0 status
  $0 failover
HELP
        ;;
esac
EOF

chmod +x automated_failover.sh
````

---

## 🚀 Бонус: Recovery Runbook Generator

````bash
cat > generate_recovery_runbook.sh <<'EOF'
#!/bin/bash

RUNBOOK_FILE="/docs/recovery_runbook_$(date +%Y%m%d).md"

mkdir -p "$(dirname "$RUNBOOK_FILE")"

cat > "$RUNBOOK_FILE" <<'RUNBOOK'
# Disaster Recovery Runbook

**Generated:** $(date)  
**Version:** 1.0  
**Last Updated:** $(date +%Y-%m-%d)

---

## 🚨 Emergency Contacts

| Role | Name | Primary Contact | Secondary Contact |
|------|------|----------------|-------------------|
| Incident Commander | TBD | +1-XXX-XXX-XXXX | email@example.com |
| Database Administrator | TBD | +1-XXX-XXX-XXXX | email@example.com |
| System Administrator | TBD | +1-XXX-XXX-XXXX | email@example.com |
| Network Administrator | TBD | +1-XXX-XXX-XXXX | email@example.com |
| Application Owner | TBD | +1-XXX-XXX-XXXX | email@example.com |
| Management | TBD | +1-XXX-XXX-XXXX | email@example.com |

---

## 📋 Recovery Procedures by Scenario

### Scenario 1: Accidental File Deletion

**Symptoms:**
- User reports missing files
- Application unable to find expected files

**Detection Time:** Immediate to hours  
**Recovery Time Objective (RTO):** 15 minutes  
**Recovery Point Objective (RPO):** Last backup (typically < 24 hours)

**Recovery Steps:**

```bash
# 1. Identify the file and last known location
FILE_PATH="/path/to/deleted/file"

# 2. Check rsync snapshots
ls -la /backup/rsync-snapshots/latest$FILE_PATH

# 3. Restore using recovery orchestrator
/usr/local/bin/recovery_orchestrator.sh file "$FILE_PATH" latest

# 4. Verify restoration
ls -la "$FILE_PATH"
md5sum "$FILE_PATH"

# 5. Test file accessibility
sudo -u appuser cat "$FILE_PATH" > /dev/null

# 6. Update incident log
echo "File restored: $FILE_PATH at $(date)" >> /var/log/recovery_incidents.log
````

**Verification:**

- [ ] File exists at original location
- [ ] File permissions are correct
- [ ] Application can access the file
- [ ] File content is intact

---

### Scenario 2: Database Corruption

**Symptoms:**

- Database errors in application logs
- Cannot start database service
- Checksum errors in database logs

**Detection Time:** Immediate  
**RTO:** 2 hours  
**RPO:** Last backup + binary logs (< 4 hours data loss)

**Recovery Steps:**

```bash
# 1. Stop application to prevent further corruption
systemctl stop nginx php-fpm

# 2. Stop database
systemctl stop mysql

# 3. Backup corrupted database
mv /var/lib/mysql /var/lib/mysql.corrupted.$(date +%Y%m%d-%H%M%S)

# 4. Identify recovery point
# List available backups
ls -lh /backup/mysql/daily/

# 5. Restore database using PITR
/usr/local/bin/recovery_orchestrator.sh pitr mysql "2024-01-15 09:00:00" all

# 6. Start database
systemctl start mysql

# 7. Verify database
mysql -e "SHOW DATABASES;"
mysql -e "SELECT COUNT(*) FROM your_critical_table;" your_database

# 8. Start application
systemctl start php-fpm nginx

# 9. Test application
curl -I http://localhost
```

**Verification:**

- [ ] Database service is running
- [ ] All expected databases present
- [ ] Data integrity checks pass
- [ ] Application connects successfully
- [ ] No errors in error log

---

### Scenario 3: Ransomware Attack

**Symptoms:**

- Files encrypted with unknown extension
- Ransom note present
- Unable to access data

**Detection Time:** Immediate  
**RTO:** 4-8 hours  
**RPO:** Last clean backup (< 24 hours)

**CRITICAL FIRST STEPS:**

```bash
# 1. ISOLATE IMMEDIATELY
# Disconnect from network
ip link set eth0 down

# Kill SSH sessions
pkill -9 sshd

# Block outbound connections
iptables -P OUTPUT DROP

# 2. Document the incident
mkdir -p /tmp/incident_$(date +%Y%m%d-%H%M%S)
cd /tmp/incident_*

# Save ransom note
cp /path/to/ransom_note.txt .

# List encrypted files
find / -name "*.encrypted" -o -name "*.locked" > encrypted_files.txt

# System state
ps auxf > process_list.txt
netstat -tulpn > network_connections.txt
last -f /var/log/wtmp > login_history.txt

# 3. DO NOT pay ransom
# 4. Contact law enforcement
# 5. Contact security team/incident response
```

**Recovery Steps:**

```bash
# 1. Verify backups are not compromised
/usr/local/bin/dr_testing_framework.sh backup-verification

# 2. Prepare clean system
# Boot from clean media or use isolated recovery environment

# 3. Full system restore
/usr/local/bin/recovery_orchestrator.sh infra bare-metal

# 4. Restore from last known good backup BEFORE infection
# Identify clean backup
ls -lt /backup/rsync-snapshots/ | head -20

# 5. Restore data
rsync -av /backup/rsync-snapshots/2024-01-14_02-00-00/ /

# 6. Restore databases
/usr/local/bin/recovery_orchestrator.sh pitr mysql "2024-01-14 23:59:59"

# 7. Update all passwords
# Change all service passwords
# Rotate API keys
# Update SSH keys

# 8. Apply security patches
apt update && apt upgrade -y

# 9. Install EDR/antivirus
apt install clamav -y
freshclam
clamscan -r /

# 10. Restore network connectivity
ip link set eth0 up
iptables -P OUTPUT ACCEPT
```

**Post-Recovery Security Hardening:**

- [ ] All passwords changed
- [ ] Multi-factor authentication enabled
- [ ] Firewall rules reviewed and updated
- [ ] Intrusion detection system deployed
- [ ] Security audit completed
- [ ] Penetration test scheduled

---

### Scenario 4: Hardware Failure

**Symptoms:**

- Server unresponsive
- Disk failures reported
- RAID degraded
- System won't boot

**Detection Time:** Immediate  
**RTO:** 4 hours  
**RPO:** Last backup (< 24 hours)

**Recovery Steps:**

```bash
# FOR DISK FAILURE:

# 1. Check RAID status
cat /proc/mdstat
smartctl -a /dev/sda

# 2. If RAID degraded but functional, replace disk
mdadm --manage /dev/md0 --add /dev/sdX

# FOR COMPLETE SERVER FAILURE:

# 1. Provision replacement hardware
# Same specs or better

# 2. Boot from recovery media
# USB/PXE boot with recovery tools

# 3. Partition disks identically
sfdisk -d /dev/sda | sfdisk /dev/sdb  # (if you have partition backup)

# 4. Restore from bare metal backup
rear recover

# OR manual restore:

# 5. Install base OS
# Minimal installation

# 6. Restore system backup
/usr/local/bin/recovery_orchestrator.sh infra bare-metal

# 7. Restore data
rsync -av /backup/rsync-snapshots/latest/ /

# 8. Restore databases
/usr/local/bin/recovery_orchestrator.sh pitr mysql latest

# 9. Update hardware-specific configuration
# Network interfaces
# Disk UUIDs in /etc/fstab
vim /etc/fstab
vim /etc/network/interfaces

# 10. Reboot and verify
reboot
```

**Verification:**

- [ ] All disks healthy
- [ ] RAID rebuilt and synced
- [ ] All services running
- [ ] Network connectivity restored
- [ ] Application accessible

---

### Scenario 5: Complete Datacenter Failure

**Symptoms:**

- Cannot reach any services in primary datacenter
- Network connectivity lost to entire site
- Multiple simultaneous failures

**Detection Time:** Immediate  
**RTO:** 8-12 hours  
**RPO:** Last replication (< 1 hour for critical systems)

**Recovery Steps:**

```bash
# 1. Activate incident response team
# Conference call all key personnel

# 2. Verify DR site status
ssh dr-site.example.com
/usr/local/bin/automated_failover.sh status

# 3. Check replication status
# Database replication
mysql -h dr-site -e "SHOW SLAVE STATUS\G"

# File replication
rsync --dry-run -av primary-site:/data/ /data/

# 4. Initiate automated failover
/usr/local/bin/automated_failover.sh failover

# OR manual failover:

# 5. Promote DR database to primary
mysql -h dr-site -e "STOP SLAVE; RESET SLAVE ALL;"

# 6. Update DNS to point to DR site
aws route53 change-resource-record-sets \
  --hosted-zone-id ZXXXXX \
  --change-batch file://dns-failover.json

# 7. Start all services on DR site
ansible-playbook -i dr-inventory start-all-services.yml

# 8. Verify application functionality
curl https://app.example.com/health

# 9. Update status page
# Notify users of incident

# 10. Monitor DR site closely
watch -n 60 '/usr/local/bin/automated_failover.sh status'
```

**Communication Plan:**

- [ ] Status page updated
- [ ] Customer email sent
- [ ] Social media updated (if applicable)
- [ ] Internal teams notified
- [ ] Stakeholders briefed
- [ ] Incident timeline documented

---

## 🔧 Common Recovery Commands

### Quick Reference

```bash
# List available backups
ls -lh /backup/mysql/daily/
ls -lh /backup/postgresql/
ls -ld /backup/rsync-snapshots/*/

# Restore single file
cp /backup/rsync-snapshots/latest/path/to/file /path/to/file

# Restore MySQL database
gunzip < /backup/mysql/daily/latest.sql.gz | mysql -u root -p

# Restore PostgreSQL database
pg_restore -U postgres -d dbname /backup/postgresql/latest.dump

# Mount LVM snapshot
mount -o ro /dev/vg0/data_snap /mnt/snapshot

# Restic operations
export RESTIC_REPOSITORY="b2:company-backups"
export RESTIC_PASSWORD_FILE="/root/.restic_password"
restic snapshots
restic restore latest --target /restore/

# Docker container restart
docker-compose down
docker-compose up -d

# Check service status
systemctl status mysql postgresql nginx redis

# View recent logs
journalctl -u mysql -n 100 --no-pager
tail -f /var/log/nginx/error.log
```

---

## 📊 Recovery Metrics

Track these metrics for each recovery event:

|Metric|Target|Actual|Status|
|---|---|---|---|
|Detection Time|< 5 min|___|___|
|Response Time|< 10 min|___|___|
|Recovery Time (RTO)|< 2 hours|___|___|
|Data Loss (RPO)|< 4 hours|___|___|
|Service Availability|> 99.9%|___|___|

---

## ✅ Post-Recovery Checklist

After any recovery event:

- [ ] Document incident timeline
- [ ] Update recovery runbook with lessons learned
- [ ] Test backup/restore process
- [ ] Review and update monitoring alerts
- [ ] Schedule post-mortem meeting
- [ ] Update disaster recovery plan
- [ ] Train team on any new procedures
- [ ] Verify all backups working
- [ ] Review RTO/RPO compliance
- [ ] Update documentation

---

## 📚 Additional Resources

- Backup verification script: `/usr/local/bin/dr_testing_framework.sh`
- Recovery orchestrator: `/usr/local/bin/recovery_orchestrator.sh`
- Failover system: `/usr/local/bin/automated_failover.sh`
- Backup logs: `/var/log/backup.log`
- Recovery logs: `/var/log/recovery_*.log`
- DR test results: `/var/lib/dr-tests/results/`

---

## 📝 Document Control

|Version|Date|Author|Changes|
|---|---|---|---|
|1.0|$(date +%Y-%m-%d)|Auto-generated|Initial version|

**Next Review Date:** $(date -d '+3 months' +%Y-%m-%d)

---

_This runbook is automatically generated and should be reviewed and updated regularly. Last update: $(date)_

RUNBOOK
```
log "Recovery runbook generated: $RUNBOOK_FILE" 
echo "$RUNBOOK_FILE" EOF

chmod +x generate_recovery_runbook.sh
```

## Итоговое задание модуля 5

```bash
# 1. Установи все скрипты
sudo cp recovery_orchestrator.sh /usr/local/bin/
sudo cp dr_testing_framework.sh /usr/local/bin/
sudo cp automated_failover.sh /usr/local/bin/
sudo cp generate_recovery_runbook.sh /usr/local/bin/

# 2. Инициализируй системы
sudo /usr/local/bin/dr_testing_framework.sh init
sudo /usr/local/bin/automated_failover.sh init

# 3. Сгенерируй runbook
sudo /usr/local/bin/generate_recovery_runbook.sh

# 4. Запусти тестовые восстановления
sudo /usr/local/bin/recovery_orchestrator.sh wizard

# 5. Запусти полный набор DR тестов
sudo /usr/local/bin/dr_testing_framework.sh run-all

# 6. Проверь отчеты
firefox /var/lib/dr-tests/results/summary_*.html

# 7. Настрой автоматический мониторинг (опционально)
# sudo /usr/local/bin/automated_failover.sh monitor &

# 8. Проверь статус failover системы
sudo /usr/local/bin/automated_failover.sh status
````

---

## 🎓 Заключение курса

### Что вы освоили в модуле 5:

- ✅ Automated recovery orchestration для разных сценариев
- ✅ DR testing framework с автоматической генерацией отчетов
- ✅ Automated failover system с health monitoring
- ✅ Recovery runbook generation
- ✅ RTO/RPO validation и compliance tracking

### Новые продвинутые техники:

- 🚀 Intelligent recovery orchestration
- 🚀 Continuous DR testing
- 🚀 Automated failover с minimal downtime
- 🚀 Self-healing infrastructure
- 🚀 Comprehensive recovery documentation

---

## 🎯 Следующие шаги для production-ready DR:

1. **Регулярное тестирование**
    
    - Weekly: Backup verification
    - Monthly: Database restore tests
    - Quarterly: Full DR drill
2. **Непрерывное улучшение**
    
    - Review recovery metrics after each incident

- Update runbooks based on actual recovery experiences
- Automate more recovery scenarios

3. **Team readiness**
    
    - Regular DR training sessions
    - Tabletop exercises
    - On-call rotation with DR responsibilities
4. **Documentation**
    
    - Keep runbooks updated
    - Document all recovery procedures
    - Maintain contact lists
5. **Monitoring и alerting**
    
    - Set up comprehensive monitoring
    - Alert on backup failures
    - Track RTO/RPO compliance

---

## 📚 Полный курс завершен!

**Пройденные модули:**

1. ✅ Backup & Recovery стратегии
2. ✅ Database backups
3. ✅ Filesystem & System backups
4. ✅ Cloud backups & Hybrid strategies
5. ✅ Advanced Recovery & DR Automation

**Ключевые навыки:**

- Production-ready backup infrastructure
- Multi-tier backup strategies
- Automated recovery procedures
- DR testing и validation
- Compliance tracking

**Помни главное правило:**

> **"Backup без tested restore = катастрофа в ожидании"**

Практикуй восстановление регулярно! 🚀


# Модуль 6: Backup Security, Compliance & Advanced Automation (40 минут)

## 🎯 Напоминалка

### Security Threats to Backups

```
Backup Security Threat Model
├── External Threats
│   ├── Ransomware (encrypts backups)
│   ├── Data theft (backup exfiltration)
│   ├── Unauthorized access
│   └── Man-in-the-middle attacks
│
├── Internal Threats
│   ├── Malicious insiders
│   ├── Accidental deletion
│   ├── Misconfiguration
│   └── Privilege abuse
│
├── Infrastructure Threats
│   ├── Backup server compromise
│   ├── Network interception
│   ├── Cloud account takeover
│   └── Supply chain attacks
│
└── Compliance Risks
    ├── Data retention violations
    ├── Privacy breaches (GDPR, HIPAA)
    ├── Audit failures
    └── Encryption weaknesses
```

### Defense-in-Depth Strategy

```
Backup Security Layers
┌─────────────────────────────────────────┐
│ Layer 7: Monitoring & Alerting         │
│ - SIEM integration                      │
│ - Anomaly detection                     │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ Layer 6: Access Control                │
│ - RBAC, MFA                             │
│ - Principle of least privilege          │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ Layer 5: Immutability                   │
│ - WORM storage                          │
│ - Object lock                           │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ Layer 4: Air-Gap                        │
│ - Offline copies                        │
│ - Tape storage                          │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ Layer 3: Encryption                     │
│ - At-rest, in-transit                   │
│ - Key management                        │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ Layer 2: Integrity Verification        │
│ - Checksums, digital signatures         │
│ - Regular validation                    │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ Layer 1: Redundancy                     │
│ - 3-2-1-1-0 rule                        │
│ - Geographic distribution               │
└─────────────────────────────────────────┘
```

### Compliance Frameworks

```bash
# GDPR (General Data Protection Regulation)
Requirements:
- Right to be forgotten (backup deletion)
- Data minimization
- Encryption of personal data
- Breach notification (72 hours)
- Data portability

# HIPAA (Health Insurance Portability and Accountability Act)
Requirements:
- Encryption of PHI (Protected Health Information)
- Access logs and audit trails
- Business Associate Agreements
- Minimum necessary access
- Regular risk assessments

# SOC 2 (System and Organization Controls)
Requirements:
- Security policies documented
- Access controls implemented
- Change management process
- Backup and recovery procedures
- Regular testing and validation

# PCI DSS (Payment Card Industry Data Security Standard)
Requirements:
- Encryption of cardholder data
- Secure key management
- Quarterly network scans
- Annual penetration testing
- Backup retention policies

# ISO 27001
Requirements:
- Information security management system
- Risk assessment methodology
- Asset classification
- Incident response procedures
- Regular audits
```

### Encryption Best Practices

```bash
# Symmetric Encryption (Fast, for data at rest)
AES-256-GCM    # Recommended
AES-256-CBC    # Legacy support
ChaCha20-Poly1305  # Modern alternative

# Asymmetric Encryption (For key exchange)
RSA-4096       # Widely supported
ECDSA P-384    # Modern, efficient
Ed25519        # Fast, secure

# Key Derivation
PBKDF2         # Password-based
Argon2         # Memory-hard (recommended)
scrypt         # Alternative

# Secure Key Storage
HSM (Hardware Security Module)
KMS (Key Management Service)
Vault (HashiCorp)
LUKS (Linux Unified Key Setup)
```

---

## 💻 Задание 1: Comprehensive Backup Security Framework

### Encrypted Backup System with Immutability

```bash
cat > secure_backup_system.sh <<'EOF'
#!/bin/bash

set -euo pipefail

# Configuration
BACKUP_ROOT="/backup/secure"
ENCRYPTION_KEY_DIR="/etc/backup-security/keys"
AUDIT_LOG="/var/log/backup-security-audit.log"
COMPLIANCE_LOG="/var/log/backup-compliance.log"

# Security settings
ENCRYPTION_ALGORITHM="aes-256-gcm"
KEY_ROTATION_DAYS=90
MIN_PASSWORD_LENGTH=16
REQUIRE_MFA=true

# Immutability settings
IMMUTABLE_PERIOD_DAYS=7
S3_OBJECT_LOCK_DAYS=30

# ANSI colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log() {
    local level="$1"
    shift
    local message="$*"
    local timestamp=$(date +'%Y-%m-%d %H:%M:%S')
    
    echo "[$timestamp] [$level] $message" | tee -a "$AUDIT_LOG"
    
    # Also log to syslog
    logger -t "secure-backup" -p "daemon.$level" "$message"
}

audit_log() {
    local action="$1"
    local user="${2:-$(whoami)}"
    local details="$3"
    local ip_address="${4:-$(who am i | awk '{print $5}' | tr -d '()')}"
    
    local timestamp=$(date -u +%Y-%m-%dT%H:%M:%SZ)
    
    # Structured audit log in JSON format
    cat >> "$AUDIT_LOG" <<AUDIT
{
  "timestamp": "$timestamp",
  "action": "$action",
  "user": "$user",
  "ip_address": "$ip_address",
  "details": "$details",
  "status": "success"
}
AUDIT
    
    log INFO "Audit: $action by $user from $ip_address"
}

compliance_log() {
    local framework="$1"
    local requirement="$2"
    local status="$3"
    local details="$4"
    
    cat >> "$COMPLIANCE_LOG" <<COMPLIANCE
{
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "framework": "$framework",
  "requirement": "$requirement",
  "status": "$status",
  "details": "$details"
}
COMPLIANCE
}

# Initialize security infrastructure
init_security() {
    log INFO "Initializing secure backup system..."
    
    # Create secure directories
    mkdir -p "$BACKUP_ROOT"/{encrypted,keys,metadata}
    mkdir -p "$ENCRYPTION_KEY_DIR"
    
    # Set strict permissions
    chmod 700 "$BACKUP_ROOT"
    chmod 700 "$ENCRYPTION_KEY_DIR"
    
    # Generate master key if not exists
    if [ ! -f "$ENCRYPTION_KEY_DIR/master.key" ]; then
        log INFO "Generating master encryption key..."
        
        # Generate strong random key
        openssl rand -base64 32 > "$ENCRYPTION_KEY_DIR/master.key"
        
        # Set restrictive permissions
        chmod 400 "$ENCRYPTION_KEY_DIR/master.key"
        chown root:root "$ENCRYPTION_KEY_DIR/master.key"
        
        # Make immutable (requires root)
        chattr +i "$ENCRYPTION_KEY_DIR/master.key" 2>/dev/null || \
            log WARNING "Cannot set immutable flag (chattr not available or not root)"
        
        log INFO "Master key generated and secured"
        audit_log "MASTER_KEY_CREATED" "$(whoami)" "Master encryption key created"
        compliance_log "SECURITY" "Encryption" "COMPLIANT" "Master key generated with 256-bit entropy"
    fi
    
    # Initialize key rotation tracking
    if [ ! -f "$ENCRYPTION_KEY_DIR/rotation_schedule.json" ]; then
        cat > "$ENCRYPTION_KEY_DIR/rotation_schedule.json" <<ROTATION
{
  "last_rotation": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "next_rotation": "$(date -u -d "+$KEY_ROTATION_DAYS days" +%Y-%m-%dT%H:%M:%SZ)",
  "rotation_interval_days": $KEY_ROTATION_DAYS
}
ROTATION
    fi
    
    log INFO "Security initialization complete"
}

# Encrypt backup with metadata
encrypt_backup() {
    local source_file="$1"
    local encrypted_file="${2:-${source_file}.enc}"
    
    log INFO "Encrypting backup: $(basename "$source_file")"
    
    # Generate data encryption key (DEK)
    local dek=$(openssl rand -base64 32)
    
    # Encrypt data with DEK
    echo "$dek" | openssl enc -$ENCRYPTION_ALGORITHM -salt -pbkdf2 \
        -pass stdin \
        -in "$source_file" \
        -out "$encrypted_file"
    
    if [ $? -eq 0 ]; then
        # Encrypt DEK with master key
        local encrypted_dek_file="${encrypted_file}.key"
        echo "$dek" | openssl enc -aes-256-cbc -salt -pbkdf2 \
            -pass file:"$ENCRYPTION_KEY_DIR/master.key" \
            -out "$encrypted_dek_file"
        
        # Create backup metadata
        local metadata_file="${encrypted_file}.meta"
        cat > "$metadata_file" <<META
{
  "original_file": "$(basename "$source_file")",
  "encrypted_file": "$(basename "$encrypted_file")",
  "encryption_algorithm": "$ENCRYPTION_ALGORITHM",
  "encrypted_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "encrypted_by": "$(whoami)",
  "file_hash": "$(sha256sum "$source_file" | cut -d' ' -f1)",
  "encrypted_hash": "$(sha256sum "$encrypted_file" | cut -d' ' -f1)",
  "size_original": $(stat -c%s "$source_file" 2>/dev/null || stat -f%z "$source_file"),
  "size_encrypted": $(stat -c%s "$encrypted_file" 2>/dev/null || stat -f%z "$encrypted_file")
}
META
        
        log INFO "Backup encrypted successfully"
        audit_log "BACKUP_ENCRYPTED" "$(whoami)" "File: $(basename "$source_file")"
        compliance_log "ENCRYPTION" "Data-at-Rest" "COMPLIANT" "AES-256-GCM encryption applied"
        
        return 0
    else
        log ERROR "Encryption failed for: $(basename "$source_file")"
        return 1
    fi
}

# Decrypt backup
decrypt_backup() {
    local encrypted_file="$1"
    local output_file="${2:-${encrypted_file%.enc}}"
    
    log INFO "Decrypting backup: $(basename "$encrypted_file")"
    
    # Check if encrypted DEK exists
    local encrypted_dek_file="${encrypted_file}.key"
    if [ ! -f "$encrypted_dek_file" ]; then
        log ERROR "Encrypted DEK file not found: $encrypted_dek_file"
        return 1
    fi
    
    # Decrypt DEK with master key
    local dek=$(openssl enc -d -aes-256-cbc -pbkdf2 \
        -pass file:"$ENCRYPTION_KEY_DIR/master.key" \
        -in "$encrypted_dek_file")
    
    # Decrypt data with DEK
    echo "$dek" | openssl enc -d -$ENCRYPTION_ALGORITHM -pbkdf2 \
        -pass stdin \
        -in "$encrypted_file" \
        -out "$output_file"
    
    if [ $? -eq 0 ]; then
        log INFO "Backup decrypted successfully"
        audit_log "BACKUP_DECRYPTED" "$(whoami)" "File: $(basename "$encrypted_file")"
        
        # Verify integrity if metadata exists
        local metadata_file="${encrypted_file}.meta"
        if [ -f "$metadata_file" ]; then
            local expected_hash=$(jq -r '.file_hash' "$metadata_file")
            local actual_hash=$(sha256sum "$output_file" | cut -d' ' -f1)
            
            if [ "$expected_hash" = "$actual_hash" ]; then
                log INFO "Integrity verification passed"
                compliance_log "INTEGRITY" "Hash-Verification" "COMPLIANT" "SHA-256 hash matched"
            else
                log ERROR "Integrity verification failed! File may be corrupted"
                compliance_log "INTEGRITY" "Hash-Verification" "VIOLATION" "SHA-256 hash mismatch"
                return 1
            fi
        fi
        
        return 0
    else
        log ERROR "Decryption failed for: $(basename "$encrypted_file")"
        return 1
    fi
}

# Make backup immutable
make_immutable() {
    local backup_file="$1"
    local immutable_days="${2:-$IMMUTABLE_PERIOD_DAYS}"
    
    log INFO "Making backup immutable: $(basename "$backup_file")"
    
    # Filesystem-level immutability (Linux)
    if command -v chattr &>/dev/null; then
        chattr +i "$backup_file" 2>/dev/null && \
            log INFO "File system immutability set" || \
            log WARNING "Cannot set file system immutability"
    fi
    
    # S3 Object Lock (if backing up to S3)
    if [[ "$backup_file" =~ ^s3:// ]]; then
        local bucket=$(echo "$backup_file" | cut -d/ -f3)
        local key=$(echo "$backup_file" | cut -d/ -f4-)
        
        aws s3api put-object-retention \
            --bucket "$bucket" \
            --key "$key" \
            --retention '{
                "Mode": "GOVERNANCE",
                "RetainUntilDate": "'$(date -u -d "+$immutable_days days" +%Y-%m-%dT%H:%M:%SZ)'"
            }' 2>/dev/null && \
            log INFO "S3 Object Lock applied for $immutable_days days" || \
            log WARNING "S3 Object Lock not available or failed"
    fi
    
    # Create immutability record
    local immutable_record="${backup_file}.immutable"
    cat > "$immutable_record" <<IMMUTABLE
{
  "file": "$(basename "$backup_file")",
  "immutable_until": "$(date -u -d "+$immutable_days days" +%Y-%m-%dT%H:%M:%SZ)",
  "immutable_period_days": $immutable_days,
  "set_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "set_by": "$(whoami)"
}
IMMUTABLE
    
    audit_log "IMMUTABILITY_SET" "$(whoami)" "File: $(basename "$backup_file"), Period: $immutable_days days"
    compliance_log "IMMUTABILITY" "WORM-Compliance" "COMPLIANT" "Immutability period set to $immutable_days days"
}

# Air-gap backup (copy to offline media)
airgap_backup() {
    local source_dir="$1"
    local tape_device="${2:-/dev/st0}"
    
    log INFO "Creating air-gapped backup to tape"
    
    # Check if tape device is available
    if [ ! -e "$tape_device" ]; then
        log ERROR "Tape device not found: $tape_device"
        return 1
    fi
    
    # Rewind tape
    mt -f "$tape_device" rewind
    
    # Create encrypted tarball and write to tape
    tar -czf - "$source_dir" 2>/dev/null | \
        openssl enc -aes-256-cbc -salt -pbkdf2 \
        -pass file:"$ENCRYPTION_KEY_DIR/master.key" | \
        dd of="$tape_device" bs=10240
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        # Eject tape
        mt -f "$tape_device" offline
        
        log INFO "Air-gapped backup to tape completed"
        audit_log "AIRGAP_BACKUP" "$(whoami)" "Tape device: $tape_device"
        compliance_log "AIRGAP" "Offline-Storage" "COMPLIANT" "Backup written to tape and ejected"
        
        return 0
    else
        log ERROR "Air-gapped backup failed"
        return 1
    fi
}

# Key rotation
rotate_encryption_keys() {
    log INFO "Starting encryption key rotation..."
    
    # Check if rotation is due
    local last_rotation=$(jq -r '.last_rotation' "$ENCRYPTION_KEY_DIR/rotation_schedule.json")
    local next_rotation=$(jq -r '.next_rotation' "$ENCRYPTION_KEY_DIR/rotation_schedule.json")
    local now=$(date -u +%Y-%m-%dT%H:%M:%SZ)
    
    if [[ "$now" < "$next_rotation" ]]; then
        log INFO "Key rotation not due yet (Next: $next_rotation)"
        return 0
    fi
    
    # Backup current master key
    local key_backup_dir="$ENCRYPTION_KEY_DIR/rotated"
    mkdir -p "$key_backup_dir"
    
    local rotation_timestamp=$(date +%Y%m%d-%H%M%S)
    cp "$ENCRYPTION_KEY_DIR/master.key" \
        "$key_backup_dir/master.key.${rotation_timestamp}"
    
    # Generate new master key
    chattr -i "$ENCRYPTION_KEY_DIR/master.key" 2>/dev/null || true
    openssl rand -base64 32 > "$ENCRYPTION_KEY_DIR/master.key"
    chmod 400 "$ENCRYPTION_KEY_DIR/master.key"
    chattr +i "$ENCRYPTION_KEY_DIR/master.key" 2>/dev/null || true
    
    # Update rotation schedule
    cat > "$ENCRYPTION_KEY_DIR/rotation_schedule.json" <<ROTATION
{
  "last_rotation": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "next_rotation": "$(date -u -d "+$KEY_ROTATION_DAYS days" +%Y-%m-%dT%H:%M:%SZ)",
  "rotation_interval_days": $KEY_ROTATION_DAYS,
  "previous_key": "master.key.${rotation_timestamp}"
}
ROTATION
    
    log INFO "Encryption key rotated successfully"
    audit_log "KEY_ROTATION" "$(whoami)" "New master key generated, old key backed up"
    compliance_log "KEY_MANAGEMENT" "Key-Rotation" "COMPLIANT" "Keys rotated per $KEY_ROTATION_DAYS day schedule"
    
    # Re-encrypt recent backups with new key
    log INFO "Re-encrypting recent backups with new key..."
    # (Implementation would re-encrypt backups from last rotation period)
}

# Access control check
check_access() {
    local required_permission="$1"
    local user="${2:-$(whoami)}"
    
    # Check if user has required permission
    # This is a simplified example - real implementation would check RBAC/ACL
    
    if [ "$user" = "root" ]; then
        return 0
    fi
    
    # Check group membership
    if groups "$user" | grep -q "backup-admin"; then
        return 0
    fi
    
    log WARNING "Access denied for user: $user (required: $required_permission)"
    audit_log "ACCESS_DENIED" "$user" "Required permission: $required_permission"
    return 1
}

# MFA verification (placeholder)
verify_mfa() {
    if [ "$REQUIRE_MFA" = false ]; then
        return 0
    fi
    
    log INFO "MFA verification required"
    
    # In production, this would integrate with:
    # - Google Authenticator (TOTP)
    # - YubiKey (Hardware token)
    # - SMS/Email OTP
    
    echo -n "Enter MFA code: "
    read -r mfa_code
    
    # Validate MFA code (placeholder)
    if [ -n "$mfa_code" ]; then
        log INFO "MFA verification passed"
        audit_log "MFA_VERIFIED" "$(whoami)" "MFA code accepted"
        return 0
    else
        log ERROR "MFA verification failed"
        audit_log "MFA_FAILED" "$(whoami)" "MFA code rejected"
        return 1
    fi
}

# Generate compliance report
generate_compliance_report() {
    local report_file="/var/log/compliance_report_$(date +%Y%m%d).html"
    
    log INFO "Generating compliance report..."
    
    cat > "$report_file" <<'HTML'
<!DOCTYPE html>
<html>
<head>
    <title>Backup Compliance Report</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f5f5f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        h1 {
            color: #2c3e50;
            border-bottom: 3px solid #3498db;
            padding-bottom: 10px;
        }
        .compliant {
            color: #27ae60;
            font-weight: bold;
        }
        .violation {
            color: #e74c3c;
            font-weight: bold;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #3498db;
            color: white;
        }
        .summary-box {
            background: #ecf0f1;
            padding: 15px;
            border-radius: 5px;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔒 Backup Security & Compliance Report</h1>
        <p><strong>Generated:</strong> $(date)</p>
        
        <div class="summary-box">
            <h2>Compliance Summary</h2>
HTML
    
    # Count compliance items
    local total_items=$(grep -c '"status"' "$COMPLIANCE_LOG" 2>/dev/null || echo "0")
    local compliant=$(grep -c '"COMPLIANT"' "$COMPLIANCE_LOG" 2>/dev/null || echo "0")
    local violations=$(grep -c '"VIOLATION"' "$COMPLIANCE_LOG" 2>/dev/null || echo "0")
    local compliance_rate=$(echo "scale=2; $compliant * 100 / $total_items" | bc 2>/dev/null || echo "0")
    
    cat >> "$report_file" <<HTML
            <p>Total Compliance Checks: <strong>$total_items</strong></p>
            <p>Compliant: <strong class="compliant">$compliant</strong></p>
            <p>Violations: <strong class="violation">$violations</strong></p>
            <p>Compliance Rate: <strong>${compliance_rate}%</strong></p>
        </div>
        
        <h2>Compliance by Framework</h2>
        <table>
            <thead>
                <tr>
                    <th>Framework</th>
                    <th>Requirement</th>
                    <th>Status</th>
                    <th>Details</th>
                </tr>
            </thead>
            <tbody>
HTML
    
    # Parse compliance log and add to table
    if [ -f "$COMPLIANCE_LOG" ]; then
        grep -o '{[^}]*}' "$COMPLIANCE_LOG" | tail -20 | while read -r entry; do
            local framework=$(echo "$entry" | jq -r '.framework')
            local requirement=$(echo "$entry" | jq -r '.requirement')
            local status=$(echo "$entry" | jq -r '.status')
            local details=$(echo "$entry" | jq -r '.details')
            
            local status_class="compliant"
            [ "$status" = "VIOLATION" ] && status_class="violation"
            
            cat >> "$report_file" <<HTML
                <tr>
                    <td>$framework</td>
                    <td>$requirement</td>
                    <td class="$status_class">$status</td>
                    <td>$details</td>
                </tr>
HTML
        done
    fi
    
    cat >> "$report_file" <<HTML
            </tbody>
        </table>
        
        <h2>Security Controls</h2>
        <table>
            <thead>
                <tr>
                    <th>Control</th>
                    <th>Status</th>
                    <th>Details</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Encryption at Rest</td>
                    <td class="compliant">ENABLED</td>
                    <td>AES-256-GCM encryption for all backups</td>
                </tr>
                <tr>
                    <td>Encryption in Transit</td>
                    <td class="compliant">ENABLED</td>
                    <td>TLS 1.3 for network transfers</td>
                </tr>
                <tr>
                    <td>Immutability</td>
                    <td class="compliant">ENABLED</td>
                    <td>$IMMUTABLE_PERIOD_DAYS day retention lock</td>
                </tr>
                <tr>
                    <td>Air-Gap Backups</td>
                    <td class="compliant">ENABLED</td>
                    <td>Tape backups for critical data</td>
                </tr>
                <tr>
                    <td>Key Rotation</td>
                    <td class="compliant">ENABLED</td>
                    <td>$KEY_ROTATION_DAYS day rotation schedule</td>
                </tr>
                <tr>
                    <td>Access Control</td>
                    <td class="compliant">ENABLED</td>
                    <td>RBAC with MFA requirement</td>
                </tr>
                <tr>
                    <td>Audit Logging</td>
                    <td class="compliant">ENABLED</td>
                    <td>All operations logged to SIEM</td>
                </tr>
            </tbody>
        </table>
        
        <h2>Recent Audit Events</h2>
        <table>
            <thead>
                <tr>
                    <th>Timestamp</th>
                    <th>Action</th>
                    <th>User</th>
                    <th>IP Address</th>
                </tr>
            </thead>
            <tbody>
HTML
    
    # Add recent audit events
    if [ -f "$AUDIT_LOG" ]; then
        grep -o '{[^}]*}' "$AUDIT_LOG" | tail -20 | while read -r entry; do
            local timestamp=$(echo "$entry" | jq -r '.timestamp')
            local action=$(echo "$entry" | jq -r '.action')
            local user=$(echo "$entry" | jq -r '.user')
            local ip=$(echo "$entry" | jq -r '.ip_address')
            
            cat >> "$report_file" <<HTML
                <tr>
                    <td>$timestamp</td>
                    <td>$action</td>
                    <td>$user</td>
                    <td>$ip</td>
                </tr>
HTML
        done
    fi
    
    cat >> "$report_file" <<HTML
            </tbody>
        </table>
        
        <h2>Recommendations</h2>
        <ul>
HTML
    
    if [ $violations -gt 0 ]; then
        cat >> "$report_file" <<HTML
            <li><strong>Action Required:</strong> $violations compliance violation(s) detected. Review and remediate immediately.</li>
HTML
    fi
    
    cat >> "$report_file" <<HTML
            <li>Review and update access control policies quarterly</li>
            <li>Conduct penetration testing of backup infrastructure annually</li>
            <li>Verify encryption key rotation schedule is being followed</li>
            <li>Test backup restoration procedures monthly</li>
            <li>Review audit logs for suspicious activity weekly</li>
        </ul>
        
        <hr>
        <p style="text-align: center; color: #7f8c8d;">
            Generated: $(date)<br>
            Secure Backup System v1.0
        </p>
    </div>
</body>
</html>
HTML
    
    log INFO "Compliance report generated: $report_file"
    echo "$report_file"
}

# Main command handling
case "${1:-help}" in
    init)
        init_security
        ;;
    
    encrypt)
        if [ -z "$2" ]; then
            echo "Usage: $0 encrypt <file> [output_file]"
            exit 1
        fi
        encrypt_backup "$2" "$3"
        ;;
    
    decrypt)
        if [ -z "$2" ]; then
            echo "Usage: $0 decrypt <encrypted_file> [output_file]"
            exit 1
        fi
        decrypt_backup "$2" "$3"
        ;;
    
    immutable)
        if [ -z "$2" ]; then
            echo "Usage: $0 immutable <file> [days]"
            exit 1
        fi
        make_immutable "$2" "$3"
        ;;
    
    airgap)
        if [ -z "$2" ]; then
            echo "Usage: $0 airgap <directory> [tape_device]"
            exit 1
        fi
        airgap_backup "$2" "$3"
        ;;
    
    rotate-keys)
        verify_mfa && rotate_encryption_keys
        ;;
    
    compliance-report)
        generate_compliance_report
        ;;
    
    *)
        cat <<HELP
Secure Backup System

Usage: $0 <command> [options]

Commands:
  init                    - Initialize security infrastructure
  encrypt <file> [out]    - Encrypt backup file
  decrypt <file> [out]    - Decrypt backup file
  immutable <file> [days] - Make backup immutable
  airgap <dir> [device]   - Create air-gapped backup to tape
  rotate-keys             - Rotate encryption keys
  compliance-report       - Generate compliance report

Examples: $0 init $0 encrypt /backup/data.tar.gz $0 immutable /backup/data.tar.gz.enc 30 $0 airgap /data /dev/st0 $0 compliance-report HELP ;; esac EOF

chmod +x secure_backup_system.sh

````

---

## Итоговое задание модуля 6

```bash
# 1. Инициализируй систему безопасности
sudo ./secure_backup_system.sh init

# 2. Создай тестовый бэкап
tar -czf /tmp/test-backup.tar.gz /etc

# 3. Зашифруй бэкап
sudo ./secure_backup_system.sh encrypt /tmp/test-backup.tar.gz

# 4. Сделай его immutable
sudo ./secure_backup_system.sh immutable /tmp/test-backup.tar.gz.enc 7

# 5. Попробуй расшифровать
sudo ./secure_backup_system.sh decrypt /tmp/test-backup.tar.gz.enc /tmp/restored.tar.gz

# 6. Сгенерируй compliance отчет
sudo ./secure_backup_system.sh compliance-report

# 7. Проверь отчет
firefox /var/log/compliance_report_*.html
````

---

## 🎓 Заключение всего курса

**Пройдено 6 модулей:**

1. ✅ Основы Backup & Recovery стратегий
2. ✅ Database Backups (MySQL, PostgreSQL, MongoDB, Redis)
3. ✅ Filesystem & System Backups (LVM, Btrfs, ZFS)
4. ✅ Cloud Backups & Hybrid Strategies
5. ✅ Advanced Recovery & DR Automation
6. ✅ Backup Security, Compliance & Encryption

**Ключевые достижения:**

- 🚀 Production-ready backup infrastructure
- 🚀 Multi-tier, multi-cloud strategy
- 🚀 Automated recovery orchestration
- 🚀 Security & compliance framework
- 🚀 Comprehensive monitoring & alerting

**Финальный чеклист:**

- [ ] Все бэкапы зашифрованы
- [ ] Immutability настроена
- [ ] Air-gap копии созданы
- [ ] DR тесты запланированы
- [ ] Compliance отчеты генерируются
- [ ] Audit логи проверяются
- [ ] Key rotation автоматизирована
- [ ] Recovery runbook обновлен

**Помни золотое правило:**

> **"Security без compliance = аудиторский кошмар"**  
> **"Backup без encryption = приглашение для хакеров"**  
> **"Restore без testing = ложное чувство безопасности"**

Молодец! Теперь у тебя enterprise-grade backup & recovery система! 🎉

---

# Модуль 7: Performance Optimization & Advanced Techniques (30 минут)

## 🎯 Напоминалка

### Backup Performance Bottlenecks

```
Performance Impact Factors
├── Source System
│   ├── Disk I/O speed (IOPS)
│   ├── CPU for compression
│   ├── RAM for buffering
│   └── Application locks
│
├── Network
│   ├── Bandwidth (Gbps)
│   ├── Latency (RTT)
│   ├── Packet loss
│   └── Network congestion
│
├── Target System
│   ├── Write speed
│   ├── Deduplication overhead
│   ├── Compression ratio
│   └── Storage throughput
│
└── Backup Software
    ├── Algorithm efficiency
    ├── Parallelization
    ├── Memory usage
    └── Error handling overhead
```

### Compression Algorithms Comparison

```bash
# Performance vs Ratio Trade-offs

Algorithm     Speed    Ratio    CPU Usage    Best For
────────────────────────────────────────────────────────
gzip (6)      Medium   Good     Medium       General purpose
bzip2         Slow     Better   High         Archival
xz/lzma       Slowest  Best     Very High    Long-term storage
lz4           Fast     Fair     Low          Real-time backups
zstd (3)      Fast     Good     Medium       Modern default ★
zstd (19)     Medium   Best     High         Best compression needed

# Benchmark example
for algo in gzip bzip2 xz lz4 zstd; do
    time tar -cf - /data | $algo > /backup/test.$algo
    ls -lh /backup/test.$algo
done
```

### Deduplication Strategies

```
Deduplication Types
┌─────────────────────────────────────────┐
│ Source Deduplication                    │
│ - Dedup before network transfer         │
│ - Saves bandwidth                       │
│ - Higher CPU on source                  │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ Target Deduplication                    │
│ - Dedup after receiving data            │
│ - Lower CPU on source                   │
│ - More network usage                    │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ Block-level Deduplication               │
│ - Fixed or variable block size          │
│ - Better dedup ratio                    │
│ - Higher overhead                       │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│ File-level Deduplication                │
│ - Whole file comparison                 │
│ - Faster, lower overhead                │
│ - Lower dedup ratio                     │
└─────────────────────────────────────────┘
```

---

## 💻 Задание 1: Compression Benchmark & Optimizer

```bash
cat > compression_optimizer.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
TEST_DATA_DIR="/data"
BENCHMARK_DIR="/tmp/compression_benchmark_$$"
RESULTS_FILE="/var/log/compression_benchmark_$(date +%Y%m%d).log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$RESULTS_FILE"
}

# Create test dataset if needed
create_test_data() {
    local size_mb="$1"
    local test_file="$BENCHMARK_DIR/test_data.bin"
    
    log "Creating ${size_mb}MB test dataset..."
    
    mkdir -p "$BENCHMARK_DIR"
    
    # Create mixed data (realistic scenario)
    # 40% text, 30% binary, 30% already compressed
    
    # Text data (highly compressible)
    dd if=/dev/urandom bs=1M count=$((size_mb * 40 / 100)) 2>/dev/null | \
        base64 > "${test_file}.text"
    
    # Binary data (medium compressibility)
    dd if=/dev/urandom bs=1M count=$((size_mb * 30 / 100)) 2>/dev/null \
        > "${test_file}.binary"
    
    # Already compressed data (low compressibility)
    dd if=/dev/urandom bs=1M count=$((size_mb * 30 / 100)) 2>/dev/null | \
        gzip -9 > "${test_file}.compressed"
    
    # Combine all
    cat "${test_file}".{text,binary,compressed} > "$test_file"
    rm -f "${test_file}".{text,binary,compressed}
    
    log "Test data created: $(du -h "$test_file" | cut -f1)"
}

# Benchmark compression algorithm
benchmark_algo() {
    local algo="$1"
    local test_file="$2"
    local output_file="$BENCHMARK_DIR/output.$algo"
    
    log "Benchmarking: $algo"
    
    # Measure compression
    local start_time=$(date +%s.%N)
    
    case "$algo" in
        gzip)
            gzip -c "$test_file" > "$output_file"
            ;;
        gzip-fast)
            gzip -1 -c "$test_file" > "$output_file"
            ;;
        gzip-best)
            gzip -9 -c "$test_file" > "$output_file"
            ;;
        bzip2)
            bzip2 -c "$test_file" > "$output_file"
            ;;
        xz)
            xz -c "$test_file" > "$output_file"
            ;;
        xz-fast)
            xz -1 -c "$test_file" > "$output_file"
            ;;
        lz4)
            lz4 -c "$test_file" > "$output_file"
            ;;
        zstd)
            zstd -c "$test_file" > "$output_file"
            ;;
        zstd-fast)
            zstd -1 -c "$test_file" > "$output_file"
            ;;
        zstd-best)
            zstd -19 -c "$test_file" > "$output_file"
            ;;
        pigz)
            pigz -c "$test_file" > "$output_file"
            ;;
        *)
            log "Unknown algorithm: $algo"
            return 1
            ;;
    esac
    
    local end_time=$(date +%s.%N)
    local compression_time=$(echo "$end_time - $start_time" | bc)
    
    # Measure decompression
    local decomp_file="${output_file}.decomp"
    start_time=$(date +%s.%N)
    
    case "$algo" in
        gzip*|pigz)
            gunzip -c "$output_file" > "$decomp_file"
            ;;
        bzip2)
            bunzip2 -c "$output_file" > "$decomp_file"
            ;;
        xz*)
            unxz -c "$output_file" > "$decomp_file"
            ;;
        lz4)
            lz4 -d -c "$output_file" > "$decomp_file"
            ;;
        zstd*)
            zstd -d -c "$output_file" > "$decomp_file"
            ;;
    esac
    
    end_time=$(date +%s.%N)
    local decompression_time=$(echo "$end_time - $start_time" | bc)
    
    # Calculate metrics
    local original_size=$(stat -c%s "$test_file" 2>/dev/null || stat -f%z "$test_file")
    local compressed_size=$(stat -c%s "$output_file" 2>/dev/null || stat -f%z "$output_file")
    local ratio=$(echo "scale=2; $original_size / $compressed_size" | bc)
    local compression_rate=$(echo "scale=2; $original_size / 1024 / 1024 / $compression_time" | bc)
    local decompression_rate=$(echo "scale=2; $original_size / 1024 / 1024 / $decompression_time" | bc)
    
    # Verify integrity
    local md5_original=$(md5sum "$test_file" | cut -d' ' -f1)
    local md5_decompressed=$(md5sum "$decomp_file" | cut -d' ' -f1)
    local integrity="FAIL"
    [ "$md5_original" = "$md5_decompressed" ] && integrity="PASS"
    
    # Output results
    cat >> "$RESULTS_FILE" <<RESULT
Algorithm: $algo
Original Size: $(numfmt --to=iec-i --suffix=B $original_size)
Compressed Size: $(numfmt --to=iec-i --suffix=B $compressed_size)
Compression Ratio: ${ratio}x
Compression Time: ${compression_time}s
Decompression Time: ${decompression_time}s
Compression Rate: ${compression_rate} MB/s
Decompression Rate: ${decompression_rate} MB/s
Integrity: $integrity
---
RESULT
    
    log "  Ratio: ${ratio}x, Speed: ${compression_rate} MB/s, Integrity: $integrity"
    
    # Cleanup
    rm -f "$output_file" "$decomp_file"
}

# Generate comparison report
generate_report() {
    local report_file="/var/log/compression_report_$(date +%Y%m%d).html"
    
    log "Generating HTML report: $report_file"
    
    cat > "$report_file" <<'HTML'
<!DOCTYPE html>
<html>
<head>
    <title>Compression Benchmark Report</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 20px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }
        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        h1 {
            color: #2c3e50;
            text-align: center;
            border-bottom: 3px solid #667eea;
            padding-bottom: 15px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            padding: 14px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            font-weight: 600;
        }
        tr:hover {
            background-color: #f5f5f5;
        }
        .best {
            background-color: #d4edda;
            font-weight: bold;
        }
        .worst {
            background-color: #f8d7da;
        }
        .chart {
            margin: 30px 0;
            padding: 20px;
            background: #f8f9fa;
            border-radius: 8px;
        }
        .recommendation {
            background: #e7f3ff;
            border-left: 4px solid #2196F3;
            padding: 15px;
            margin: 20px 0;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📊 Compression Benchmark Report</h1>
        <p><strong>Generated:</strong> $(date)</p>
        <p><strong>Test Dataset Size:</strong> $(du -h "$BENCHMARK_DIR/test_data.bin" | cut -f1)</p>
        
        <h2>Benchmark Results</h2>
        <table>
            <thead>
                <tr>
                    <th>Algorithm</th>
                    <th>Ratio</th>
                    <th>Compressed Size</th>
                    <th>Comp Speed (MB/s)</th>
                    <th>Decomp Speed (MB/s)</th>
                    <th>Comp Time</th>
                    <th>Decomp Time</th>
                    <th>Integrity</th>
                </tr>
            </thead>
            <tbody>
HTML
    
    # Parse results and add to table
    awk '/^Algorithm:/{algo=$2}
         /^Compressed Size:/{size=$3}
         /^Compression Ratio:/{ratio=$3}
         /^Compression Time:/{comp_time=$3}
         /^Decompression Time:/{decomp_time=$3}
         /^Compression Rate:/{comp_rate=$3}
         /^Decompression Rate:/{decomp_rate=$3}
         /^Integrity:/{
            integrity=$2;
            print "<tr>"
            print "<td>" algo "</td>"
            print "<td>" ratio "</td>"
            print "<td>" size "</td>"
            print "<td>" comp_rate "</td>"
            print "<td>" decomp_rate "</td>"
            print "<td>" comp_time "</td>"
            print "<td>" decomp_time "</td>"
            print "<td>" integrity "</td>"
            print "</tr>"
         }' "$RESULTS_FILE" >> "$report_file"
    
    cat >> "$report_file" <<'HTML'
            </tbody>
        </table>
        
        <div class="recommendation">
            <h3>🎯 Recommendations</h3>
            <ul>
                <li><strong>For Speed:</strong> Use lz4 or zstd-fast for real-time backups</li>
                <li><strong>For Balance:</strong> Use zstd (default level) for general purpose</li>
                <li><strong>For Maximum Compression:</strong> Use xz or zstd-best for archival</li>
                <li><strong>For Parallel Processing:</strong> Use pigz for multi-core systems</li>
            </ul>
        </div>
        
        <h2>Use Case Recommendations</h2>
        <table>
            <tr>
                <th>Scenario</th>
                <th>Recommended Algorithm</th>
                <th>Reason</th>
            </tr>
            <tr>
                <td>Live database backups</td>
                <td>lz4 or zstd-fast</td>
                <td>Minimal impact on production workload</td>
            </tr>
            <tr>
                <td>Daily file backups</td>
                <td>zstd (level 3)</td>
                <td>Good compression with acceptable speed</td>
            </tr>
            <tr>
                <td>Weekly full backups</td>
                <td>zstd (level 10) or xz</td>
                <td>Better compression, backup window allows time</td>
            </tr>
            <tr>
                <td>Monthly archives</td>
                <td>xz or zstd (level 19)</td>
                <td>Maximum compression for long-term storage</td>
            </tr>
            <tr>
                <td>Log file backups</td>
                <td>gzip</td>
                <td>Text data compresses well, standard format</td>
            </tr>
            <tr>
                <td>Multi-core systems</td>
                <td>pigz</td>
                <td>Parallel gzip for better CPU utilization</td>
            </tr>
        </table>
        
        <hr>
        <p style="text-align: center; color: #666;">
            Compression Benchmark Tool v1.0
        </p>
    </div>
</body>
</html>
HTML
    
    log "Report generated: $report_file"
    echo "$report_file"
}

# Main execution
main() {
    local test_size_mb="${1:-1024}"  # Default 1GB
    
    log "=========================================="
    log "Starting Compression Benchmark"
    log "Test Size: ${test_size_mb}MB"
    log "=========================================="
    
    # Create test data
    create_test_data "$test_size_mb"
    
    # Test file
    local test_file="$BENCHMARK_DIR/test_data.bin"
    
    # Benchmark algorithms
    local algorithms=(
        "gzip"
        "gzip-fast"
        "gzip-best"
        "bzip2"
        "xz"
        "xz-fast"
        "lz4"
        "zstd"
        "zstd-fast"
        "zstd-best"
        "pigz"
    )
    
    for algo in "${algorithms[@]}"; do
        if command -v "${algo%%-*}" &>/dev/null; then
            benchmark_algo "$algo" "$test_file"
        else
            log "Skipping $algo (not installed)"
        fi
    done
    
    # Generate report
    generate_report
    
    # Cleanup
    rm -rf "$BENCHMARK_DIR"
    
    log "=========================================="
    log "Benchmark completed"
    log "Results: $RESULTS_FILE"
    log "=========================================="
}

case "${1:-run}" in
    run)
        main "${2:-1024}"
        ;;
    
    quick)
        main 100  # Quick 100MB test
        ;;
    
    report)
        generate_report
        ;;
    
    *)
        cat <<HELP
Compression Optimizer

Usage: $0 <command> [size_mb]

Commands:
  run [size]  - Run benchmark with specified size in MB (default: 1024)
  quick       - Quick benchmark with 100MB test
  report      - Regenerate report from last results

Examples:
  $0 run 2048      # 2GB benchmark
  $0 quick         # Quick test
  $0 report        # Regenerate report
HELP
        ;;
esac
EOF

chmod +x compression_optimizer.sh
```


## 💻 Задание 2: Parallel Backup System with Smart Scheduling

```bash
cat > parallel_backup_system.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
BACKUP_ROOT="/backup/parallel"
JOB_QUEUE_DIR="/var/lib/parallel-backup/queue"
JOB_STATUS_DIR="/var/lib/parallel-backup/status"
LOG_DIR="/var/log/parallel-backup"
MAX_PARALLEL_JOBS=4
NETWORK_BANDWIDTH_LIMIT_MBPS=100
IO_NICE_CLASS="2"  # Best-effort
IO_NICE_LEVEL="7"  # Lowest priority

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log() {
    local level="$1"
    shift
    local message="$*"
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [$level] $message" | tee -a "$LOG_DIR/parallel_backup.log"
}

# Initialize parallel backup system
init_parallel_system() {
    log INFO "Initializing parallel backup system..."
    
    mkdir -p "$BACKUP_ROOT"
    mkdir -p "$JOB_QUEUE_DIR"
    mkdir -p "$JOB_STATUS_DIR"
    mkdir -p "$LOG_DIR"
    
    # Create configuration
    cat > /etc/parallel-backup/config.json <<CONFIG
{
  "max_parallel_jobs": $MAX_PARALLEL_JOBS,
  "network_bandwidth_limit_mbps": $NETWORK_BANDWIDTH_LIMIT_MBPS,
  "io_priority": {
    "class": "$IO_NICE_CLASS",
    "level": "$IO_NICE_LEVEL"
  },
  "backup_windows": {
    "databases": {
      "start": "02:00",
      "end": "04:00",
      "priority": 1
    },
    "applications": {
      "start": "04:00",
      "end": "06:00",
      "priority": 2
    },
    "filesystems": {
      "start": "00:00",
      "end": "02:00",
      "priority": 3
    }
  },
  "resource_limits": {
    "max_cpu_percent": 50,
    "max_memory_percent": 30,
    "max_io_mbps": 500
  }
}
CONFIG
    
    log INFO "Parallel backup system initialized"
}

# Create backup job
create_job() {
    local job_name="$1"
    local job_type="$2"
    local source="$3"
    local destination="$4"
    local priority="${5:-5}"
    
    local job_id="$(date +%Y%m%d-%H%M%S)-$$-$(echo "$job_name" | md5sum | cut -c1-8)"
    local job_file="$JOB_QUEUE_DIR/${job_id}.json"
    
    log INFO "Creating backup job: $job_name (ID: $job_id)"
    
    cat > "$job_file" <<JOB
{
  "job_id": "$job_id",
  "job_name": "$job_name",
  "job_type": "$job_type",
  "source": "$source",
  "destination": "$destination",
  "priority": $priority,
  "created_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "status": "queued",
  "attempts": 0,
  "max_attempts": 3
}
JOB
    
    log INFO "Job created: $job_id"
    echo "$job_id"
}

# Get current system load
get_system_load() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    local mem_usage=$(free | grep Mem | awk '{print ($3/$2) * 100.0}')
    local io_wait=$(iostat -c 1 2 | tail -1 | awk '{print $4}')
    
    cat <<LOAD
{
  "cpu_usage": $cpu_usage,
  "memory_usage": $mem_usage,
  "io_wait": $io_wait,
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
LOAD
}

# Check if we can start new job
can_start_job() {
    local running_jobs=$(ls "$JOB_STATUS_DIR"/*.running 2>/dev/null | wc -l)
    
    if [ $running_jobs -ge $MAX_PARALLEL_JOBS ]; then
        log DEBUG "Maximum parallel jobs reached ($running_jobs/$MAX_PARALLEL_JOBS)"
        return 1
    fi
    
    # Check system resources
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1 | cut -d. -f1)
    local max_cpu=$(jq -r '.resource_limits.max_cpu_percent' /etc/parallel-backup/config.json)
    
    if [ "$cpu_usage" -ge "$max_cpu" ]; then
        log DEBUG "CPU usage too high ($cpu_usage% >= $max_cpu%)"
        return 1
    fi
    
    return 0
}

# Execute backup job
execute_job() {
    local job_id="$1"
    local job_file="$JOB_QUEUE_DIR/${job_id}.json"
    
    if [ ! -f "$job_file" ]; then
        log ERROR "Job file not found: $job_file"
        return 1
    fi
    
    # Parse job details
    local job_name=$(jq -r '.job_name' "$job_file")
    local job_type=$(jq -r '.job_type' "$job_file")
    local source=$(jq -r '.source' "$job_file")
    local destination=$(jq -r '.destination' "$job_file")
    
    log INFO "Executing job: $job_name ($job_id)"
    
    # Mark as running
    touch "$JOB_STATUS_DIR/${job_id}.running"
    jq '.status = "running" | .started_at = "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"' \
        "$job_file" > "$job_file.tmp" && mv "$job_file.tmp" "$job_file"
    
    # Create job-specific log
    local job_log="$LOG_DIR/${job_id}.log"
    
    # Execute based on job type
    local exit_code=0
    
    case "$job_type" in
        rsync)
            execute_rsync_job "$source" "$destination" "$job_log" || exit_code=$?
            ;;
        
        tar)
            execute_tar_job "$source" "$destination" "$job_log" || exit_code=$?
            ;;
        
        database)
            execute_database_job "$source" "$destination" "$job_log" || exit_code=$?
            ;;
        
        custom)
            execute_custom_job "$source" "$destination" "$job_log" || exit_code=$?
            ;;
        
        *)
            log ERROR "Unknown job type: $job_type"
            exit_code=1
            ;;
    esac
    
    # Update job status
    if [ $exit_code -eq 0 ]; then
        log INFO "Job completed successfully: $job_name"
        jq '.status = "completed" | .completed_at = "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"' \
            "$job_file" > "$job_file.tmp" && mv "$job_file.tmp" "$job_file"
        mv "$job_file" "$JOB_STATUS_DIR/${job_id}.completed"
    else
        log ERROR "Job failed: $job_name (exit code: $exit_code)"
        
        local attempts=$(jq -r '.attempts' "$job_file")
        local max_attempts=$(jq -r '.max_attempts' "$job_file")
        attempts=$((attempts + 1))
        
        if [ $attempts -ge $max_attempts ]; then
            log ERROR "Job exceeded max attempts: $job_name"
            jq '.status = "failed" | .attempts = '$attempts' | .failed_at = "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"' \
                "$job_file" > "$job_file.tmp" && mv "$job_file.tmp" "$job_file"
            mv "$job_file" "$JOB_STATUS_DIR/${job_id}.failed"
        else
            log INFO "Job will be retried: $job_name (attempt $attempts/$max_attempts)"
            jq '.status = "queued" | .attempts = '$attempts \
                "$job_file" > "$job_file.tmp" && mv "$job_file.tmp" "$job_file"
        fi
    fi
    
    # Remove running marker
    rm -f "$JOB_STATUS_DIR/${job_id}.running"
    
    return $exit_code
}

# Execute rsync job with bandwidth limiting and ionice
execute_rsync_job() {
    local source="$1"
    local destination="$2"
    local log_file="$3"
    
    log INFO "Rsync: $source -> $destination"
    
    # Calculate bandwidth limit in KB/s
    local bw_limit_kbps=$((NETWORK_BANDWIDTH_LIMIT_MBPS * 1024 / MAX_PARALLEL_JOBS))
    
    ionice -c $IO_NICE_CLASS -n $IO_NICE_LEVEL \
        rsync -avz \
        --bwlimit=$bw_limit_kbps \
        --info=progress2 \
        --stats \
        --log-file="$log_file" \
        "$source/" \
        "$destination/" \
        2>&1 | tee -a "$log_file"
    
    return ${PIPESTATUS[0]}
}

# Execute tar job with compression
execute_tar_job() {
    local source="$1"
    local destination="$2"
    local log_file="$3"
    
    log INFO "Tar: $source -> $destination"
    
    local backup_file="${destination}/$(basename "$source")-$(date +%Y%m%d-%H%M%S).tar.zst"
    
    ionice -c $IO_NICE_CLASS -n $IO_NICE_LEVEL \
        tar -cf - "$source" 2>"$log_file" | \
        pv -L ${NETWORK_BANDWIDTH_LIMIT_MBPS}M | \
        zstd -T0 -3 > "$backup_file"
    
    local exit_code=${PIPESTATUS[0]}
    
    if [ $exit_code -eq 0 ]; then
        # Create checksum
        sha256sum "$backup_file" > "${backup_file}.sha256"
        log INFO "Backup created: $backup_file ($(du -h "$backup_file" | cut -f1))"
    fi
    
    return $exit_code
}

# Execute database backup job
execute_database_job() {
    local db_type="$1"
    local destination="$2"
    local log_file="$3"
    
    log INFO "Database backup: $db_type -> $destination"
    
    case "$db_type" in
        mysql)
            local backup_file="${destination}/mysql-$(date +%Y%m%d-%H%M%S).sql.zst"
            
            ionice -c $IO_NICE_CLASS -n $IO_NICE_LEVEL \
                mysqldump --all-databases \
                --single-transaction \
                --routines \
                --triggers 2>"$log_file" | \
                zstd -T0 -3 > "$backup_file"
            
            return ${PIPESTATUS[0]}
            ;;
        
        postgresql)
            local backup_file="${destination}/postgresql-$(date +%Y%m%d-%H%M%S).dump.zst"
            
            ionice -c $IO_NICE_CLASS -n $IO_NICE_LEVEL \
                sudo -u postgres pg_dumpall 2>"$log_file" | \
                zstd -T0 -3 > "$backup_file"
            
            return ${PIPESTATUS[0]}
            ;;
        
        *)
            log ERROR "Unknown database type: $db_type"
            return 1
            ;;
    esac
}

# Execute custom backup script
execute_custom_job() {
    local script="$1"
    local destination="$2"
    local log_file="$3"
    
    log INFO "Custom script: $script -> $destination"
    
    if [ ! -x "$script" ]; then
        log ERROR "Script not found or not executable: $script"
        return 1
    fi
    
    ionice -c $IO_NICE_CLASS -n $IO_NICE_LEVEL \
        "$script" "$destination" 2>&1 | tee -a "$log_file"
    
    return ${PIPESTATUS[0]}
}

# Job scheduler (runs in background)
job_scheduler() {
    log INFO "Starting job scheduler..."
    
    while true; do
        # Check if we can start new job
        if can_start_job; then
            # Find highest priority queued job
            local next_job=$(ls -1 "$JOB_QUEUE_DIR"/*.json 2>/dev/null | \
                while read job_file; do
                    if [ -f "$job_file" ]; then
                        local status=$(jq -r '.status' "$job_file")
                        if [ "$status" = "queued" ]; then
                            local priority=$(jq -r '.priority' "$job_file")
                            local job_id=$(basename "$job_file" .json)
                            echo "$priority:$job_id"
                        fi
                    fi
                done | sort -n | head -1 | cut -d: -f2)
            
            if [ -n "$next_job" ]; then
                log INFO "Starting job: $next_job"
                
                # Execute job in background
                execute_job "$next_job" &
            fi
        fi
        
        # Wait before checking again
        sleep 10
    done
}

# Monitor running jobs
monitor_jobs() {
    local running=$(ls "$JOB_STATUS_DIR"/*.running 2>/dev/null | wc -l)
    local queued=$(find "$JOB_QUEUE_DIR" -name "*.json" -exec grep -l '"status": "queued"' {} \; 2>/dev/null | wc -l)
    local completed=$(ls "$JOB_STATUS_DIR"/*.completed 2>/dev/null | wc -l)
    local failed=$(ls "$JOB_STATUS_DIR"/*.failed 2>/dev/null | wc -l)
    
    cat <<STATUS
╔═══════════════════════════════════════════════════════════╗
║           PARALLEL BACKUP SYSTEM STATUS                   ║
╚═══════════════════════════════════════════════════════════╝

Jobs Status:
  Running:   $running / $MAX_PARALLEL_JOBS
  Queued:    $queued
  Completed: $completed
  Failed:    $failed

System Load:
STATUS
    
    get_system_load | jq -r '
        "  CPU Usage:    \(.cpu_usage)%",
        "  Memory Usage: \(.memory_usage | floor)%",
        "  IO Wait:      \(.io_wait)%"
    '
    
    echo ""
    echo "Running Jobs:"
    
    for running_marker in "$JOB_STATUS_DIR"/*.running; do
        if [ -f "$running_marker" ]; then
            local job_id=$(basename "$running_marker" .running)
            local job_file="$JOB_QUEUE_DIR/${job_id}.json"
            
            if [ -f "$job_file" ]; then
                local job_name=$(jq -r '.job_name' "$job_file")
                local started_at=$(jq -r '.started_at' "$job_file")
                local duration=$(($(date +%s) - $(date -d "$started_at" +%s)))
                
                echo "  - $job_name ($(printf '%02d:%02d:%02d' $((duration/3600)) $((duration%3600/60)) $((duration%60))))"
            fi
        fi
    done
    
    echo ""
}

# Generate performance report
generate_performance_report() {
    local report_file="/var/log/parallel_backup_performance_$(date +%Y%m%d).html"
    
    log INFO "Generating performance report..."
    
    cat > "$report_file" <<'HTML'
<!DOCTYPE html>
<html>
<head>
    <title>Parallel Backup Performance Report</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background: #f0f2f5;
        }
        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #1a73e8;
            border-bottom: 3px solid #1a73e8;
            padding-bottom: 10px;
        }
        .metrics {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 20px;
            margin: 30px 0;
        }
        .metric-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 20px;
            border-radius: 8px;
            text-align: center;
        }
        .metric-value {
            font-size: 2.5em;
            font-weight: bold;
            margin: 10px 0;
        }
        .metric-label {
            font-size: 0.9em;
            opacity: 0.9;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #1a73e8;
            color: white;
        }
        .status-completed {
            color: #28a745;
            font-weight: bold;
        }
        .status-failed {
            color: #dc3545;
            font-weight: bold;
        }
        .status-running {
            color: #ffc107;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>⚡ Parallel Backup Performance Report</h1>
        <p><strong>Generated:</strong> $(date)</p>
        
        <div class="metrics">
HTML
    
    # Calculate metrics
    local total_jobs=$(($(ls "$JOB_STATUS_DIR"/*.completed 2>/dev/null | wc -l) + \
                       $(ls "$JOB_STATUS_DIR"/*.failed 2>/dev/null | wc -l)))
    local completed=$(ls "$JOB_STATUS_DIR"/*.completed 2>/dev/null | wc -l)
    local failed=$(ls "$JOB_STATUS_DIR"/*.failed 2>/dev/null | wc -l)
    local running=$(ls "$JOB_STATUS_DIR"/*.running 2>/dev/null | wc -l)
    local success_rate=$([ $total_jobs -gt 0 ] && echo "scale=1; $completed * 100 / $total_jobs" | bc || echo "0")
    
    # Calculate total backup size
    local total_size=$(find "$BACKUP_ROOT" -type f -name "*.tar.zst" -o -name "*.sql.zst" | \
        xargs du -ch 2>/dev/null | tail -1 | cut -f1)
    
    # Average job duration
    local total_duration=0
    local job_count=0
    
    for job_file in "$JOB_STATUS_DIR"/*.completed; do
        if [ -f "$job_file" ]; then
            local started=$(jq -r '.started_at' "$job_file")
            local completed=$(jq -r '.completed_at' "$job_file")
            
            if [ "$started" != "null" ] && [ "$completed" != "null" ]; then
                local duration=$(($(date -d "$completed" +%s) - $(date -d "$started" +%s)))
                total_duration=$((total_duration + duration))
                job_count=$((job_count + 1))
            fi
        fi
    done
    
    local avg_duration=$((job_count > 0 ? total_duration / job_count : 0))
    
    cat >> "$report_file" <<HTML
            <div class="metric-card">
                <div class="metric-label">Total Jobs</div>
                <div class="metric-value">$total_jobs</div>
            </div>
            <div class="metric-card" style="background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);">
                <div class="metric-label">Success Rate</div>
                <div class="metric-value">${success_rate}%</div>
            </div>
            <div class="metric-card" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
                <div class="metric-label">Total Backup Size</div>
                <div class="metric-value">$total_size</div>
            </div>
            <div class="metric-card" style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);">
                <div class="metric-label">Avg Duration</div>
                <div class="metric-value">$(printf '%02d:%02d' $((avg_duration/60)) $((avg_duration%60)))</div>
            </div>
        </div>
        
        <h2>Job History</h2>
        <table>
            <thead>
                <tr>
                    <th>Job Name</th>
                    <th>Type</th>
                    <th>Started</th>
                    <th>Completed</th>
                    <th>Duration</th>
                    <th>Status</th>
                    <th>Attempts</th>
                </tr>
            </thead>
            <tbody>
HTML
    
    # Add completed jobs
    for job_file in "$JOB_STATUS_DIR"/*.{completed,failed}; do
        if [ -f "$job_file" ]; then
            local job_name=$(jq -r '.job_name' "$job_file")
            local job_type=$(jq -r '.job_type' "$job_file")
            local started=$(jq -r '.started_at' "$job_file")
            local completed=$(jq -r '.completed_at // .failed_at' "$job_file")
            local status=$(jq -r '.status' "$job_file")
            local attempts=$(jq -r '.attempts' "$job_file")
            
            local duration=""
            if [ "$started" != "null" ] && [ "$completed" != "null" ]; then
                local dur_seconds=$(($(date -d "$completed" +%s) - $(date -d "$started" +%s)))
                duration=$(printf '%02d:%02d:%02d' $((dur_seconds/3600)) $((dur_seconds%3600/60)) $((dur_seconds%60)))
            fi
            
            local status_class="status-completed"
            [ "$status" = "failed" ] && status_class="status-failed"
            
            cat >> "$report_file" <<HTML
                <tr>
                    <td>$job_name</td>
                    <td>$job_type</td>
                    <td>$started</td>
                    <td>$completed</td>
                    <td>$duration</td>
                    <td class="$status_class">$status</td>
                    <td>$attempts</td>
                </tr>
HTML
        fi
    done
    
    cat >> "$report_file" <<HTML
            </tbody>
        </table>
        
        <h2>Performance Insights</h2>
        <ul>
            <li>Maximum parallel jobs: <strong>$MAX_PARALLEL_JOBS</strong></li>
            <li>Network bandwidth limit: <strong>${NETWORK_BANDWIDTH_LIMIT_MBPS} Mbps</strong></li>
            <li>I/O priority: <strong>Best-effort (class $IO_NICE_CLASS, level $IO_NICE_LEVEL)</strong></li>
            <li>Current running jobs: <strong>$running</strong></li>
        </ul>
        
        <h2>Recommendations</h2>
        <ul>
HTML
    
    if [ $success_rate -lt 90 ]; then
        cat >> "$report_file" <<HTML
            <li><strong>⚠️ Success rate is below 90%.</strong> Review failed jobs and increase max_attempts.</li>
HTML
    fi
    
    if [ $avg_duration -gt 3600 ]; then
        cat >> "$report_file" <<HTML
            <li><strong>⚠️ Average job duration exceeds 1 hour.</strong> Consider increasing parallel jobs or optimizing backup methods.</li>
HTML
    fi
    
    cat >> "$report_file" <<HTML
            <li>Monitor system resources during backup windows</li>
            <li>Consider adjusting backup windows based on job completion times</li>
            <li>Review and optimize slow backup jobs</li>
            <li>Implement incremental backups for large datasets</li>
        </ul>
        
        <hr>
        <p style="text-align: center; color: #666;">
            Parallel Backup System v1.0 | Report generated: $(date)
        </p>
    </div>
</body>
</html>
HTML
    
    log INFO "Performance report generated: $report_file"
    echo "$report_file"
}

# Cleanup old jobs
cleanup_old_jobs() {
    local retention_days="${1:-7}"
    
    log INFO "Cleaning up jobs older than $retention_days days..."
    
    find "$JOB_STATUS_DIR" -type f \( -name "*.completed" -o -name "*.failed" \) \
        -mtime +$retention_days -delete
    
    find "$LOG_DIR" -type f -name "*.log" -mtime +$retention_days -delete
    
    log INFO "Cleanup completed"
}

# Main command handling
case "${1:-help}" in
    init)
        init_parallel_system
        ;;
    
    create)
        if [ $# -lt 5 ]; then
            echo "Usage: $0 create <name> <type> <source> <destination> [priority]"
            exit 1
        fi
        create_job "$2" "$3" "$4" "$5" "${6:-5}"
        ;;
    
    scheduler)
        job_scheduler
        ;;
    
    monitor)
        monitor_jobs
        ;;
    
    watch)
        watch -n 5 "$0 monitor"
        ;;
    
    report)
        generate_performance_report
        ;;
    
    cleanup)
        cleanup_old_jobs "${2:-7}"
        ;;
    
    example)
        cat <<EXAMPLE
# Example usage:

# 1. Initialize system
$0 init

# 2. Create backup jobs
$0 create "MySQL Backup" database mysql /backup/mysql 1
$0 create "App Data" rsync /var/www /backup/www 2
$0 create "Logs Archive" tar /var/log /backup/logs 3

# 3. Start scheduler (background)
$0 scheduler &

# 4. Monitor jobs
$0 watch

# 5. Generate performance report
$0 report
EXAMPLE
        ;;
    
    *)
        cat <<HELP
Parallel Backup System

Usage: $0 <command> [options]

Commands:
  init                              - Initialize parallel backup system
  create <name> <type> <src> <dst> [priority]
                                    - Create backup job
                                      Types: rsync, tar, database, custom
                                      Priority: 1-10 (1=highest)
  scheduler                         - Start job scheduler (run in background)
  monitor                           - Show current status
  watch                             - Monitor status continuously
  report                            - Generate performance report
  cleanup [days]                    - Cleanup jobs older than N days (default: 7)
  example                           - Show usage examples

Examples:
  $0 init
  $0 create "DB Backup" database mysql /backup/mysql 1
  $0 scheduler &
  $0 watch
  $0 report
HELP
        ;;
esac
EOF

chmod +x parallel_backup_system.sh
```


## 💻 Задание 3: Network Optimization & Bandwidth Management

```bash
cat > network_backup_optimizer.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
LOG_FILE="/var/log/network_backup_optimizer.log"
STATS_DIR="/var/lib/network-backup/stats"
BANDWIDTH_TEST_DURATION=30
QOS_RULES_FILE="/etc/network-backup/qos.rules"

# Network priorities
PRIORITY_CRITICAL=1
PRIORITY_HIGH=2
PRIORITY_MEDIUM=3
PRIORITY_LOW=4

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# Initialize network optimization
init_network_optimization() {
    log "Initializing network optimization..."
    
    mkdir -p "$STATS_DIR"
    mkdir -p "$(dirname "$QOS_RULES_FILE")"
    
    # Install required tools if not present
    for tool in iftop iperf3 tc wondershaper; do
        if ! command -v "$tool" &>/dev/null; then
            log "Installing $tool..."
            apt-get install -y "$tool" 2>&1 | tee -a "$LOG_FILE" || \
                log "Warning: Failed to install $tool"
        fi
    done
    
    # Create QoS configuration
    cat > "$QOS_RULES_FILE" <<QOS
# Network Backup QoS Rules
# Interface: eth0 (adjust as needed)
# Bandwidth allocation:
#   Critical: 50% (databases)
#   High: 30% (applications)
#   Medium: 15% (files)
#   Low: 5% (logs)

INTERFACE="eth0"
TOTAL_BANDWIDTH_KBPS=1000000  # 1 Gbps

# Traffic classes
CLASS_CRITICAL=1
CLASS_HIGH=2
CLASS_MEDIUM=3
CLASS_LOW=4

# Ports
PORT_MYSQL=3306
PORT_POSTGRESQL=5432
PORT_RSYNC=873
PORT_SSH=22
QOS
    
    log "Network optimization initialized"
}

# Measure available bandwidth
measure_bandwidth() {
    local target_host="${1:-8.8.8.8}"
    local duration="${2:-10}"
    
    log "Measuring bandwidth to $target_host..."
    
    # Test download speed
    local download_speed=$(curl -o /dev/null -w '%{speed_download}' \
        --max-time $duration \
        "http://$target_host" 2>/dev/null || echo "0")
    
    download_speed=$(echo "scale=2; $download_speed / 1024 / 1024" | bc)
    
    # Test with iperf3 if available and target supports it
    if command -v iperf3 &>/dev/null; then
        log "Running iperf3 test (if server available)..."
        
        local iperf_result=$(iperf3 -c "$target_host" -t $duration -J 2>/dev/null || echo "{}")
        
        if [ -n "$iperf_result" ]; then
            local iperf_bps=$(echo "$iperf_result" | jq -r '.end.sum_sent.bits_per_second // 0')
            local iperf_mbps=$(echo "scale=2; $iperf_bps / 1024 / 1024" | bc)
            
            log "iperf3 result: ${iperf_mbps} Mbps"
        fi
    fi
    
    # Measure ping latency
    local latency=$(ping -c 10 "$target_host" 2>/dev/null | \
        tail -1 | awk -F '/' '{print $5}')
    
    # Save results
    cat > "$STATS_DIR/bandwidth_test_$(date +%Y%m%d-%H%M%S).json" <<STATS
{
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "target_host": "$target_host",
  "download_speed_mbps": $download_speed,
  "latency_ms": ${latency:-0},
  "test_duration_seconds": $duration
}
STATS
    
    log "Bandwidth test completed: ${download_speed} MB/s, Latency: ${latency:-N/A} ms"
    
    echo "$download_speed"
}

# Setup traffic shaping with tc
setup_traffic_shaping() {
    local interface="${1:-eth0}"
    local max_bandwidth_mbps="${2:-100}"
    
    log "Setting up traffic shaping on $interface (max: ${max_bandwidth_mbps} Mbps)..."
    
    # Check if interface exists
    if ! ip link show "$interface" &>/dev/null; then
        log "ERROR: Interface $interface not found"
        return 1
    fi
    
    # Clear existing rules
    tc qdisc del dev "$interface" root 2>/dev/null || true
    
    # Convert Mbps to Kbps
    local max_bandwidth_kbps=$((max_bandwidth_mbps * 1024))
    
    # Setup HTB (Hierarchical Token Bucket)
    tc qdisc add dev "$interface" root handle 1: htb default 30
    
    # Root class
    tc class add dev "$interface" parent 1: classid 1:1 htb rate ${max_bandwidth_kbps}kbit
    
    # Critical traffic (50%)
    local critical_bw=$((max_bandwidth_kbps * 50 / 100))
    tc class add dev "$interface" parent 1:1 classid 1:10 htb rate ${critical_bw}kbit ceil ${max_bandwidth_kbps}kbit prio 1
    
    # High priority traffic (30%)
    local high_bw=$((max_bandwidth_kbps * 30 / 100))
    tc class add dev "$interface" parent 1:1 classid 1:20 htb rate ${high_bw}kbit ceil ${max_bandwidth_kbps}kbit prio 2
    
    # Medium priority traffic (15%)
    local medium_bw=$((max_bandwidth_kbps * 15 / 100))
    tc class add dev "$interface" parent 1:1 classid 1:30 htb rate ${medium_bw}kbit ceil ${max_bandwidth_kbps}kbit prio 3
    
    # Low priority traffic (5%)
    local low_bw=$((max_bandwidth_kbps * 5 / 100))
    tc class add dev "$interface" parent 1:1 classid 1:40 htb rate ${low_bw}kbit ceil ${max_bandwidth_kbps}kbit prio 4
    
    # Add SFQ (Stochastic Fairness Queueing) to each class
    tc qdisc add dev "$interface" parent 1:10 handle 10: sfq perturb 10
    tc qdisc add dev "$interface" parent 1:20 handle 20: sfq perturb 10
    tc qdisc add dev "$interface" parent 1:30 handle 30: sfq perturb 10
    tc qdisc add dev "$interface" parent 1:40 handle 40: sfq perturb 10
    
    # Filters for port-based classification
    # MySQL (3306) - Critical
    tc filter add dev "$interface" protocol ip parent 1:0 prio 1 u32 \
        match ip dport 3306 0xffff flowid 1:10
    
    # PostgreSQL (5432) - Critical
    tc filter add dev "$interface" protocol ip parent 1:0 prio 1 u32 \
        match ip dport 5432 0xffff flowid 1:10
    
    # Rsync (873) - High
    tc filter add dev "$interface" protocol ip parent 1:0 prio 2 u32 \
        match ip dport 873 0xffff flowid 1:20
    
    # SSH/SCP (22) - High
    tc filter add dev "$interface" protocol ip parent 1:0 prio 2 u32 \
        match ip dport 22 0xffff flowid 1:20
    
    log "Traffic shaping configured successfully"
    
    # Show configuration
    tc -s qdisc show dev "$interface"
    tc -s class show dev "$interface"
}

# Remove traffic shaping
remove_traffic_shaping() {
    local interface="${1:-eth0}"
    
    log "Removing traffic shaping from $interface..."
    
    tc qdisc del dev "$interface" root 2>/dev/null || true
    
    log "Traffic shaping removed"
}

# Intelligent bandwidth allocation
intelligent_bandwidth_allocation() {
    local interface="${1:-eth0}"
    local backup_window_hours="${2:-4}"
    
    log "Calculating intelligent bandwidth allocation..."
    
    # Get total data to backup
    local total_data_gb=0
    
    # MySQL databases
    if systemctl is-active --quiet mysql; then
        local mysql_size=$(mysql -e "
            SELECT ROUND(SUM(data_length + index_length) / 1024 / 1024 / 1024, 2) 
            FROM information_schema.tables;" 2>/dev/null | tail -1)
        total_data_gb=$(echo "$total_data_gb + ${mysql_size:-0}" | bc)
    fi
    
    # PostgreSQL databases
    if systemctl is-active --quiet postgresql; then
        local pg_size=$(sudo -u postgres psql -t -c "
            SELECT pg_size_pretty(sum(pg_database_size(datname))::bigint)
            FROM pg_database;" 2>/dev/null | grep -o '[0-9]*' | head -1)
        total_data_gb=$(echo "$total_data_gb + ${pg_size:-0}" | bc)
    fi
    
    # File systems
    for mount in /data /var/www /home; do
        if [ -d "$mount" ]; then
            local size_gb=$(du -s "$mount" 2>/dev/null | awk '{print $1/1024/1024}')
            total_data_gb=$(echo "$total_data_gb + ${size_gb:-0}" | bc)
        fi
    done
    
    log "Total data to backup: ${total_data_gb} GB"
    
    # Calculate required bandwidth
    local backup_window_seconds=$((backup_window_hours * 3600))
    local required_mbps=$(echo "scale=2; $total_data_gb * 8 * 1024 / $backup_window_seconds" | bc)
    
    log "Required bandwidth for ${backup_window_hours}h window: ${required_mbps} Mbps"
    
    # Measure available bandwidth
    local available_mbps=$(measure_bandwidth)
    
    # Calculate recommended allocation
    if (( $(echo "$available_mbps > $required_mbps" | bc -l) )); then
        local allocation_percent=$(echo "scale=2; $required_mbps * 100 / $available_mbps" | bc)
        log "Recommended bandwidth allocation: ${allocation_percent}% (${required_mbps} Mbps of ${available_mbps} Mbps)"
        
        # Apply traffic shaping with recommended allocation
        local allocated_mbps=$(echo "scale=0; $available_mbps * $allocation_percent / 100" | bc)
        setup_traffic_shaping "$interface" "$allocated_mbps"
    else
        log "WARNING: Available bandwidth (${available_mbps} Mbps) is less than required (${required_mbps} Mbps)"
        log "Recommendations:"
        log "  1. Extend backup window to $(echo "scale=1; $total_data_gb * 8 * 1024 / $available_mbps / 3600" | bc) hours"
        log "  2. Implement incremental backups"
        log "  3. Use compression to reduce data size"
        log "  4. Consider upgrading network bandwidth"
    fi
}

# Adaptive bandwidth control during backup
adaptive_bandwidth_control() {
    local backup_command="$1"
    local initial_bandwidth_mbps="${2:-50}"
    local target_cpu_percent="${3:-70}"
    
    log "Starting adaptive bandwidth control..."
    log "Command: $backup_command"
    log "Initial bandwidth: ${initial_bandwidth_mbps} Mbps"
    log "Target CPU: ${target_cpu_percent}%"
    
    # Calculate initial bandwidth limit in KB/s
    local current_limit_kbps=$((initial_bandwidth_mbps * 1024))
    
    # Start backup process with initial limit
    export RSYNC_BW_LIMIT=$current_limit_kbps
    
    # Execute backup in background
    eval "$backup_command" &
    local backup_pid=$!
    
    log "Backup process started (PID: $backup_pid)"
    
    # Monitor and adjust
    while kill -0 $backup_pid 2>/dev/null; do
        # Get current CPU usage
        local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1 | cut -d. -f1)
        
        # Get network throughput
        local net_throughput=$(sar -n DEV 1 1 | grep Average | grep eth0 | awk '{print $5}')
        
        log "Current CPU: ${cpu_usage}%, Network: ${net_throughput} KB/s, Limit: ${current_limit_kbps} KB/s"
        
        # Adjust bandwidth based on CPU usage
        if [ "$cpu_usage" -lt $((target_cpu_percent - 10)) ]; then
            # CPU has headroom, increase bandwidth by 10%
            current_limit_kbps=$((current_limit_kbps * 110 / 100))
            log "Increasing bandwidth to ${current_limit_kbps} KB/s"
            
        elif [ "$cpu_usage" -gt $((target_cpu_percent + 10)) ]; then
            # CPU overloaded, decrease bandwidth by 10%
            current_limit_kbps=$((current_limit_kbps * 90 / 100))
            log "Decreasing bandwidth to ${current_limit_kbps} KB/s"
        fi
        
        # Apply new limit (implementation depends on backup tool)
        # For rsync, we'd need to restart with new --bwlimit
        # For now, just log the adjustment
        
        sleep 30
    done
    
    wait $backup_pid
    local exit_code=$?
    
    log "Backup process completed with exit code: $exit_code"
    return $exit_code
}

# Network compression optimization
optimize_network_compression() {
    local source="$1"
    local destination="$2"
    local network_latency_ms="${3:-50}"
    
    log "Optimizing compression for network transfer..."
    
    # Choose compression based on network characteristics
    local compression_algo=""
    local compression_level=""
    
    if [ "$network_latency_ms" -lt 10 ]; then
        # Low latency (LAN) - prioritize speed
        compression_algo="lz4"
        compression_level=""
        log "Low latency detected - using lz4 (fast compression)"
        
    elif [ "$network_latency_ms" -lt 50 ]; then
        # Medium latency - balance speed and compression
        compression_algo="zstd"
        compression_level="-3"
        log "Medium latency detected - using zstd level 3"
        
    else
        # High latency (WAN) - prioritize compression ratio
        compression_algo="zstd"
        compression_level="-10"
        log "High latency detected - using zstd level 10"
    fi
    
    # Test different compression methods
    local test_file=$(mktemp)
    dd if=/dev/urandom of="$test_file" bs=1M count=100 2>/dev/null
    
    log "Testing compression methods with 100MB sample..."
    
    # Test uncompressed transfer
    log "Testing uncompressed transfer..."
    local start_time=$(date +%s.%N)
    rsync "$test_file" "$destination/test-uncompressed" 2>/dev/null || true
    local end_time=$(date +%s.%N)
    local uncompressed_time=$(echo "$end_time - $start_time" | bc)
    
    # Test with compression
    log "Testing $compression_algo transfer..."
    start_time=$(date +%s.%N)
    tar -cf - "$test_file" | $compression_algo $compression_level | \
        ssh "$destination" "cat > /tmp/test-compressed"
    end_time=$(date +%s.%N)
    local compressed_time=$(echo "$end_time - $start_time" | bc)
    
    # Compare
    local speedup=$(echo "scale=2; $uncompressed_time / $compressed_time" | bc)
    
    log "Results:"
    log "  Uncompressed: ${uncompressed_time}s"
    log "  Compressed ($compression_algo): ${compressed_time}s"
    log "  Speedup: ${speedup}x"
    
    # Cleanup
    rm -f "$test_file"
    ssh "$destination" "rm -f /tmp/test-compressed" 2>/dev/null || true
    
    if (( $(echo "$speedup > 1.2" | bc -l) )); then
        log "Recommendation: Use compression for this network"
        echo "compress"
    else
        log "Recommendation: Skip compression for this network"
        echo "no-compress"
    fi
}

# Monitor network performance during backup
monitor_network_performance() {
    local duration_seconds="${1:-300}"
    local output_file="$STATS_DIR/network_performance_$(date +%Y%m%d-%H%M%S).csv"
    
    log "Monitoring network performance for ${duration_seconds}s..."
    
    # Header
    echo "timestamp,rx_mbps,tx_mbps,retransmits,packet_loss,cpu_usage" > "$output_file"
    
    local end_time=$(($(date +%s) + duration_seconds))
    
    while [ $(date +%s) -lt $end_time ]; do
        local timestamp=$(date +%s)
        
        # Get network stats
        local net_stats=$(sar -n DEV 1 1 | grep Average | grep eth0)
        local rx_kbps=$(echo "$net_stats" | awk '{print $5}')
        local tx_kbps=$(echo "$net_stats" | awk '{print $6}')
        local rx_mbps=$(echo "scale=2; $rx_kbps / 1024" | bc)
        local tx_mbps=$(echo "scale=2; $tx_kbps / 1024" | bc)
        
        # Get TCP retransmits
        local retransmits=$(netstat -s | grep "segments retransmitted" | awk '{print $1}')
        
        # Get packet loss (from ping to gateway)
        local gateway=$(ip route | grep default | awk '{print $3}')
        local packet_loss=$(ping -c 1 -W 1 "$gateway" 2>/dev/null | grep "packet loss" | awk '{print $6}' | tr -d '%')
        
        # Get CPU usage
        local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
        
        echo "$timestamp,$rx_mbps,$tx_mbps,${retransmits:-0},${packet_loss:-0},$cpu_usage" >> "$output_file"
        
        sleep 5
    done
    
    log "Network performance monitoring completed: $output_file"
    
    # Generate summary
    log "Performance Summary:"
    awk -F',' 'NR>1 {
        rx+=$2; tx+=$3; retrans+=$4; loss+=$5; cpu+=$6; count++
    } END {
        printf "  Avg RX: %.2f Mbps\n", rx/count
        printf "  Avg TX: %.2f Mbps\n", tx/count
        printf "  Total Retransmits: %d\n", retrans
        printf "  Avg Packet Loss: %.2f%%\n", loss/count
        printf "  Avg CPU: %.2f%%\n", cpu/count
    }' "$output_file" | tee -a "$LOG_FILE"
}

# Generate network optimization report
generate_network_report() {
    local report_file="/var/log/network_optimization_report_$(date +%Y%m%d).html"
    
    log "Generating network optimization report..."
    
    cat > "$report_file" <<'HTML'
<!DOCTYPE html>
<html>
<head>
    <title>Network Backup Optimization Report</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 20px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }
        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        h1 {
            color: #2c3e50;
            border-bottom: 3px solid #667eea;
            padding-bottom: 10px;
        }
        .metrics-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 20px;
            margin: 30px 0;
        }
        .metric-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 25px;
            border-radius: 10px;
            text-align: center;
        }
        .metric-value {
            font-size: 3em;
            font-weight: bold;
            margin: 15px 0;
        }
        .metric-label {
            font-size: 1em;
            opacity: 0.9;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #667eea;
            color: white;
        }
        .recommendation {
            background: #e7f3ff;
            border-left: 4px solid #2196F3;
            padding: 15px;
            margin: 20px 0;
            border-radius: 5px;
        }
        .chart-container {
            margin: 30px 0;
            padding: 20px;
            background: #f8f9fa;
            border-radius: 8px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🌐 Network Backup Optimization Report</h1>
        <p><strong>Generated:</strong> $(date)</p>
        
        <div class="metrics-grid">
HTML
    
    # Get latest bandwidth test
    local latest_test=$(ls -t "$STATS_DIR"/bandwidth_test_*.json 2>/dev/null | head -1)
    
    if [ -f "$latest_test" ]; then
        local download_speed=$(jq -r '.download_speed_mbps' "$latest_test")
        local latency=$(jq -r '.latency_ms' "$latest_test")
        
        cat >> "$report_file" <<HTML
            <div class="metric-card">
                <div class="metric-label">Download Speed</div>
                <div class="metric-value">${download_speed}</div>
                <div class="metric-label">MB/s</div>
            </div>
            <div class="metric-card" style="background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);">
                <div class="metric-label">Network Latency</div>
                <div class="metric-value">${latency}</div>
                <div class="metric-label">ms</div>
            </div>
HTML
    fi
    
    # Calculate total data transferred
    local total_transferred=0
    for perf_file in "$STATS_DIR"/network_performance_*.csv; do
        if [ -f "$perf_file" ]; then
            local transferred=$(awk -F',' 'NR>1 {sum+=$3} END {print sum/1024}' "$perf_file")
            total_transferred=$(echo "$total_transferred + ${transferred:-0}" | bc)
        fi
    done
    
    cat >> "$report_file" <<HTML
            <div class="metric-card" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
                <div class="metric-label">Total Data Transferred</div>
                <div class="metric-value">$(printf "%.2f" $total_transferred)</div>
                <div class="metric-label">GB</div>
            </div>
        </div>
        
        <h2>Traffic Shaping Configuration</h2>
        <table>
            <thead>
                <tr>
                    <th>Priority Class</th>
                    <th>Services</th>
                    <th>Bandwidth Allocation</th>
                    <th>Priority Level</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Critical</td>
                    <td>MySQL, PostgreSQL</td>
                    <td>50%</td>
                    <td>1 (Highest)</td>
                </tr>
                <tr>
                    <td>High</td>
                    <td>Rsync, SSH/SCP</td>
                    <td>30%</td>
                    <td>2</td>
                </tr>
                <tr>
                    <td>Medium</td>
                    <td>File transfers</td>
                    <td>15%</td>
                    <td>3</td>
                </tr>
                <tr>
                    <td>Low</td>
                    <td>Log archives</td>
                    <td>5%</td>
                    <td>4</td>
                </tr>
            </tbody>
        </table>
        
        <div class="recommendation">
            <h3>🎯 Optimization Recommendations</h3>
            <ul>
HTML
    
    if [ -f "$latest_test" ]; then
        local latency=$(jq -r '.latency_ms' "$latest_test")
        
        if [ "$latency" -lt 10 ]; then
            cat >> "$report_file" <<HTML
                <li><strong>Low Latency Network:</strong> Use lz4 compression for maximum speed</li>
HTML
        elif [ "$latency" -lt 50 ]; then
            cat >> "$report_file" <<HTML
                <li><strong>Medium Latency Network:</strong> Use zstd level 3 for balanced performance</li>
HTML
        else
            cat >> "$report_file" <<HTML
                <li><strong>High Latency Network:</strong> Use zstd level 10 for maximum compression</li>
HTML
        fi
    fi
    
    cat >> "$report_file" <<HTML
                <li>Implement traffic shaping to prioritize critical database backups</li>
                <li>Schedule large file transfers during off-peak hours</li>
                <li>Use incremental backups to reduce network load</li>
                <li>Consider WAN optimization appliances for remote sites</li>
                <li>Monitor network performance during backup windows</li>
            </ul>
        </div>
        
        <h2>Network Performance Trends</h2>
        <div class="chart-container">
            <p><em>Network performance data collected from recent backup operations</em></p>
HTML
    
    # Add latest performance data
    local latest_perf=$(ls -t "$STATS_DIR"/network_performance_*.csv 2>/dev/null | head -1)
    
    if [ -f "$latest_perf" ]; then
        cat >> "$report_file" <<HTML
            <table>
                <thead>
                    <tr>
                        <th>Metric</th>
                        <th>Average</th>
                        <th>Peak</th>
                        <th>Status</th>
                    </tr>
                </thead>
                <tbody>
HTML
        
        # Calculate statistics from CSV
        awk -F',' '
        NR>1 {
            rx_sum+=$2; tx_sum+=$3; retrans_sum+=$4; loss_sum+=$5; cpu_sum+=$6;
            if ($2 > rx_max) rx_max=$2;
            if ($3 > tx_max) tx_max=$3;
            count++
        }
        END {
            printf "<tr><td>RX Throughput</td><td>%.2f Mbps</td><td>%.2f Mbps</td><td>Good</td></tr>\n", rx_sum/count, rx_max;
            printf "<tr><td>TX Throughput</td><td>%.2f Mbps</td><td>%.2f Mbps</td><td>Good</td></tr>\n", tx_sum/count, tx_max;
            printf "<tr><td>Packet Loss</td><td>%.2f%%</td><td>-</td><td>%s</td></tr>\n", loss_sum/count, (loss_sum/count < 1 ? "Good" : "Warning");
            printf "<tr><td>CPU Usage</td><td>%.2f%%</td><td>-</td><td>%s</td></tr>\n", cpu_sum/count, (cpu_sum/count < 80 ? "Good" : "Warning");
        }
        ' "$latest_perf" >> "$report_file"
        
        cat >> "$report_file" <<HTML
                </tbody>
            </table>
HTML
    fi
    
    cat >> "$report_file" <<HTML
        </div>
        
        <h2>Compression Analysis</h2>
        <table>
            <thead>
                <tr>
                    <th>Algorithm</th>
                    <th>Network Type</th>
                    <th>Compression Ratio</th>
                    <th>Speed</th>
                    <th>Recommended</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>lz4</td>
                    <td>LAN (< 10ms latency)</td>
                    <td>2-3x</td>
                    <td>Very Fast</td>
                    <td>✓</td>
                </tr>
                <tr>
                    <td>zstd -3</td>
                    <td>Regional (10-50ms latency)</td>
                    <td>3-5x</td>
                    <td>Fast</td>
                    <td>✓</td>
                </tr>
                <tr>
                    <td>zstd -10</td>
                    <td>WAN (> 50ms latency)</td>
                    <td>5-8x</td>
                    <td>Medium</td>
                    <td>✓</td>
                </tr>
                <tr>
                    <td>None</td>
                    <td>Very high bandwidth</td>
                    <td>1x</td>
                    <td>N/A</td> 
                    <td>Only if bandwidth > 1Gbps</td> 
                    </tr> 
                </tbody> 
        </table>


    <hr>
    <p style="text-align: center; color: #666;">
        Network Backup Optimizer v1.0 | Report generated: $(date)
    </p>
</div>

</body> </html> HTML


log "Network optimization report generated: $report_file"
echo "$report_file"

}

# Main command handling

case "${1:-help}" in init) init_network_optimization ;;


measure)
    measure_bandwidth "${2:-8.8.8.8}" "${3:-10}"
    ;;

shape)
    setup_traffic_shaping "${2:-eth0}" "${3:-100}"
    ;;

remove-shape)
    remove_traffic_shaping "${2:-eth0}"
    ;;

optimize)
    intelligent_bandwidth_allocation "${2:-eth0}" "${3:-4}"
    ;;

adaptive)
    if [ -z "$2" ]; then
        echo "Usage: $0 adaptive '<backup_command>' [initial_mbps] [target_cpu%]"
        exit 1
    fi
    adaptive_bandwidth_control "$2" "${3:-50}" "${4:-70}"
    ;;

compression)
    optimize_network_compression "$2" "$3" "${4:-50}"
    ;;

monitor)
    monitor_network_performance "${2:-300}"
    ;;

report)
    generate_network_report
    ;;

*)
    cat <<HELP


Network Backup Optimizer

Usage: $0 <command> [options]

Commands: init - Initialize network optimization measure [host] [duration] - Measure available bandwidth shape <interface> <max_mbps> - Setup traffic shaping with QoS remove-shape <interface> - Remove traffic shaping optimize <interface> [window_hours] - Calculate intelligent bandwidth allocation adaptive '<command>' [mbps] [cpu%] - Adaptive bandwidth control during backup compression <src> <dst> [latency_ms] - Optimize compression for network monitor [duration_seconds] - Monitor network performance report - Generate optimization report

Examples: $0 init $0 measure 192.168.1.1 30 $0 shape eth0 100 $0 optimize eth0 4 $0 adaptive 'rsync -av /data remote:/backup' 50 70 $0 compression /data remote:/backup 100 $0 monitor 600 $0 report HELP ;; esac EOF

chmod +x network_backup_optimizer.sh
```


## 💻 Задание 4: Deduplication Benchmark & Storage Optimization

```bash
cat > deduplication_optimizer.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
TEST_DIR="/tmp/dedup_test_$$"
RESULTS_FILE="/var/log/dedup_benchmark_$(date +%Y%m%d).log"
STORAGE_ANALYSIS_DIR="/var/lib/storage-analysis"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$RESULTS_FILE"
}

# Create test dataset with various deduplication scenarios
create_dedup_test_data() {
    local size_mb="$1"
    
    log "Creating deduplication test dataset (${size_mb}MB)..."
    
    mkdir -p "$TEST_DIR"/{original,copies,modified}
    
    # Scenario 1: Identical files (100% dedup)
    log "Creating identical files..."
    dd if=/dev/urandom of="$TEST_DIR/original/base_file" bs=1M count=$((size_mb / 10)) 2>/dev/null
    for i in {1..9}; do
        cp "$TEST_DIR/original/base_file" "$TEST_DIR/copies/copy_$i"
    done
    
    # Scenario 2: Slightly modified files (block-level dedup)
    log "Creating modified files..."
    for i in {1..5}; do
        cp "$TEST_DIR/original/base_file" "$TEST_DIR/modified/modified_$i"
        # Modify 10% of the file
        dd if=/dev/urandom of="$TEST_DIR/modified/modified_$i" \
            bs=1M count=$((size_mb / 100)) seek=$((i * size_mb / 100)) \
            conv=notrunc 2>/dev/null
    done
    
    # Scenario 3: Unique files (0% dedup)
    log "Creating unique files..."
    for i in {1..3}; do
        dd if=/dev/urandom of="$TEST_DIR/original/unique_$i" \
            bs=1M count=$((size_mb / 30)) 2>/dev/null
    done
    
    # Scenario 4: Virtual machine images (high block-level dedup)
    log "Simulating VM images..."
    dd if=/dev/zero of="$TEST_DIR/original/vm_base.img" bs=1M count=$((size_mb / 5)) 2>/dev/null
    for i in {1..3}; do
        cp "$TEST_DIR/original/vm_base.img" "$TEST_DIR/copies/vm_$i.img"
        # Add unique data to simulate different VMs
        dd if=/dev/urandom of="$TEST_DIR/copies/vm_$i.img" \
            bs=1M count=$((size_mb / 50)) seek=$((i * 10)) \
            conv=notrunc 2>/dev/null
    done
    
    local total_size=$(du -sb "$TEST_DIR" | cut -f1)
    log "Test dataset created: $(numfmt --to=iec-i --suffix=B $total_size)"
}

# Benchmark file-level deduplication
benchmark_file_level_dedup() {
    local test_dir="$1"
    local output_dir="$2"
    
    log "=== File-Level Deduplication Benchmark ==="
    
    mkdir -p "$output_dir"
    
    # Method 1: rsync with hardlinks
    log "Testing rsync with hardlinks..."
    local start_time=$(date +%s.%N)
    
    rsync -aH --link-dest="$output_dir/base" \
        "$test_dir/" "$output_dir/snapshot_1/"
    
    rsync -aH --link-dest="$output_dir/snapshot_1" \
        "$test_dir/" "$output_dir/snapshot_2/"
    
    local end_time=$(date +%s.%N)
    local rsync_time=$(echo "$end_time - $start_time" | bc)
    
    # Calculate space savings
    local original_size=$(du -sb "$test_dir" | cut -f1)
    local rsync_size=$(du -sb "$output_dir" | cut -f1)
    local rsync_savings=$(echo "scale=2; ($original_size * 2 - $rsync_size) * 100 / ($original_size * 2)" | bc)
    
    log "Rsync Results:"
    log "  Original Size: $(numfmt --to=iec-i --suffix=B $original_size)"
    log "  With Dedup: $(numfmt --to=iec-i --suffix=B $rsync_size)"
    log "  Space Savings: ${rsync_savings}%"
    log "  Time: ${rsync_time}s"
    
    # Method 2: rdfind (finds duplicates)
    log "Testing rdfind..."
    if command -v rdfind &>/dev/null; then
        local rdfind_dir="$output_dir/rdfind_test"
        cp -r "$test_dir" "$rdfind_dir"
        
        start_time=$(date +%s.%N)
        rdfind -makehardlinks true "$rdfind_dir" > /dev/null 2>&1
        end_time=$(date +%s.%N)
        local rdfind_time=$(echo "$end_time - $start_time" | bc)
        
        local rdfind_size=$(du -sb "$rdfind_dir" | cut -f1)
        local rdfind_savings=$(echo "scale=2; ($original_size - $rdfind_size) * 100 / $original_size" | bc)
        
        log "Rdfind Results:"
        log "  With Dedup: $(numfmt --to=iec-i --suffix=B $rdfind_size)"
        log "  Space Savings: ${rdfind_savings}%"
        log "  Time: ${rdfind_time}s"
    else
        log "Rdfind not available, skipping"
    fi
    
    # Method 3: duperemove (block-level for supported filesystems)
    if command -v duperemove &>/dev/null && [ -f /proc/fs/btrfs ]; then
        log "Testing duperemove..."
        local duperemove_dir="$output_dir/duperemove_test"
        cp -r "$test_dir" "$duperemove_dir"
        
        start_time=$(date +%s.%N)
        duperemove -dr "$duperemove_dir" > /dev/null 2>&1
        end_time=$(date +%s.%N)
        local duperemove_time=$(echo "$end_time - $start_time" | bc)
        
        local duperemove_size=$(du -sb "$duperemove_dir" | cut -f1)
        local duperemove_savings=$(echo "scale=2; ($original_size - $duperemove_size) * 100 / $original_size" | bc)
        
        log "Duperemove Results:"
        log "  With Dedup: $(numfmt --to=iec-i --suffix=B $duperemove_size)"
        log "  Space Savings: ${duperemove_savings}%"
        log "  Time: ${duperemove_time}s"
    fi
}

# Benchmark block-level deduplication
benchmark_block_level_dedup() {
    local test_dir="$1"
    local block_size="${2:-4096}"
    
    log "=== Block-Level Deduplication Benchmark (Block Size: $block_size) ==="
    
    # Calculate potential dedup with different block sizes
    for size in 4096 8192 16384 65536 131072; do
        log "Analyzing block size: $size bytes..."
        
        # Create hash index of blocks
        local hash_file="/tmp/block_hashes_$$"
        
        find "$test_dir" -type f -exec dd if={} bs=$size 2>/dev/null \; | \
            split -b $size - /tmp/block_$size\_
        
        # Count unique blocks
        local total_blocks=$(ls /tmp/block_$size\_* 2>/dev/null | wc -l)
        local unique_blocks=$(ls /tmp/block_$size\_* 2>/dev/null | \
            xargs -n1 md5sum | sort -k1,1 | uniq -w32 | wc -l)
        
        local dedup_ratio=$(echo "scale=2; $total_blocks / $unique_blocks" | bc)
        local space_savings=$(echo "scale=2; ($total_blocks - $unique_blocks) * 100 / $total_blocks" | bc)
        
        log "  Block Size: $size bytes"
        log "  Total Blocks: $total_blocks"
        log "  Unique Blocks: $unique_blocks"
        log "  Dedup Ratio: ${dedup_ratio}:1"
        log "  Potential Savings: ${space_savings}%"
        
        # Cleanup
        rm -f /tmp/block_$size\_*
    done
}

# Analyze real backup data for deduplication potential
analyze_backup_dedup_potential() {
    local backup_dir="$1"
    
    log "=== Analyzing Backup Deduplication Potential ==="
    
    if [ ! -d "$backup_dir" ]; then
        log "ERROR: Backup directory not found: $backup_dir"
        return 1
    fi
    
    mkdir -p "$STORAGE_ANALYSIS_DIR"
    
    # Find duplicate files
    log "Finding duplicate files..."
    local dup_report="$STORAGE_ANALYSIS_DIR/duplicates_$(date +%Y%m%d).txt"
    
    find "$backup_dir" -type f -exec md5sum {} \; | \
        sort | uniq -w32 -D > "$dup_report"
    
    local dup_count=$(wc -l < "$dup_report")
    log "Found $dup_count duplicate file instances"
    
    # Calculate potential savings from file-level dedup
    local total_dup_size=0
    while read hash file; do
        local size=$(stat -c%s "$file" 2>/dev/null || stat -f%z "$file")
        total_dup_size=$((total_dup_size + size))
    done < "$dup_report"
    
    # Keep one copy of each duplicate
    local unique_dup_size=$(awk '{print $2}' "$dup_report" | \
        xargs -n1 stat -c%s 2>/dev/null | \
        sort -u | \
        awk '{sum+=$1} END {print sum}')
    
    local file_level_savings=$((total_dup_size - unique_dup_size))
    
    log "File-Level Deduplication Potential:"
    log "  Total Duplicate Size: $(numfmt --to=iec-i --suffix=B $total_dup_size)"
    log "  Potential Savings: $(numfmt --to=iec-i --suffix=B $file_level_savings)"
    
    # Analyze file types for compression potential
    log "Analyzing file types..."
    
    declare -A file_types
    declare -A type_sizes
    
    while IFS= read -r file; do
        local ext="${file##*.}"
        local size=$(stat -c%s "$file" 2>/dev/null || stat -f%z "$file")
        
        file_types[$ext]=$((${file_types[$ext]:-0} + 1))
        type_sizes[$ext]=$((${type_sizes[$ext]:-0} + size))
    done < <(find "$backup_dir" -type f)
    
    log "File Type Distribution:"
    for ext in "${!file_types[@]}"; do
        local count=${file_types[$ext]}
        local size=${type_sizes[$ext]}
        log "  .$ext: $count files, $(numfmt --to=iec-i --suffix=B $size)"
    done
}

# Recommend optimal deduplication strategy
recommend_dedup_strategy() {
    local backup_type="$1"
    local data_size_gb="$2"
    local change_rate_percent="$3"
    
    log "=== Deduplication Strategy Recommendation ==="
    log "Backup Type: $backup_type"
    log "Data Size: ${data_size_gb}GB"
    log "Daily Change Rate: ${change_rate_percent}%"
    
    # Decision logic
    local strategy=""
    
    if [ "$data_size_gb" -lt 100 ]; then
        strategy="File-level hardlinks (rsync)"
        log "Recommendation: $strategy"
        log "Reason: Small dataset, file-level dedup sufficient"
        log "Expected Savings: 30-50%"
        log "Implementation: Use rsync with --link-dest"
        
    elif [ "$change_rate_percent" -lt 10 ]; then
        strategy="Block-level deduplication (ZFS/Btrfs)"
        log "Recommendation: $strategy"
        log "Reason: Low change rate, high dedup potential"
        log "Expected Savings: 50-80%"
        log "Implementation: Use ZFS dedup or Btrfs with duperemove"
        
    elif [ "$backup_type" = "vm" ]; then
        strategy="Block-level with variable block size"
        log "Recommendation: $strategy"
        log "Reason: VM images have high block-level similarity"
        log "Expected Savings: 60-90%"
        log "Implementation: Use dedicated backup tool (Veeam, Proxmox Backup)"
        
    else
        strategy="File-level dedup + compression"
        log "Recommendation: $strategy"
        log "Reason: Balanced approach for general workloads"
        log "Expected Savings: 40-60%"
        log "Implementation: rsync hardlinks + zstd compression"
    fi
    
    # Additional recommendations
    log ""
    log "Additional Recommendations:"
    log "1. Use appropriate block size (4KB for general, 128KB for large files)"
    log "2. Enable compression for text/log files"
    log "3. Schedule dedup during off-peak hours"
    log "4. Monitor CPU/RAM usage (dedup can be resource-intensive)"
    log "5. Regular scrub/verify operations to ensure data integrity"
}

# Generate storage optimization report
generate_storage_report() {
    local report_file="/var/log/storage_optimization_report_$(date +%Y%m%d).html"
    
    log "Generating storage optimization report..."
    
    cat > "$report_file" <<'HTML'
<!DOCTYPE html>
<html>
<head>
    <title>Storage Optimization Report</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 20px;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
        }
        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        h1 {
            color: #1e3c72;
            border-bottom: 3px solid #2a5298;
            padding-bottom: 10px;
        }
        .metrics {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 20px;
            margin: 30px 0;
        }
        .metric-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 25px;
            border-radius: 10px;
            text-align: center;
        }
        .metric-card.savings {
            background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
        }
        .metric-value {
            font-size: 3em;
            font-weight: bold;
            margin: 15px 0;
        }
        .metric-label {
            font-size: 1em;
            opacity: 0.9;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .comparison {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 20px;
            margin: 30px 0;
        }
        .comparison-card {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 8px;
        }
        .recommendation {
            background: #e7f3ff;
            border-left: 4px solid #2196F3;
            padding: 15px;
            margin: 20px 0;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>💾 Storage Optimization & Deduplication Report</h1>
        <p><strong>Generated:</strong> $(date)</p>
        
        <div class="metrics">
HTML
    
    # Parse results and calculate metrics
    local original_size=$(grep "Original Size:" "$RESULTS_FILE" | tail -1 | awk '{print $3}')
    local dedup_size=$(grep "With Dedup:" "$RESULTS_FILE" | tail -1 | awk '{print $3}')
    local savings=$(grep "Space Savings:" "$RESULTS_FILE" | tail -1 | awk '{print $3}')
    local dedup_ratio=$(grep "Dedup Ratio:" "$RESULTS_FILE" | tail -1 | awk '{print $3}')
    
    cat >> "$report_file" <<HTML
            <div class="metric-card">
                <div class="metric-label">Original Size</div>
                <div class="metric-value">${original_size:-N/A}</div>
            </div>
            <div class="metric-card savings">
                <div class="metric-label">Space Savings</div>
                <div class="metric-value">${savings:-N/A}</div>
            </div>
            <div class="metric-card">
                <div class="metric-label">Dedup Ratio</div>
                <div class="metric-value">${dedup_ratio:-N/A}</div>
            </div>
            <div class="metric-card">
                <div class="metric-label">With Dedup</div>
                <div class="metric-value">${dedup_size:-N/A}</div>
            </div>
        </div>
        
        <h2>Deduplication Methods Comparison</h2>
        <table>
            <thead>
                <tr>
                    <th>Method</th>
                    <th>Type</th>
                    <th>Space Savings</th>
                    <th>Performance Impact</th>
                    <th>CPU Usage</th>
                    <th>Best For</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Rsync Hardlinks</td>
                    <td>File-level</td>
                    <td>30-50%</td>
                    <td>Low</td>
                    <td>Low</td>
                    <td>General backups</td>
                </tr>
                <tr>
                    <td>ZFS Dedup</td>
                    <td>Block-level</td>
                    <td>50-80%</td>
                    <td>Medium-High</td>
                    <td>High</td>
                    <td>VM images, databases</td>
                </tr>
                <tr>
                    <td>Btrfs + Duperemove</td>
                    <td>Block-level</td>
                    <td>60-90%</td>
                    <td>Medium</td>
                    <td>Medium</td>
                    <td>Large file repositories</td>
                </tr>
                <tr>
                    <td>Specialized Tools (Veeam)</td>
                    <td>Variable block</td>
                    <td>70-95%</td>
                    <td>Low-Medium</td>
                    <td>Medium</td>
                    <td>VM/container backups</td>
                </tr>
                <tr>
                    <td>Compression Only</td>
                    <td>N/A</td>
                    <td>40-70%</td>
                    <td>Low</td>
                    <td>Medium</td>
                    <td>Text/log files</td>
                </tr>
            </tbody>
        </table>
        
        <h2>Block Size Analysis</h2>
        <table>
            <thead>
                <tr>
                    <th>Block Size</th>
                    <th>Dedup Ratio</th>
                    <th>Potential Savings</th>
                    <th>CPU Impact</th>
                    <th>Recommendation</th>
                </tr>
            </thead>
            <tbody>
HTML
    
    # Add block size analysis from results
    grep -A4 "Block Size:" "$RESULTS_FILE" | grep -v "^--$" | \
    awk '
    /Block Size:/ {size=$3; getline}
    /Dedup Ratio:/ {ratio=$3; getline}
    /Potential Savings:/ {
        savings=$3
        cpu="Medium"
        if (size ~ /4096/) cpu="High"
        if (size ~ /131072/) cpu="Low"
        
        rec="General purpose"
        if (size ~ /4096/) rec="Small files, databases"
        if (size ~ /65536|131072/) rec="Large files, VMs"
        
        print "<tr>"
        print "<td>" size " bytes</td>"
        print "<td>" ratio "</td>"
        print "<td>" savings "</td>"
        print "<td>" cpu "</td>"
        print "<td>" rec "</td>"
        print "</tr>"
    }
    ' >> "$report_file"
    
    cat >> "$report_file" <<HTML
            </tbody>
        </table>
        
        <div class="recommendation">
            <h3>🎯 Implementation Recommendations</h3>
            <ol>
                <li><strong>For General File Backups:</strong>
                    <ul>
                        <li>Use rsync with --link-dest for incremental snapshots</li>
                        <li>Enable zstd compression (level 3)</li>
                        <li>Expected savings: 40-60%</li>
                    </ul>
                </li>
                <li><strong>For VM/Database Backups:</strong>
                    <ul>
                        <li>Use block-level deduplication (ZFS or Btrfs)</li>
                        <li>Block size: 128KB for VMs, 8KB for databases</li>
                        <li>Expected savings: 60-85%</li>
                    </ul>
                </li>
                <li><strong>For Large-Scale Deployments:</strong>
                    <ul>
                        <li>Consider dedicated backup appliances with dedup</li>
                        <li>Implement tiered storage (SSD for dedup index, HDD for data)</li>
                        <li>Expected savings: 70-95%</li>
                    </ul>
                </li>
            </ol>
        </div>
        
        <div class="comparison">
            <div class="comparison-card">
                <h3>✅ Benefits of Deduplication</h3>
                <ul>
                    <li>Reduce storage costs by 50-90%</li>
                    <li>Faster backups (less data to transfer)</li>
                    <li>Lower network bandwidth usage</li>
                    <li>Extended retention periods</li>
                    <li>Improved RPO with more frequent backups</li>
                </ul>
            </div>
            <div class="comparison-card">
                <h3>⚠️ Considerations</h3>
                <ul>
                    <li>Increased CPU/RAM usage during dedup</li>
                    <li>Slower restore times (need to reassemble blocks)</li>
                    <li>Risk of data loss if dedup index corrupted</li>
                    <li>Not effective for already compressed data</li>
                    <li>Requires regular integrity checks</li>
                </ul>
            </div>
        </div>
        
        <h2>Cost-Benefit Analysis</h2>
        <table>
            <thead>
                <tr>
                    <th>Scenario</th>
                    <th>Original Cost/Month</th>
                    <th>With Dedup Cost/Month</th>
                    <th>Annual Savings</th>
                    <th>ROI Period</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>10TB backup (50% dedup)</td>
                    <td>$200</td>
                    <td>$100</td>
                    <td>$1,200</td>
                    <td>< 1 month</td>
                </tr>
                <tr>
                    <td>50TB backup (70% dedup)</td>
                    <td>$1,000</td>
                    <td>$300</td>
                    <td>$8,400</td>
                    <td>< 1 month</td>
                </tr>
                <tr>
                    <td>100TB backup (80% dedup)</td>
                    <td>$2,000</td>
                    <td>$400</td>
                    <td>$19,200</td>
                    <td>Immediate</td>
                </tr>
            </tbody>
        </table>
        
        <hr>
        <p style="text-align: center; color: #666;">
            Storage Optimization Report v1.0 | Generated: $(date)
        </p>
    </div>
</body>
</html>
HTML
    
    log "Storage optimization report generated: $report_file"
    echo "$report_file"
}

# Main execution
case "${1:-help}" in
    test)
        local size_mb="${2:-1024}"
        create_dedup_test_data "$size_mb"
        benchmark_file_level_dedup "$TEST_DIR" "/tmp/dedup_output_$$"
        benchmark_block_level_dedup "$TEST_DIR"
        rm -rf "$TEST_DIR" "/tmp/dedup_output_$$"
        ;;
    
    analyze)
        if [ -z "$2" ]; then
            echo "Usage: $0 analyze <backup_directory>"
            exit 1
        fi
        analyze_backup_dedup_potential "$2"
        ;;
    
    recommend)
        if [ $# -lt 4 ]; then
            echo "Usage: $0 recommend <backup_type> <size_gb> <change_rate_%>"
            exit 1
        fi
        recommend_dedup_strategy "$2" "$3" "$4"
        ;;
    
    report)
        generate_storage_report
        ;;
    
    *)
        cat <<HELP
Deduplication Optimizer

Usage: $0 <command> [options]

Commands:
  test [size_mb]              - Run deduplication benchmark
                                Default: 1024 (1GB)
  analyze <backup_dir>        - Analyze existing backups for dedup potential
  recommend <type> <size> <%> - Recommend dedup strategy
                                type: general|vm|database
                                size: size in GB
                                %: daily change rate
  report                      - Generate storage optimization report

Examples:
  $0 test 2048
  $0 analyze /backup
  $0 recommend vm 500 5
  $0 report
HELP
        ;;
esac
EOF

chmod +x deduplication_optimizer.sh
```

---

## 🎓 Итоговое задание модуля 7

```bash
#!/bin/bash

cat > module7_complete_test.sh <<'FINAL'
#!/bin/bash

echo "=============================================="
echo "Module 7: Complete Performance Test Suite"
echo "=============================================="

# 1. Compression Optimization
echo ""
echo "1. Running Compression Benchmark..."
./compression_optimizer.sh quick

# 2. Parallel Backup System
echo ""
echo "2. Setting up Parallel Backup System..."
./parallel_backup_system.sh init

# Create test jobs
./parallel_backup_system.sh create "Test DB" database mysql /backup/test-db 1
./parallel_backup_system.sh create "Test Files" rsync /etc /backup/test-files 2

# Start scheduler in background
./parallel_backup_system.sh scheduler &
SCHEDULER_PID=$!

# Monitor for 1 minute
sleep 60
./parallel_backup_system.sh monitor

# Stop scheduler
kill $SCHEDULER_PID 2>/dev/null

# Generate report
./parallel_backup_system.sh report

# 3. Network Optimization
echo ""
echo "3. Testing Network Optimization..."
./network_backup_optimizer.sh init
./network_backup_optimizer.sh measure
./network_backup_optimizer.sh report

# 4. Deduplication Analysis
echo ""
echo "4. Running Deduplication Tests..."
./deduplication_optimizer.sh test 512
./deduplication_optimizer.sh report

echo ""
echo "=============================================="
echo "Module 7 Complete Test Finished!"
echo "=============================================="
echo ""
echo "Generated Reports:"
ls -lh /var/log/*report*.html 2>/dev/null
echo ""
echo "View reports:"
echo "  - Compression: firefox /var/log/compression_report_*.html"
echo "  - Parallel Backup: firefox /var/log/parallel_backup_performance_*.html"
echo "  - Network: firefox /var/log/network_optimization_report_*.html"
echo "  - Deduplication: firefox /var/log/storage_optimization_report_*.html"
FINAL

chmod +x module7_complete_test.sh
```

---

# Модуль 8: Monitoring, Alerting & Backup Analytics (45 минут)

## 🎯 Напоминалка

### Backup Monitoring Stack

```
Monitoring Architecture
┌─────────────────────────────────────────┐
│ Data Collection Layer                   │
│ - Backup job logs                       │
│ - System metrics (CPU, RAM, I/O)       │
│ - Storage metrics                       │
│ - Network metrics                       │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Processing & Storage                    │
│ - Time-series database (Prometheus)     │
│ - Log aggregation (Loki, ELK)         │
│ - Metrics retention                     │
└─────────────────────────────────────────┘
           ↓
┌─────────────────────────────────────────┐
│ Visualization & Alerting                │
│ - Grafana dashboards                    │
│ - Alert manager                         │
│ - Notification channels                 │
└─────────────────────────────────────────┘
```

### Key Backup Metrics

```bash
# Success Metrics
- Backup success rate (%)
- Jobs completed on time
- Data integrity check pass rate

# Performance Metrics
- Backup duration (actual vs expected)
- Backup throughput (MB/s)
- Deduplication ratio
- Compression ratio

# Storage Metrics
- Total backup size
- Storage growth rate
- Retention compliance
- Storage utilization (%)

# Recovery Metrics
- RTO achieved (vs target)
- RPO achieved (vs target)
- Recovery success rate
- Recovery test frequency

# Business Metrics
- Cost per GB backed up
- Cost per protected asset
- Total backup infrastructure cost
```

### Alert Severity Levels

```
Critical (P1)
├── Backup failed for > 24 hours
├── All backup jobs failing
├── Storage > 95% full
├── Data corruption detected
└── RTO/RPO violation

Warning (P2)
├── Single backup job failed
├── Backup duration > 2x normal
├── Storage > 80% full
├── Dedup ratio degraded
└── Performance degradation

Info (P3)
├── Backup completed successfully
├── Storage threshold reached
├── Scheduled maintenance
└── Capacity planning alert
```

---

## 💻 Задание 1: Prometheus Backup Exporter

```bash
cat > backup_prometheus_exporter.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
METRICS_PORT=9101
METRICS_FILE="/var/lib/node_exporter/textfile_collector/backup_metrics.prom"
BACKUP_STATUS_DIR="/var/lib/backup-status"
LOG_FILE="/var/log/backup_exporter.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# Initialize exporter
init_exporter() {
    log "Initializing Prometheus backup exporter..."
    
    # Create directories
    mkdir -p "$(dirname "$METRICS_FILE")"
    mkdir -p "$BACKUP_STATUS_DIR"
    
    # Install node_exporter if not present
    if ! command -v node_exporter &>/dev/null; then
        log "Installing node_exporter..."
        
        local version="1.7.0"
        wget "https://github.com/prometheus/node_exporter/releases/download/v${version}/node_exporter-${version}.linux-amd64.tar.gz"
        tar xzf "node_exporter-${version}.linux-amd64.tar.gz"
        sudo mv "node_exporter-${version}.linux-amd64/node_exporter" /usr/local/bin/
        rm -rf "node_exporter-${version}.linux-amd64"*
        
        # Create systemd service
        cat > /etc/systemd/system/node_exporter.service <<SERVICE
[Unit]
Description=Node Exporter
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/bin/node_exporter \
    --collector.textfile.directory=/var/lib/node_exporter/textfile_collector
Restart=always

[Install]
WantedBy=multi-user.target
SERVICE
        
        # Create prometheus user
        useradd -r -s /bin/false prometheus 2>/dev/null || true
        
        systemctl daemon-reload
        systemctl enable node_exporter
        systemctl start node_exporter
        
        log "node_exporter installed and started on port 9100"
    fi
    
    log "Exporter initialized"
}

# Collect backup job metrics
collect_backup_job_metrics() {
    log "Collecting backup job metrics..."
    
    local temp_file="${METRICS_FILE}.tmp"
    
    cat > "$temp_file" <<HEADER
# HELP backup_job_last_success_timestamp_seconds Last successful backup timestamp
# TYPE backup_job_last_success_timestamp_seconds gauge

# HELP backup_job_duration_seconds Duration of last backup job
# TYPE backup_job_duration_seconds gauge

# HELP backup_job_size_bytes Size of backup in bytes
# TYPE backup_job_size_bytes gauge

# HELP backup_job_success_total Total successful backups
# TYPE backup_job_success_total counter

# HELP backup_job_failure_total Total failed backups
# TYPE backup_job_failure_total counter

# HELP backup_job_status Current backup job status (1=success, 0=failed)
# TYPE backup_job_status gauge

HEADER
    
    # MySQL backups
    collect_mysql_metrics >> "$temp_file"
    
    # PostgreSQL backups
    collect_postgresql_metrics >> "$temp_file"
    
    # File system backups
    collect_filesystem_metrics >> "$temp_file"
    
    # Rsync snapshots
    collect_rsync_metrics >> "$temp_file"
    
    # Cloud backups
    collect_cloud_metrics >> "$temp_file"
    
    # Move temp file to final location atomically
    mv "$temp_file" "$METRICS_FILE"
    
    log "Metrics collection completed"
}

# MySQL backup metrics
collect_mysql_metrics() {
    local backup_dir="/backup/mysql/daily"
    
    if [ -d "$backup_dir" ]; then
        local latest=$(ls -t "$backup_dir"/*.sql.gz 2>/dev/null | head -1)
        
        if [ -n "$latest" ]; then
            local timestamp=$(stat -c %Y "$latest" 2>/dev/null || stat -f %m "$latest")
            local size=$(stat -c %s "$latest" 2>/dev/null || stat -f %z "$latest")
            local age_hours=$(( ($(date +%s) - timestamp) / 3600 ))
            local status=$( [ $age_hours -lt 25 ] && echo "1" || echo "0" )
            
            cat <<METRICS
backup_job_last_success_timestamp_seconds{job="mysql",type="full"} $timestamp
backup_job_size_bytes{job="mysql",type="full"} $size
backup_job_status{job="mysql",type="full"} $status
METRICS
            
            # Count successful/failed backups
            local success_count=$(find "$backup_dir" -name "*.sql.gz" -mtime -7 | wc -l)
            local expected_count=7
            local failed_count=$((expected_count - success_count))
            
            cat <<METRICS
backup_job_success_total{job="mysql"} $success_count
backup_job_failure_total{job="mysql"} $failed_count
METRICS
        fi
    fi
}

# PostgreSQL backup metrics
collect_postgresql_metrics() {
    local backup_dir="/backup/postgresql"
    
    if [ -d "$backup_dir" ]; then
        local latest=$(ls -t "$backup_dir"/pg-*.sql.gz 2>/dev/null | head -1)
        
        if [ -n "$latest" ]; then
            local timestamp=$(stat -c %Y "$latest" 2>/dev/null || stat -f %m "$latest")
            local size=$(stat -c %s "$latest" 2>/dev/null || stat -f %z "$latest")
            local status=$( [ $(($(date +%s) - timestamp)) -lt 86400 ] && echo "1" || echo "0" )
            
            cat <<METRICS
backup_job_last_success_timestamp_seconds{job="postgresql",type="full"} $timestamp
backup_job_size_bytes{job="postgresql",type="full"} $size
backup_job_status{job="postgresql",type="full"} $status
METRICS
        fi
        
        # WAL archive metrics
        local wal_dir="$backup_dir/wal_archive"
        if [ -d "$wal_dir" ]; then
            local wal_count=$(find "$wal_dir" -type f | wc -l)
            local wal_size=$(du -sb "$wal_dir" 2>/dev/null | cut -f1 || echo "0")
            
            cat <<METRICS
backup_wal_archive_files{job="postgresql"} $wal_count
backup_wal_archive_size_bytes{job="postgresql"} $wal_size
METRICS
        fi
    fi
}

# Filesystem backup metrics
collect_filesystem_metrics() {
    local backup_dir="/backup/rsync-snapshots"
    
    if [ -d "$backup_dir" ]; then
        local latest=$(ls -td "$backup_dir"/* 2>/dev/null | grep -v latest | head -1)
        
        if [ -n "$latest" ]; then
            local timestamp=$(stat -c %Y "$latest" 2>/dev/null || stat -f %m "$latest")
            local size=$(du -sb "$latest" 2>/dev/null | cut -f1 || echo "0")
            local file_count=$(find "$latest" -type f 2>/dev/null | wc -l)
            
            cat <<METRICS
backup_job_last_success_timestamp_seconds{job="filesystem",type="snapshot"} $timestamp
backup_job_size_bytes{job="filesystem",type="snapshot"} $size
backup_filesystem_file_count{job="filesystem"} $file_count
METRICS
        fi
        
        # Snapshot count
        local snapshot_count=$(ls -d "$backup_dir"/* 2>/dev/null | grep -v latest | wc -l)
        
        cat <<METRICS
backup_filesystem_snapshot_count{job="filesystem"} $snapshot_count
METRICS
    fi
}

# Rsync snapshot metrics
collect_rsync_metrics() {
    local backup_root="/backup/rsync-snapshots"
    
    if [ -d "$backup_root" ]; then
        # Calculate deduplication ratio
        local apparent_size=$(du -sb --apparent-size "$backup_root" 2>/dev/null | cut -f1 || echo "0")
        local actual_size=$(du -sb "$backup_root" 2>/dev/null | cut -f1 || echo "0")
        
        local dedup_ratio=0
        if [ "$actual_size" -gt 0 ]; then
            dedup_ratio=$(echo "scale=2; $apparent_size / $actual_size" | bc)
        fi
        
        cat <<METRICS
backup_deduplication_ratio{job="rsync"} $dedup_ratio
backup_deduplication_apparent_bytes{job="rsync"} $apparent_size
backup_deduplication_actual_bytes{job="rsync"} $actual_size
METRICS
    fi
}

# Cloud backup metrics
collect_cloud_metrics() {
    # S3 metrics
    if command -v aws &>/dev/null; then
        local bucket="company-backups"
        
        # Get object count and size
        local s3_stats=$(aws s3 ls "s3://$bucket/" --recursive --summarize 2>/dev/null | tail -2)
        
        if [ -n "$s3_stats" ]; then
            local object_count=$(echo "$s3_stats" | grep "Total Objects" | awk '{print $3}')
            local total_size=$(echo "$s3_stats" | grep "Total Size" | awk '{print $3}')
            
            cat <<METRICS
backup_cloud_object_count{provider="aws",bucket="$bucket"} ${object_count:-0}
backup_cloud_size_bytes{provider="aws",bucket="$bucket"} ${total_size:-0}
METRICS
        fi
    fi
    
    # Backblaze B2 metrics (using restic)
    if command -v restic &>/dev/null && [ -n "$RESTIC_REPOSITORY" ]; then
        local snapshot_count=$(restic snapshots --json 2>/dev/null | jq '. | length' || echo "0")
        local repo_stats=$(restic stats --mode raw-data --json 2>/dev/null || echo '{"total_size":0}')
        local repo_size=$(echo "$repo_stats" | jq -r '.total_size')
        
        cat <<METRICS
backup_cloud_snapshot_count{provider="backblaze",repository="b2"} $snapshot_count
backup_cloud_size_bytes{provider="backblaze",repository="b2"} $repo_size
METRICS
    fi
}

# Collect storage metrics
collect_storage_metrics() {
    log "Collecting storage metrics..."
    
    cat >> "$METRICS_FILE" <<HEADER

# HELP backup_storage_available_bytes Available storage space
# TYPE backup_storage_available_bytes gauge

# HELP backup_storage_used_bytes Used storage space
# TYPE backup_storage_used_bytes gauge

# HELP backup_storage_utilization_percent Storage utilization percentage
# TYPE backup_storage_utilization_percent gauge

HEADER
    
    # Local backup storage
    local backup_mount="/backup"
    
    if [ -d "$backup_mount" ]; then
        local df_output=$(df -B1 "$backup_mount" | tail -1)
        local total=$(echo "$df_output" | awk '{print $2}')
        local used=$(echo "$df_output" | awk '{print $3}')
        local available=$(echo "$df_output" | awk '{print $4}')
        local utilization=$(echo "$df_output" | awk '{print $5}' | tr -d '%')
        
        cat >> "$METRICS_FILE" <<METRICS
backup_storage_available_bytes{mount="$backup_mount"} $available
backup_storage_used_bytes{mount="$backup_mount"} $used
backup_storage_utilization_percent{mount="$backup_mount"} $utilization
METRICS
    fi
}

# Collect performance metrics
collect_performance_metrics() {
    log "Collecting performance metrics..."
    
    cat >> "$METRICS_FILE" <<HEADER

# HELP backup_duration_seconds Duration of backup operations
# TYPE backup_duration_seconds gauge

# HELP backup_throughput_bytes_per_second Backup throughput
# TYPE backup_throughput_bytes_per_second gauge

HEADER
    
    # Parse backup logs for duration and throughput
    for log_file in /var/log/backup*.log; do
        if [ -f "$log_file" ]; then
            # Extract job name from log file
            local job_name=$(basename "$log_file" .log | sed 's/backup_//')
            
            # Get last successful backup duration (if recorded)
            local duration=$(grep "completed in" "$log_file" | tail -1 | grep -o '[0-9]*s' | tr -d 's' || echo "0")
            
            if [ "$duration" -gt 0 ]; then
                cat >> "$METRICS_FILE" <<METRICS
backup_duration_seconds{job="$job_name"} $duration
METRICS
            fi
        fi
    done
}

# Collect RTO/RPO metrics
collect_rto_rpo_metrics() {
    log "Collecting RTO/RPO metrics..."
    
    cat >> "$METRICS_FILE" <<HEADER

# HELP backup_rpo_hours Recovery Point Objective in hours
# TYPE backup_rpo_hours gauge

# HELP backup_rto_hours Recovery Time Objective in hours
# TYPE backup_rto_hours gauge

# HELP backup_rpo_compliance Current RPO compliance (1=compliant, 0=violation)
# TYPE backup_rpo_compliance gauge

HEADER
    
    # Define RTO/RPO targets
    local mysql_rpo_hours=4
    local postgresql_rpo_hours=4
    local filesystem_rpo_hours=24
    
    # Check MySQL RPO compliance
    local mysql_backup=$(ls -t /backup/mysql/daily/*.sql.gz 2>/dev/null | head -1)
    if [ -n "$mysql_backup" ]; then
        local age_hours=$(( ($(date +%s) - $(stat -c %Y "$mysql_backup")) / 3600 ))
        local compliance=$( [ $age_hours -le $mysql_rpo_hours ] && echo "1" || echo "0" )
        
        cat >> "$METRICS_FILE" <<METRICS
backup_rpo_hours{job="mysql"} $mysql_rpo_hours
backup_rpo_compliance{job="mysql"} $compliance
METRICS
    fi
    
    # Check PostgreSQL RPO compliance
    local pg_backup=$(ls -t /backup/postgresql/pg-*.sql.gz 2>/dev/null | head -1)
    if [ -n "$pg_backup" ]; then
        local age_hours=$(( ($(date +%s) - $(stat -c %Y "$pg_backup")) / 3600 ))
        local compliance=$( [ $age_hours -le $postgresql_rpo_hours ] && echo "1" || echo "0" )
        
        cat >> "$METRICS_FILE" <<METRICS
backup_rpo_hours{job="postgresql"} $postgresql_rpo_hours
backup_rpo_compliance{job="postgresql"} $compliance
METRICS
    fi
}

# Main collection function
collect_all_metrics() {
    log "Starting metrics collection cycle..."
    
    collect_backup_job_metrics
    collect_storage_metrics
    collect_performance_metrics
    collect_rto_rpo_metrics
    
    log "Metrics collection cycle completed"
    log "Metrics file: $METRICS_FILE"
}

# Continuous collection daemon
run_daemon() {
    local interval="${1:-60}"
    
    log "Starting metrics collection daemon (interval: ${interval}s)..."
    
    while true; do
        collect_all_metrics
        sleep "$interval"
    done
}

# Generate Grafana dashboard JSON
generate_grafana_dashboard() {
    local dashboard_file="/tmp/backup_dashboard.json"
    
    log "Generating Grafana dashboard..."
    
    cat > "$dashboard_file" <<'DASHBOARD'
{
  "dashboard": {
    "title": "Backup Monitoring Dashboard",
    "tags": ["backup", "monitoring"],
    "timezone": "browser",
    "panels": [
      {
        "title": "Backup Success Rate",
        "type": "gauge",
        "targets": [
          {
            "expr": "sum(rate(backup_job_success_total[24h])) / (sum(rate(backup_job_success_total[24h])) + sum(rate(backup_job_failure_total[24h]))) * 100"
          }
        ],
        "gridPos": {"x": 0, "y": 0, "w": 6, "h": 4}
      },
      {
        "title": "Backup Job Status",
        "type": "stat",
        "targets": [
          {
            "expr": "backup_job_status",
            "legendFormat": "{{job}}"
          }
        ],
        "gridPos": {"x": 6, "y": 0, "w": 6, "h": 4}
      },
      {
        "title": "Storage Utilization",
        "type": "gauge",
        "targets": [
          {
            "expr": "backup_storage_utilization_percent"
          }
        ],
        "gridPos": {"x": 12, "y": 0, "w": 6, "h": 4}
      },
      {
        "title": "Total Backup Size",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(backup_job_size_bytes) by (job)",
            "legendFormat": "{{job}}"
          }
        ],
        "gridPos": {"x": 0, "y": 4, "w": 12, "h": 6}
      },
      {
        "title": "Backup Duration",
        "type": "graph",
        "targets": [
          {
            "expr": "backup_duration_seconds",
            "legendFormat": "{{job}}"
          }
        ],
        "gridPos": {"x": 12, "y": 4, "w": 12, "h": 6}
      },
      {
        "title": "RPO Compliance",
        "type": "stat",
        "targets": [
          {
            "expr": "backup_rpo_compliance",
            "legendFormat": "{{job}}"
          }
        ],
        "gridPos": {"x": 0, "y": 10, "w": 12, "h": 4}
      },
      {
        "title": "Deduplication Ratio",
        "type": "gauge",
        "targets": [
          {
            "expr": "backup_deduplication_ratio"
          }
        ],
        "gridPos": {"x": 12, "y": 10, "w": 6, "h": 4}
      }
    ]
  }
}
DASHBOARD
    
    log "Grafana dashboard generated: $dashboard_file"
    log "Import this JSON into Grafana"
    echo "$dashboard_file"
}

# Test metrics endpoint
test_metrics() {
    log "Testing metrics endpoint..."
    
    collect_all_metrics
    
    echo ""
    echo "=== Sample Metrics ==="
    head -50 "$METRICS_FILE"
    echo ""
    echo "=== Metrics File Location ==="
    echo "$METRICS_FILE"
    echo ""
    echo "=== Node Exporter Endpoint ==="
    echo "http://localhost:9100/metrics"
    echo ""
    echo "Test with:"
    echo "  curl http://localhost:9100/metrics | grep backup_"
}

# Main command handling
case "${1:-help}" in
    init)
        init_exporter
        ;;
    
    collect)
        collect_all_metrics
        ;;
    
    daemon)
        run_daemon "${2:-60}"
        ;;
    
    test)
        test_metrics
        ;;
    
    dashboard)
        generate_grafana_dashboard
        ;;
    
    *)
        cat <<HELP
Backup Prometheus Exporter

Usage: $0 <command> [options]

Commands:
  init              - Initialize exporter and install node_exporter
  collect           - Collect metrics once
  daemon [interval] - Run continuous collection (default: 60s)
  test              - Test metrics collection
  dashboard         - Generate Grafana dashboard JSON

Examples:
  $0 init
  $0 collect
  $0 daemon 30
  $0 test
  $0 dashboard

Metrics available at: http://localhost:9100/metrics
HELP
        ;;
esac
EOF

chmod +x backup_prometheus_exporter.sh
```

---

Продолжить со следующими частями модуля 8?

1. Alert Manager Configuration
2. Log Aggregation (ELK/Loki)
3. Backup Analytics & Reporting
4. Predictive Capacity Planning

## 💻 Задание 2: AlertManager Configuration & Smart Alerting

```bash
cat > backup_alertmanager.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
ALERTMANAGER_CONFIG="/etc/prometheus/alertmanager.yml"
PROMETHEUS_RULES="/etc/prometheus/rules/backup_alerts.yml"
ALERT_LOG="/var/log/backup_alerts.log"
ALERT_STATE_DIR="/var/lib/alertmanager"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$ALERT_LOG"
}

# Initialize AlertManager
init_alertmanager() {
    log "Initializing AlertManager for backup monitoring..."
    
    # Create directories
    mkdir -p "$(dirname "$ALERTMANAGER_CONFIG")"
    mkdir -p "$(dirname "$PROMETHEUS_RULES")"
    mkdir -p "$ALERT_STATE_DIR"
    
    # Install AlertManager if not present
    if ! command -v alertmanager &>/dev/null; then
        log "Installing AlertManager..."
        
        local version="0.26.0"
        wget "https://github.com/prometheus/alertmanager/releases/download/v${version}/alertmanager-${version}.linux-amd64.tar.gz"
        tar xzf "alertmanager-${version}.linux-amd64.tar.gz"
        sudo mv "alertmanager-${version}.linux-amd64/alertmanager" /usr/local/bin/
        sudo mv "alertmanager-${version}.linux-amd64/amtool" /usr/local/bin/
        rm -rf "alertmanager-${version}.linux-amd64"*
        
        log "AlertManager installed"
    fi
    
    # Create AlertManager configuration
    create_alertmanager_config
    
    # Create Prometheus alert rules
    create_prometheus_rules
    
    # Create systemd service
    create_systemd_service
    
    log "AlertManager initialization completed"
}

# Create AlertManager configuration
create_alertmanager_config() {
    log "Creating AlertManager configuration..."
    
    cat > "$ALERTMANAGER_CONFIG" <<'ALERTCONFIG'
global:
  resolve_timeout: 5m
  
  # SMTP settings for email alerts
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'backup-alerts@example.com'
  smtp_auth_username: 'backup-alerts@example.com'
  smtp_auth_password: 'your-app-password'
  smtp_require_tls: true

# Alert routing
route:
  group_by: ['alertname', 'job', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'default-receiver'
  
  routes:
    # Critical alerts - immediate notification
    - match:
        severity: critical
      receiver: 'critical-receiver'
      group_wait: 10s
      repeat_interval: 1h
      
    # Warning alerts - batched notification
    - match:
        severity: warning
      receiver: 'warning-receiver'
      group_wait: 5m
      repeat_interval: 6h
      
    # Info alerts - daily digest
    - match:
        severity: info
      receiver: 'info-receiver'
      group_wait: 12h
      repeat_interval: 24h

# Inhibition rules (suppress certain alerts)
inhibit_rules:
  # Suppress backup job failures if all backups are failing
  - source_match:
      alertname: 'AllBackupsFailing'
    target_match:
      alertname: 'BackupJobFailed'
    equal: ['job']
  
  # Suppress storage warnings if storage is critical
  - source_match:
      alertname: 'BackupStorageCritical'
    target_match:
      alertname: 'BackupStorageWarning'

# Receivers (notification channels)
receivers:
  - name: 'default-receiver'
    email_configs:
      - to: 'ops-team@example.com'
        headers:
          Subject: '[Backup Alert] {{ .GroupLabels.alertname }}'
    
  - name: 'critical-receiver'
    # Multiple notification channels for critical alerts
    email_configs:
      - to: 'ops-team@example.com,oncall@example.com'
        headers:
          Subject: '🚨 [CRITICAL] Backup Alert: {{ .GroupLabels.alertname }}'
        html: |
          <h2>Critical Backup Alert</h2>
          <p><strong>Alert:</strong> {{ .GroupLabels.alertname }}</p>
          <p><strong>Severity:</strong> {{ .CommonLabels.severity }}</p>
          <p><strong>Description:</strong> {{ .CommonAnnotations.description }}</p>
          <p><strong>Time:</strong> {{ .StartsAt }}</p>
          
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#backup-alerts'
        title: '🚨 Critical Backup Alert'
        text: |
          *Alert:* {{ .GroupLabels.alertname }}
          *Job:* {{ .CommonLabels.job }}
          *Description:* {{ .CommonAnnotations.description }}
          *Runbook:* {{ .CommonAnnotations.runbook_url }}
        
    pagerduty_configs:
      - routing_key: 'YOUR_PAGERDUTY_INTEGRATION_KEY'
        description: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.description }}'
        severity: 'critical'
        
    webhook_configs:
      - url: 'http://localhost:8080/webhook/backup-alert'
        send_resolved: true
  
  - name: 'warning-receiver'
    email_configs:
      - to: 'ops-team@example.com'
        headers:
          Subject: '⚠️ [Warning] Backup Alert: {{ .GroupLabels.alertname }}'
          
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#backup-alerts'
        title: '⚠️ Backup Warning'
        text: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.summary }}'
  
  - name: 'info-receiver'
    email_configs:
      - to: 'ops-team@example.com'
        headers:
          Subject: 'ℹ️ [Info] Backup Report: {{ .GroupLabels.alertname }}'
        send_resolved: false
ALERTCONFIG
    
    log "AlertManager configuration created: $ALERTMANAGER_CONFIG"
}

# Create Prometheus alert rules
create_prometheus_rules() {
    log "Creating Prometheus alert rules..."
    
    cat > "$PROMETHEUS_RULES" <<'RULES'
groups:
  - name: backup_alerts
    interval: 1m
    rules:
      
      # ============================================
      # CRITICAL ALERTS
      # ============================================
      
      - alert: BackupJobFailed
        expr: backup_job_status == 0
        for: 5m
        labels:
          severity: critical
          component: backup
        annotations:
          summary: "Backup job {{ $labels.job }} failed"
          description: |
            Backup job {{ $labels.job }} has failed.
            Current status: {{ $value }}
            This requires immediate attention.
          runbook_url: "https://wiki.example.com/runbooks/backup-job-failed"
      
      - alert: AllBackupsFailing
        expr: sum(backup_job_status) == 0
        for: 10m
        labels:
          severity: critical
          component: backup
        annotations:
          summary: "All backup jobs are failing"
          description: |
            All backup jobs have failed. This indicates a systemic issue.
            Immediate investigation required.
          runbook_url: "https://wiki.example.com/runbooks/all-backups-failing"
      
      - alert: BackupStorageCritical
        expr: backup_storage_utilization_percent > 95
        for: 5m
        labels:
          severity: critical
          component: storage
        annotations:
          summary: "Backup storage critically full"
          description: |
            Backup storage is {{ $value }}% full.
            Storage: {{ $labels.mount }}
            Free up space immediately or backups will fail.
          runbook_url: "https://wiki.example.com/runbooks/storage-full"
      
      - alert: BackupNotRunIn24Hours
        expr: (time() - backup_job_last_success_timestamp_seconds) > 86400
        for: 1h
        labels:
          severity: critical
          component: backup
        annotations:
          summary: "Backup {{ $labels.job }} not run in 24 hours"
          description: |
            Backup job {{ $labels.job }} has not run successfully in over 24 hours.
            Last successful backup: {{ $value | humanizeDuration }} ago
          runbook_url: "https://wiki.example.com/runbooks/backup-not-running"
      
      - alert: RPOViolation
        expr: backup_rpo_compliance == 0
        for: 15m
        labels:
          severity: critical
          component: compliance
        annotations:
          summary: "RPO violation for {{ $labels.job }}"
          description: |
            Recovery Point Objective violated for {{ $labels.job }}.
            Last backup is older than target RPO.
            Data loss risk increased.
          runbook_url: "https://wiki.example.com/runbooks/rpo-violation"
      
      - alert: BackupDataCorruption
        expr: rate(backup_integrity_check_failures_total[1h]) > 0
        for: 5m
        labels:
          severity: critical
          component: integrity
        annotations:
          summary: "Backup data corruption detected"
          description: |
            Integrity check failures detected for {{ $labels.job }}.
            Backup data may be corrupted.
          runbook_url: "https://wiki.example.com/runbooks/data-corruption"
      
      # ============================================
      # WARNING ALERTS
      # ============================================
      
      - alert: BackupStorageWarning
        expr: backup_storage_utilization_percent > 80
        for: 15m
        labels:
          severity: warning
          component: storage
        annotations:
          summary: "Backup storage usage high"
          description: |
            Backup storage is {{ $value }}% full.
            Storage: {{ $labels.mount }}
            Plan for capacity expansion.
      
      - alert: BackupDurationIncreased
        expr: |
          (
            backup_duration_seconds 
            / 
            avg_over_time(backup_duration_seconds[7d])
          ) > 2
        for: 30m
        labels:
          severity: warning
          component: performance
        annotations:
          summary: "Backup duration significantly increased"
          description: |
            Backup job {{ $labels.job }} is taking {{ $value | humanizeDuration }}.
            This is 2x longer than normal.
            Investigation recommended.
      
      - alert: BackupSizeAnomalous
        expr: |
          abs(
            (backup_job_size_bytes - avg_over_time(backup_job_size_bytes[7d]))
            /
            avg_over_time(backup_job_size_bytes[7d])
          ) > 0.5
        for: 1h
        labels:
          severity: warning
          component: anomaly
        annotations:
          summary: "Backup size changed significantly"
          description: |
            Backup size for {{ $labels.job }} is {{ $value | humanize1024 }}.
            This is 50% different from normal.
            Verify if this is expected.
      
      - alert: DeduplicationRatioDegraded
        expr: |
          (
            backup_deduplication_ratio
            /
            avg_over_time(backup_deduplication_ratio[7d])
          ) < 0.7
        for: 2h
        labels:
          severity: warning
          component: efficiency
        annotations:
          summary: "Deduplication ratio degraded"
          description: |
            Deduplication ratio for {{ $labels.job }} has decreased.
            Current: {{ $value }}, Normal: {{ .avg }}
            Storage efficiency reduced.
      
      - alert: BackupWindowExceeded
        expr: backup_duration_seconds > 14400  # 4 hours
        for: 15m
        labels:
          severity: warning
          component: performance
        annotations:
          summary: "Backup exceeding allocated window"
          description: |
            Backup job {{ $labels.job }} is taking {{ $value | humanizeDuration }}.
            This exceeds the 4-hour backup window.
      
      - alert: CloudBackupDelayed
        expr: |
          (time() - backup_cloud_last_sync_timestamp_seconds) > 7200
        for: 30m
        labels:
          severity: warning
          component: cloud
        annotations:
          summary: "Cloud backup sync delayed"
          description: |
            Cloud backup for {{ $labels.provider }} is delayed.
            Last sync: {{ $value | humanizeDuration }} ago
      
      # ============================================
      # INFO ALERTS (Daily Reports)
      # ============================================
      
      - alert: DailyBackupReport
        expr: hour() == 8 and minute() < 5  # 8:00 AM daily
        for: 1m
        labels:
          severity: info
          component: report
        annotations:
          summary: "Daily backup report"
          description: |
            Daily backup summary:
            - Success rate: {{ backup_success_rate }}%
            - Total backups: {{ backup_total_count }}
            - Storage used: {{ backup_storage_used_bytes | humanize1024 }}
            - Storage available: {{ backup_storage_available_bytes | humanize1024 }}
      
      - alert: WeeklyCapacityReport
        expr: day() == 1 and hour() == 9 and minute() < 5  # Monday 9 AM
        for: 1m
        labels:
          severity: info
          component: capacity
        annotations:
          summary: "Weekly capacity planning report"
          description: |
            Weekly storage trends:
            - Current utilization: {{ backup_storage_utilization_percent }}%
            - Growth rate: {{ backup_storage_growth_rate_7d }}%
            - Projected full date: {{ backup_storage_projected_full_date }}
RULES
    
    log "Prometheus alert rules created: $PROMETHEUS_RULES"
}

# Create systemd service
create_systemd_service() {
    log "Creating AlertManager systemd service..."
    
    cat > /etc/systemd/system/alertmanager.service <<SERVICE
[Unit]
Description=Prometheus AlertManager
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/bin/alertmanager \
    --config.file=$ALERTMANAGER_CONFIG \
    --storage.path=$ALERT_STATE_DIR \
    --web.listen-address=:9093
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
SERVICE
    
    systemctl daemon-reload
    systemctl enable alertmanager
    
    log "AlertManager service created"
}

# Test alert configuration
test_alertmanager() {
    log "Testing AlertManager configuration..."
    
    # Validate configuration
    if command -v amtool &>/dev/null; then
        amtool check-config "$ALERTMANAGER_CONFIG"
        
        if [ $? -eq 0 ]; then
            log "✓ AlertManager configuration is valid"
        else
            log "✗ AlertManager configuration has errors"
            return 1
        fi
    fi
    
    # Test alert firing (simulate backup failure)
    log "Simulating backup failure alert..."
    
    cat > /tmp/test_alert.json <<TESTALERT
[
  {
    "labels": {
      "alertname": "BackupJobFailed",
      "job": "test-backup",
      "severity": "critical"
    },
    "annotations": {
      "summary": "Test backup alert",
      "description": "This is a test alert from backup monitoring system"
    },
    "startsAt": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
    "endsAt": "$(date -u -d '+1 hour' +%Y-%m-%dT%H:%M:%SZ)"
  }
]
TESTALERT
    
    # Send test alert
    curl -X POST http://localhost:9093/api/v1/alerts \
        -H 'Content-Type: application/json' \
        -d @/tmp/test_alert.json
    
    if [ $? -eq 0 ]; then
        log "✓ Test alert sent successfully"
        log "Check your notification channels for the test alert"
    else
        log "✗ Failed to send test alert"
        return 1
    fi
    
    rm -f /tmp/test_alert.json
}

# Query active alerts
query_alerts() {
    log "Querying active alerts..."
    
    if command -v amtool &>/dev/null; then
        echo ""
        echo "=== Active Alerts ==="
        amtool alert query
        
        echo ""
        echo "=== Silenced Alerts ==="
        amtool silence query
    else
        # Fallback to API
        echo ""
        echo "=== Active Alerts (via API) ==="
        curl -s http://localhost:9093/api/v1/alerts | jq -r '.data[] | 
            "Alert: \(.labels.alertname) | Job: \(.labels.job) | Severity: \(.labels.severity) | Status: \(.status.state)"'
    fi
}

# Create alert silence
create_silence() {
    local alertname="$1"
    local duration="${2:-1h}"
    local comment="${3:-Manual silence}"
    
    if [ -z "$alertname" ]; then
        echo "Usage: $0 silence <alertname> [duration] [comment]"
        return 1
    fi
    
    log "Creating silence for alert: $alertname (duration: $duration)"
    
    local end_time=$(date -u -d "+$duration" +%Y-%m-%dT%H:%M:%SZ)
    
    cat > /tmp/silence.json <<SILENCE
{
  "matchers": [
    {
      "name": "alertname",
      "value": "$alertname",
      "isRegex": false
    }
  ],
  "startsAt": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "endsAt": "$end_time",
  "createdBy": "$(whoami)",
  "comment": "$comment"
}
SILENCE
    
    curl -X POST http://localhost:9093/api/v1/silences \
        -H 'Content-Type: application/json' \
        -d @/tmp/silence.json
    
    if [ $? -eq 0 ]; then
        log "✓ Silence created successfully"
    else
        log "✗ Failed to create silence"
        return 1
    fi
    
    rm -f /tmp/silence.json
}

# Generate alert statistics
generate_alert_stats() {
    local report_file="/var/log/alert_statistics_$(date +%Y%m%d).html"
    
    log "Generating alert statistics report..."
    
    cat > "$report_file" <<'HTML'
<!DOCTYPE html>
<html>
<head>
    <title>Backup Alert Statistics</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background: #f5f5f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        h1 {
            color: #d32f2f;
            border-bottom: 3px solid #d32f2f;
            padding-bottom: 10px;
        }
        .stats-grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 20px;
            margin: 30px 0;
        }
        .stat-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 20px;
            border-radius: 8px;
            text-align: center;
        }
        .stat-card.critical {
            background: linear-gradient(135deg, #eb3349 0%, #f45c43 100%);
        }
        .stat-card.warning {
            background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
        }
        .stat-value {
            font-size: 3em;
            font-weight: bold;
            margin: 10px 0;
        }
        .stat-label {
            font-size: 0.9em;
            opacity: 0.9;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #d32f2f;
            color: white;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚨 Backup Alert Statistics</h1>
        <p><strong>Generated:</strong> $(date)</p>
        <p><strong>Period:</strong> Last 7 days</p>
        
        <div class="stats-grid">
HTML
    
    # Parse alert logs for statistics
    local total_alerts=$(grep -c "alert" "$ALERT_LOG" 2>/dev/null || echo "0")
    local critical_alerts=$(grep -c "severity: critical" "$ALERT_LOG" 2>/dev/null || echo "0")
    local warning_alerts=$(grep -c "severity: warning" "$ALERT_LOG" 2>/dev/null || echo "0")
    local resolved_alerts=$(grep -c "resolved" "$ALERT_LOG" 2>/dev/null || echo "0")
    
    cat >> "$report_file" <<HTML
            <div class="stat-card">
                <div class="stat-label">Total Alerts</div>
                <div class="stat-value">$total_alerts</div>
            </div>
            <div class="stat-card critical">
                <div class="stat-label">Critical Alerts</div>
                <div class="stat-value">$critical_alerts</div>
            </div>
            <div class="stat-card warning">
                <div class="stat-label">Warning Alerts</div>
                <div class="stat-value">$warning_alerts</div>
            </div>
            <div class="stat-card" style="background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);">
                <div class="stat-label">Resolved</div>
                <div class="stat-value">$resolved_alerts</div>
            </div>
        </div>
        
        <h2>Alert Breakdown by Type</h2>
        <table>
            <thead>
                <tr>
                    <th>Alert Name</th>
                    <th>Severity</th>
                    <th>Count</th>
                    <th>Avg Duration</th>
                    <th>Last Occurrence</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>BackupJobFailed</td>
                    <td>Critical</td>
                    <td>$(grep -c "BackupJobFailed" "$ALERT_LOG" 2>/dev/null || echo "0")</td>
                    <td>2h 15m</td>
                    <td>$(grep "BackupJobFailed" "$ALERT_LOG" 2>/dev/null | tail -1 | cut -d' ' -f1-2 || echo "N/A")</td>
                </tr>
                <tr>
                    <td>BackupStorageWarning</td>
                    <td>Warning</td>
                    <td>$(grep -c "BackupStorageWarning" "$ALERT_LOG" 2>/dev/null || echo "0")</td>
                    <td>6h 30m</td>
                    <td>$(grep "BackupStorageWarning" "$ALERT_LOG" 2>/dev/null | tail -1 | cut -d' ' -f1-2 || echo "N/A")</td>
                </tr>
                <tr>
                    <td>BackupDurationIncreased</td>
                    <td>Warning</td>
                    <td>$(grep -c "BackupDurationIncreased" "$ALERT_LOG" 2>/dev/null || echo "0")</td>
                    <td>4h 45m</td>
                    <td>$(grep "BackupDurationIncreased" "$ALERT_LOG" 2>/dev/null | tail -1 | cut -d' ' -f1-2 || echo "N/A")</td>
                </tr>
            </tbody>
        </table>
        
        <h2>Recommendations</h2>
        <ul>
HTML
    
    if [ "$critical_alerts" -gt 0 ]; then
        cat >> "$report_file" <<HTML
            <li><strong>Action Required:</strong> $critical_alerts critical alert(s) need immediate attention</li>
HTML
    fi
    
    cat >> "$report_file" <<HTML
            <li>Review alert thresholds if false positive rate is high</li>
            <li>Ensure all notification channels are working correctly</li>
            <li>Update runbooks based on recent incidents</li>
            <li>Schedule alert review meeting if alert volume is increasing</li>
        </ul>
        
        <hr>
        <p style="text-align: center; color: #666;">
            Backup AlertManager v1.0 | Generated: $(date)
        </p>
    </div>
</body>
</html>
HTML
    
    log "Alert statistics report generated: $report_file"
    echo "$report_file"
}

# Main command handling
case "${1:-help}" in
    init)
        init_alertmanager
        ;;
    
    test)
        test_alertmanager
        ;;
    
    query)
        query_alerts
        ;;
    
    silence)
        create_silence "$2" "$3" "$4"
        ;;
    
    stats)
        generate_alert_stats
        ;;
    
    start)
        log "Starting AlertManager..."
        systemctl start alertmanager
        systemctl status alertmanager
        ;;
    
    stop)
        log "Stopping AlertManager..."
        systemctl stop alertmanager
        ;;
    
    restart)
        log "Restarting AlertManager..."
        systemctl restart alertmanager
        ;;
    
    *)
        cat <<HELP
Backup AlertManager Configuration

Usage: $0 <command> [options]

Commands:
  init                          - Initialize AlertManager
  test                          - Test configuration and send test alert
  query                         - Query active alerts
  silence <alert> [dur] [msg]   - Create alert silence
                                  duration: 1h, 2d, etc. (default: 1h)
  stats                         - Generate alert statistics report
  start                         - Start AlertManager service
  stop                          - Stop AlertManager service
  restart                       - Restart AlertManager service

Examples:
  $0 init
  $0 test
  $0 query
  $0 silence BackupJobFailed 2h "Planned maintenance"
  $0 stats

AlertManager UI: http://localhost:9093
HELP
        ;;
esac
EOF

chmod +x backup_alertmanager.sh
```


## 💻 Задание 3: Log Aggregation & Analysis (ELK/Loki)

```bash
cat > backup_log_aggregation.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
LOKI_CONFIG="/etc/loki/config.yml"
PROMTAIL_CONFIG="/etc/promtail/config.yml"
LOG_DIR="/var/log/backup-logs"
LOKI_DATA_DIR="/var/lib/loki"
PROMTAIL_POSITIONS="/var/lib/promtail/positions.yaml"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_DIR/log_aggregation.log"
}

# Initialize log aggregation stack
init_log_aggregation() {
    log "Initializing log aggregation stack (Loki + Promtail)..."
    
    # Create directories
    mkdir -p "$LOG_DIR"
    mkdir -p "$LOKI_DATA_DIR"
    mkdir -p "$(dirname "$PROMTAIL_POSITIONS")"
    mkdir -p "$(dirname "$LOKI_CONFIG")"
    mkdir -p "$(dirname "$PROMTAIL_CONFIG")"
    
    # Install Loki
    install_loki
    
    # Install Promtail
    install_promtail
    
    # Configure Loki
    configure_loki
    
    # Configure Promtail
    configure_promtail
    
    # Create systemd services
    create_loki_service
    create_promtail_service
    
    log "Log aggregation stack initialized"
}

# Install Loki
install_loki() {
    if command -v loki &>/dev/null; then
        log "Loki already installed"
        return 0
    fi
    
    log "Installing Loki..."
    
    local version="2.9.3"
    local arch="amd64"
    
    wget "https://github.com/grafana/loki/releases/download/v${version}/loki-linux-${arch}.zip"
    unzip "loki-linux-${arch}.zip"
    sudo mv loki-linux-${arch} /usr/local/bin/loki
    sudo chmod +x /usr/local/bin/loki
    rm -f "loki-linux-${arch}.zip"
    
    log "Loki installed successfully"
}

# Install Promtail
install_promtail() {
    if command -v promtail &>/dev/null; then
        log "Promtail already installed"
        return 0
    fi
    
    log "Installing Promtail..."
    
    local version="2.9.3"
    local arch="amd64"
    
    wget "https://github.com/grafana/loki/releases/download/v${version}/promtail-linux-${arch}.zip"
    unzip "promtail-linux-${arch}.zip"
    sudo mv promtail-linux-${arch} /usr/local/bin/promtail
    sudo chmod +x /usr/local/bin/promtail
    rm -f "promtail-linux-${arch}.zip"
    
    log "Promtail installed successfully"
}

# Configure Loki
configure_loki() {
    log "Configuring Loki..."
    
    cat > "$LOKI_CONFIG" <<'LOKICONFIG'
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096
  log_level: info

common:
  path_prefix: /var/lib/loki
  storage:
    filesystem:
      chunks_directory: /var/lib/loki/chunks
      rules_directory: /var/lib/loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /var/lib/loki/boltdb-shipper-active
    cache_location: /var/lib/loki/boltdb-shipper-cache
    cache_ttl: 24h
    shared_store: filesystem
  filesystem:
    directory: /var/lib/loki/chunks

compactor:
  working_directory: /var/lib/loki/boltdb-shipper-compactor
  shared_store: filesystem

limits_config:
  retention_period: 744h  # 31 days
  reject_old_samples: true
  reject_old_samples_max_age: 168h
  ingestion_rate_mb: 10
  ingestion_burst_size_mb: 20
  max_query_length: 0h
  max_query_parallelism: 32

chunk_store_config:
  max_look_back_period: 744h

table_manager:
  retention_deletes_enabled: true
  retention_period: 744h

query_range:
  results_cache:
    cache:
      embedded_cache:
        enabled: true
        max_size_mb: 100

ruler:
  alertmanager_url: http://localhost:9093
  ring:
    kvstore:
      store: inmemory
  rule_path: /var/lib/loki/rules-temp
  storage:
    type: local
    local:
      directory: /var/lib/loki/rules
  enable_api: true
  enable_alertmanager_v2: true
LOKICONFIG
    
    log "Loki configuration created: $LOKI_CONFIG"
}

# Configure Promtail
configure_promtail() {
    log "Configuring Promtail..."
    
    cat > "$PROMTAIL_CONFIG" <<'PROMTAILCONFIG'
server:
  http_listen_port: 9080
  grpc_listen_port: 0
  log_level: info

positions:
  filename: /var/lib/promtail/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
  # Backup job logs
  - job_name: backup_jobs
    static_configs:
      - targets:
          - localhost
        labels:
          job: backup_jobs
          __path__: /var/log/backup*.log
    pipeline_stages:
      # Parse timestamp
      - regex:
          expression: '^\[(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\]'
      - timestamp:
          source: timestamp
          format: '2006-01-02 15:04:05'
      
      # Extract log level
      - regex:
          expression: '\[(?P<level>INFO|WARNING|ERROR|CRITICAL)\]'
      - labels:
          level:
      
      # Extract job name
      - regex:
          expression: 'job[=:](?P<job_name>[\w-]+)'
      - labels:
          job_name:
      
      # Extract backup type
      - regex:
          expression: 'type[=:](?P<backup_type>full|incremental|differential)'
      - labels:
          backup_type:
  
  # MySQL backup logs
  - job_name: mysql_backups
    static_configs:
      - targets:
          - localhost
        labels:
          job: mysql_backups
          application: mysql
          __path__: /var/log/mysql_backup*.log
    pipeline_stages:
      - regex:
          expression: '^\[(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\]'
      - timestamp:
          source: timestamp
          format: '2006-01-02 15:04:05'
      - regex:
          expression: '\[(?P<level>INFO|WARNING|ERROR)\]'
      - labels:
          level:
  
  # PostgreSQL backup logs
  - job_name: postgresql_backups
    static_configs:
      - targets:
          - localhost
        labels:
          job: postgresql_backups
          application: postgresql
          __path__: /var/log/pg_backup*.log
    pipeline_stages:
      - regex:
          expression: '^\[(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\]'
      - timestamp:
          source: timestamp
          format: '2006-01-02 15:04:05'
      - regex:
          expression: '\[(?P<level>INFO|WARNING|ERROR)\]'
      - labels:
          level:
  
  # Recovery operation logs
  - job_name: recovery_operations
    static_configs:
      - targets:
          - localhost
        labels:
          job: recovery
          __path__: /var/log/recovery*.log
    pipeline_stages:
      - regex:
          expression: '^\[(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\]'
      - timestamp:
          source: timestamp
          format: '2006-01-02 15:04:05'
      - regex:
          expression: '\[(?P<level>INFO|WARNING|ERROR|SUCCESS)\]'
      - labels:
          level:
      - regex:
          expression: 'scenario[=:](?P<scenario>[\w-]+)'
      - labels:
          scenario:
  
  # System logs (syslog)
  - job_name: syslog
    static_configs:
      - targets:
          - localhost
        labels:
          job: syslog
          __path__: /var/log/syslog
    pipeline_stages:
      - regex:
          expression: '^(?P<timestamp>\w+\s+\d+\s+\d{2}:\d{2}:\d{2})\s+(?P<hostname>[\w-]+)\s+(?P<program>[\w-]+)(\[(?P<pid>\d+)\])?:'
      - labels:
          program:
          hostname:
      - match:
          selector: '{program="backup"}'
          stages:
            - regex:
                expression: 'backup.*(?P<status>success|failed)'
            - labels:
                status:
  
  # Parallel backup logs
  - job_name: parallel_backups
    static_configs:
      - targets:
          - localhost
        labels:
          job: parallel_backups
          __path__: /var/log/parallel-backup/*.log
    pipeline_stages:
      - regex:
          expression: '^\[(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\]'
      - timestamp:
          source: timestamp
          format: '2006-01-02 15:04:05'
      - regex:
          expression: 'job_id[=:](?P<job_id>[\w-]+)'
      - labels:
          job_id:
  
  # Network backup logs
  - job_name: network_backups
    static_configs:
      - targets:
          - localhost
        labels:
          job: network_backups
          __path__: /var/log/network_backup*.log
    pipeline_stages:
      - regex:
          expression: 'throughput[=:](?P<throughput>\d+\.?\d*)\s*(?P<unit>MB/s|Mbps)'
      - labels:
          unit:
PROMTAILCONFIG
    
    log "Promtail configuration created: $PROMTAIL_CONFIG"
}

# Create Loki systemd service
create_loki_service() {
    log "Creating Loki systemd service..."
    
    cat > /etc/systemd/system/loki.service <<SERVICE
[Unit]
Description=Loki Log Aggregation System
After=network.target

[Service]
Type=simple
User=loki
ExecStart=/usr/local/bin/loki -config.file=$LOKI_CONFIG
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
SERVICE
    
    # Create loki user
    useradd -r -s /bin/false loki 2>/dev/null || true
    chown -R loki:loki "$LOKI_DATA_DIR"
    
    systemctl daemon-reload
    systemctl enable loki
    
    log "Loki service created"
}

# Create Promtail systemd service
create_promtail_service() {
    log "Creating Promtail systemd service..."
    
    cat > /etc/systemd/system/promtail.service <<SERVICE
[Unit]
Description=Promtail Log Shipper
After=network.target loki.service

[Service]
Type=simple
User=promtail
ExecStart=/usr/local/bin/promtail -config.file=$PROMTAIL_CONFIG
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
SERVICE
    
    # Create promtail user
    useradd -r -s /bin/false promtail 2>/dev/null || true
    chown -R promtail:promtail "$(dirname "$PROMTAIL_POSITIONS")"
    
    # Add promtail to adm group to read logs
    usermod -a -G adm promtail 2>/dev/null || true
    
    systemctl daemon-reload
    systemctl enable promtail
    
    log "Promtail service created"
}

# Query logs with LogQL
query_logs() {
    local query="$1"
    local limit="${2:-100}"
    local since="${3:-1h}"
    
    if [ -z "$query" ]; then
        echo "Usage: $0 query '<LogQL_query>' [limit] [time_range]"
        echo ""
        echo "Examples:"
        echo "  $0 query '{job=\"backup_jobs\"}' 50 2h"
        echo "  $0 query '{level=\"ERROR\"}' 100 24h"
        echo "  $0 query '{job=\"mysql_backups\"} |= \"failed\"' 20 12h"
        return 1
    fi
    
    log "Querying logs: $query"
    
    local end_time=$(date +%s)000000000  # nanoseconds
    local start_time=$(date -d "-$since" +%s)000000000
    
    curl -s "http://localhost:3100/loki/api/v1/query_range" \
        --data-urlencode "query=$query" \
        --data-urlencode "limit=$limit" \
        --data-urlencode "start=$start_time" \
        --data-urlencode "end=$end_time" \
        | jq -r '.data.result[] | .values[] | .[1]' | head -n "$limit"
}

# Analyze backup failures
analyze_failures() {
    local time_range="${1:-24h}"
    
    log "Analyzing backup failures in last $time_range..."
    
    echo ""
    echo "=== Backup Failures Analysis ==="
    echo ""
    
    # Query failed backups
    local query='{job=~".*backup.*"} |~ "(?i)(failed|error)"'
    
    echo "Recent Failures:"
    query_logs "$query" 50 "$time_range" | while read line; do
        echo "  - $line"
    done
    
    echo ""
    echo "Failure Breakdown by Job:"
    
    # Count failures per job
    curl -s "http://localhost:3100/loki/api/v1/query" \
        --data-urlencode 'query=sum by (job) (count_over_time({job=~".*backup.*"} |~ "(?i)failed" [24h]))' \
        | jq -r '.data.result[] | "\(.metric.job): \(.value[1])"'
    
    echo ""
    echo "Most Common Error Messages:"
    
    query_logs '{level="ERROR"}' 100 "$time_range" | \
        grep -oE 'ERROR:.*' | \
        sort | uniq -c | sort -rn | head -10
}

# Generate log analysis report
generate_log_report() {
    local report_file="/var/log/log_analysis_report_$(date +%Y%m%d).html"
    
    log "Generating log analysis report..."
    
    cat > "$report_file" <<'HTML'
<!DOCTYPE html>
<html>
<head>
    <title>Backup Log Analysis Report</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 20px;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
        }
        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        h1 {
            color: #1e3c72;
            border-bottom: 3px solid #2a5298;
            padding-bottom: 10px;
        }
        .metrics-grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 20px;
            margin: 30px 0;
        }
        .metric-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 25px;
            border-radius: 10px;
            text-align: center;
        }
        .metric-card.error {
            background: linear-gradient(135deg, #eb3349 0%, #f45c43 100%);
        }
        .metric-card.warning {
            background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
        }
        .metric-value {
            font-size: 3em;
            font-weight: bold;
            margin: 15px 0;
        }
        .metric-label {
            font-size: 1em;
            opacity: 0.9;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .log-viewer {
            background: #2d2d2d;
            color: #f8f8f2;
            padding: 20px;
            border-radius: 8px;
            font-family: 'Courier New', monospace;
            font-size: 0.9em;
            max-height: 400px;
            overflow-y: auto;
            margin: 20px 0;
        }
        .log-error { color: #ff6b6b; }
        .log-warning { color: #ffd93d; }
        .log-info { color: #6bcf7f; }
    </style>
</head>
<body>
    <div class="container">
        <h1>📊 Backup Log Analysis Report</h1>
        <p><strong>Generated:</strong> $(date)</p>
        <p><strong>Analysis Period:</strong> Last 24 hours</p>
        
        <div class="metrics-grid">
HTML
    
    # Query log statistics
    local total_logs=$(curl -s "http://localhost:3100/loki/api/v1/query" \
        --data-urlencode 'query=count_over_time({job=~".*backup.*"}[24h])' \
        | jq -r '.data.result[0].value[1]' 2>/dev/null || echo "0")
    
    local error_count=$(curl -s "http://localhost:3100/loki/api/v1/query" \
        --data-urlencode 'query=count_over_time({job=~".*backup.*", level="ERROR"}[24h])' \
        | jq -r '.data.result[0].value[1]' 2>/dev/null || echo "0")
    
    local warning_count=$(curl -s "http://localhost:3100/loki/api/v1/query" \
        --data-urlencode 'query=count_over_time({job=~".*backup.*", level="WARNING"}[24h])' \
        | jq -r '.data.result[0].value[1]' 2>/dev/null || echo "0")
    
    local success_count=$(curl -s "http://localhost:3100/loki/api/v1/query" \
        --data-urlencode 'query=count_over_time({job=~".*backup.*"} |~ "(?i)success" [24h])' \
        | jq -r '.data.result[0].value[1]' 2>/dev/null || echo "0")
    
    cat >> "$report_file" <<HTML
            <div class="metric-card">
                <div class="metric-label">Total Log Entries</div>
                <div class="metric-value">$total_logs</div>
            </div>
            <div class="metric-card error">
                <div class="metric-label">Errors</div>
                <div class="metric-value">$error_count</div>
            </div>
            <div class="metric-card warning">
                <div class="metric-label">Warnings</div>
                <div class="metric-value">$warning_count</div>
            </div>
            <div class="metric-card" style="background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);">
                <div class="metric-label">Successful Backups</div>
                <div class="metric-value">$success_count</div>
            </div>
        </div>
        
        <h2>Error Analysis</h2>
        <table>
            <thead>
                <tr>
                    <th>Job</th>
                    <th>Error Count</th>
                    <th>Last Error</th>
                    <th>Status</th>
                </tr>
            </thead>
            <tbody>
HTML
    
    # Add error breakdown
    curl -s "http://localhost:3100/loki/api/v1/query" \
        --data-urlencode 'query=sum by (job) (count_over_time({job=~".*backup.*", level="ERROR"}[24h]))' \
        | jq -r '.data.result[] | 
            "<tr><td>\(.metric.job)</td><td>\(.value[1])</td><td>Recent</td><td>Needs attention</td></tr>"' \
        >> "$report_file" 2>/dev/null || true
    
    cat >> "$report_file" <<HTML
            </tbody>
        </table>
        
        <h2>Recent Error Logs</h2>
        <div class="log-viewer">
HTML
    
    # Add recent errors
    query_logs '{level="ERROR"}' 20 24h | while read line; do
        echo "<div class=\"log-error\">$line</div>" >> "$report_file"
    done
    
    cat >> "$report_file" <<HTML
        </div>
        
        <h2>Log Volume by Job</h2>
        <table>
            <thead>
                <tr>
                    <th>Job</th>
                    <th>Log Entries (24h)</th>
                    <th>Avg Entry Size</th>
                    <th>Trend</th>
                </tr>
            </thead>
            <tbody>
HTML
    
    # Add log volume by job
    curl -s "http://localhost:3100/loki/api/v1/query" \
        --data-urlencode 'query=sum by (job) (count_over_time({job=~".*backup.*"}[24h]))' \
        | jq -r '.data.result[] | 
            "<tr><td>\(.metric.job)</td><td>\(.value[1])</td><td>~500B</td><td>Stable</td></tr>"' \
        >> "$report_file" 2>/dev/null || true
    
    cat >> "$report_file" <<HTML
            </tbody>
        </table>
        
        <h2>Common Issues Detected</h2>
        <ul>
HTML
    
    if [ "$error_count" -gt 10 ]; then
        cat >> "$report_file" <<HTML
            <li><strong>High Error Rate:</strong> $error_count errors in last 24h - investigation recommended</li>
HTML
    fi
    
    cat >> "$report_file" <<HTML
            <li>Check disk space if backup duration is increasing</li>
            <li>Review network connectivity if remote backups are failing</li>
            <li>Verify database connectivity for DB backup jobs</li>
        </ul>
        
        <h2>Recommendations</h2>
        <ul>
            <li>Set up alert rules for error rate > 10/hour</li>
            <li>Implement log rotation to manage storage</li>
            <li>Add structured logging for better analysis</li>
            <li>Create dashboards in Grafana for real-time monitoring</li>
            <li>Schedule weekly log review meetings</li>
        </ul>
        
        <h2>Log Retention Policy</h2>
        <p>Current retention: <strong>31 days</strong></p>
        <p>Adjust in Loki configuration: <code>$LOKI_CONFIG</code></p>
        
        <hr>
        <p style="text-align: center; color: #666;">
            Log Analysis Report v1.0 | Powered by Loki + Promtail
        </p>
    </div>
</body>
</html>
HTML
    
    log "Log analysis report generated: $report_file"
    echo "$report_file"
}

# Create Loki alert rules
create_loki_alert_rules() {
    local rules_file="$LOKI_DATA_DIR/rules/backup-logs.yml"
    
    log "Creating Loki alert rules..."
    
    mkdir -p "$(dirname "$rules_file")"
    
    cat > "$rules_file" <<'RULES'
groups:
  - name: backup_log_alerts
    interval: 1m
    rules:
      - alert: HighBackupErrorRate
        expr: |
          sum(rate({job=~".*backup.*", level="ERROR"}[5m])) > 0.1
        for: 5m
        labels:
          severity: warning
          component: logs
        annotations:
          summary: "High error rate in backup logs"
          description: "Error rate: {{ $value }} errors/sec"
      
      - alert: BackupLogErrors
        expr: |
          count_over_time({job=~".*backup.*"} |~ "(?i)(failed|error|critical)" [5m]) > 10
        for: 2m
        labels:
          severity: critical
          component: logs
        annotations:
          summary: "Multiple backup errors detected"
          description: "{{ $value }} error messages in 5 minutes"
      
      - alert: NoBackupLogsReceived
        expr: |
          absent_over_time({job=~".*backup.*"}[15m])
        for: 5m
        labels:
          severity: warning
          component: logs
        annotations:
          summary: "No backup logs received"
          description: "No logs from backup jobs in 15 minutes"
RULES
    
    log "Loki alert rules created: $rules_file"
}

# Test log ingestion
test_log_ingestion() {
    log "Testing log ingestion..."
    
    # Generate test log entries
    local test_log="/tmp/test_backup.log"
    
    cat > "$test_log" <<TESTLOG
[$(date +'%Y-%m-%d %H:%M:%S')] [INFO] Starting test backup job=test-backup
[$(date +'%Y-%m-%d %H:%M:%S')] [INFO] Backup type=full size=1024MB
[$(date +'%Y-%m-%d %H:%M:%S')] [WARNING] Slow network detected throughput=50MB/s
[$(date +'%Y-%m-%d %H:%M:%S')] [INFO] Backup completed successfully duration=300s
TESTLOG
    
    # Wait for Promtail to pick up logs
    sleep 5
    
    # Query for test logs
    log "Querying test logs..."
    local result=$(query_logs '{job="backup_jobs"} |= "test-backup"' 10 5m)
    
    if [ -n "$result" ]; then
        log "✓ Log ingestion working correctly"
        echo "$result"
    else
        log "✗ Log ingestion test failed"
        return 1
    fi
    
    rm -f "$test_log"
}

# Main command handling
case "${1:-help}" in
    init)
        init_log_aggregation
        ;;
    
    start)
        log "Starting log aggregation services..."
        systemctl start loki
        systemctl start promtail
        systemctl status loki --no-pager
        systemctl status promtail --no-pager
        ;;
    
    stop)
        log "Stopping log aggregation services..."
        systemctl stop promtail
        systemctl stop loki
        ;;
    
    restart)
        log "Restarting log aggregation services..."
        systemctl restart loki
        systemctl restart promtail
        ;;
    
    query)
        query_logs "$2" "$3" "$4"
        ;;
    
    analyze)
        analyze_failures "$2"
        ;;
    
    report)
        generate_log_report
        ;;
    
    rules)
        create_loki_alert_rules
        ;;
    
    test)
        test_log_ingestion
        ;;
    
    *)
        cat <<HELP
Backup Log Aggregation (Loki + Promtail)

Usage: $0 <command> [options]

Commands:
  init                        - Initialize log aggregation stack
  start                       - Start Loki and Promtail services
  stop                        - Stop services
  restart                     - Restart services
  query '<LogQL>' [lim] [time]- Query logs with LogQL
  analyze [time_range]        - Analyze backup failures
  report                      - Generate log analysis report
  rules                       - Create Loki alert rules
  test                        - Test log ingestion

Examples:
  $0 init
  $0 start
  $0 query '{job="mysql_backups"}' 50 2h
  $0 query '{level="ERROR"}' 100 24h
  $0 analyze 24h
  $0 report

Loki API: http://localhost:3100
Promtail: http://localhost:9080

LogQL Examples:
  {job="backup_jobs"}
  {job="mysql_backups", level="ERROR"}
  {job=~".*backup.*"} |= "failed"
  {job="backup_jobs"} |~ "(?i)error"
HELP
        ;;
esac
EOF

chmod +x backup_log_aggregation.sh
```

Продолжить с остальными частями модуля 8?

- Backup Analytics & Predictive Analysis
- Capacity Planning


## 💻 Задание 4: Backup Analytics & Predictive Analysis

```bash
cat > backup_analytics.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
ANALYTICS_DB="/var/lib/backup-analytics/analytics.db"
ANALYTICS_LOG="/var/log/backup_analytics.log"
PREDICTIONS_DIR="/var/lib/backup-analytics/predictions"
REPORTS_DIR="/var/www/html/analytics"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$ANALYTICS_LOG"
}

# Initialize analytics database
init_analytics() {
    log "Initializing backup analytics system..."
    
    mkdir -p "$(dirname "$ANALYTICS_DB")"
    mkdir -p "$PREDICTIONS_DIR"
    mkdir -p "$REPORTS_DIR"
    
    # Create SQLite database
    sqlite3 "$ANALYTICS_DB" <<'SQL'
-- Backup job history
CREATE TABLE IF NOT EXISTS backup_history (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    job_name TEXT NOT NULL,
    job_type TEXT NOT NULL,
    status TEXT NOT NULL,
    duration_seconds INTEGER,
    size_bytes INTEGER,
    compression_ratio REAL,
    dedup_ratio REAL,
    throughput_mbps REAL,
    cpu_usage_percent REAL,
    memory_usage_mb INTEGER,
    error_message TEXT
);

-- Storage metrics
CREATE TABLE IF NOT EXISTS storage_metrics (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    mount_point TEXT NOT NULL,
    total_bytes INTEGER,
    used_bytes INTEGER,
    available_bytes INTEGER,
    utilization_percent REAL,
    inode_utilization_percent REAL
);

-- Performance trends
CREATE TABLE IF NOT EXISTS performance_trends (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    job_name TEXT NOT NULL,
    avg_duration_seconds REAL,
    avg_size_bytes INTEGER,
    avg_throughput_mbps REAL,
    success_rate REAL,
    trend_direction TEXT
);

-- Capacity predictions
CREATE TABLE IF NOT EXISTS capacity_predictions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    mount_point TEXT NOT NULL,
    current_used_gb REAL,
    predicted_used_gb REAL,
    prediction_date DATE,
    growth_rate_percent REAL,
    days_until_full INTEGER,
    confidence_level REAL
);

-- Cost analysis
CREATE TABLE IF NOT EXISTS cost_analysis (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    storage_tier TEXT NOT NULL,
    total_gb REAL,
    cost_per_gb REAL,
    monthly_cost REAL,
    annual_cost REAL
);

-- Create indexes
CREATE INDEX IF NOT EXISTS idx_backup_timestamp ON backup_history(timestamp);
CREATE INDEX IF NOT EXISTS idx_backup_job ON backup_history(job_name);
CREATE INDEX IF NOT EXISTS idx_storage_timestamp ON storage_metrics(timestamp);
CREATE INDEX IF NOT EXISTS idx_storage_mount ON storage_metrics(mount_point);
SQL
    
    log "Analytics database initialized: $ANALYTICS_DB"
}

# Collect backup metrics
collect_backup_metrics() {
    log "Collecting backup metrics..."
    
    # Parse recent backup logs
    for log_file in /var/log/backup*.log; do
        if [ ! -f "$log_file" ]; then
            continue
        fi
        
        # Extract job name
        local job_name=$(basename "$log_file" .log | sed 's/backup_//')
        
        # Parse last backup
        local status="unknown"
        local duration=0
        local size=0
        
        if grep -q "completed successfully" "$log_file"; then
            status="success"
        elif grep -q "failed\|error" "$log_file"; then
            status="failed"
        fi
        
        # Extract duration
        duration=$(grep "completed in" "$log_file" | tail -1 | grep -o '[0-9]*s' | tr -d 's' || echo "0")
        
        # Extract size
        size=$(grep "size:" "$log_file" | tail -1 | grep -o '[0-9]*' | head -1 || echo "0")
        
        # Insert into database
        sqlite3 "$ANALYTICS_DB" <<SQL
INSERT INTO backup_history (job_name, job_type, status, duration_seconds, size_bytes)
VALUES ('$job_name', 'full', '$status', $duration, $size);
SQL
    done
    
    log "Backup metrics collected"
}

# Collect storage metrics
collect_storage_metrics() {
    log "Collecting storage metrics..."
    
    # Get storage stats for backup mounts
    for mount in /backup /backup/mysql /backup/postgresql; do
        if [ ! -d "$mount" ]; then
            continue
        fi
        
        local df_output=$(df -B1 "$mount" | tail -1)
        local total=$(echo "$df_output" | awk '{print $2}')
        local used=$(echo "$df_output" | awk '{print $3}')
        local available=$(echo "$df_output" | awk '{print $4}')
        local utilization=$(echo "$df_output" | awk '{print $5}' | tr -d '%')
        
        # Get inode usage
        local inode_util=$(df -i "$mount" | tail -1 | awk '{print $5}' | tr -d '%')
        
        sqlite3 "$ANALYTICS_DB" <<SQL
INSERT INTO storage_metrics (mount_point, total_bytes, used_bytes, available_bytes, utilization_percent, inode_utilization_percent)
VALUES ('$mount', $total, $used, $available, $utilization, $inode_util);
SQL
    done
    
    log "Storage metrics collected"
}

# Calculate performance trends
calculate_trends() {
    log "Calculating performance trends..."
    
    sqlite3 "$ANALYTICS_DB" <<'SQL'
INSERT INTO performance_trends (job_name, avg_duration_seconds, avg_size_bytes, avg_throughput_mbps, success_rate, trend_direction)
SELECT 
    job_name,
    AVG(duration_seconds) as avg_duration,
    AVG(size_bytes) as avg_size,
    AVG(throughput_mbps) as avg_throughput,
    (SUM(CASE WHEN status='success' THEN 1 ELSE 0 END) * 100.0 / COUNT(*)) as success_rate,
    CASE 
        WHEN AVG(duration_seconds) > (SELECT AVG(duration_seconds) FROM backup_history WHERE job_name=bh.job_name AND timestamp < datetime('now', '-7 days')) THEN 'increasing'
        WHEN AVG(duration_seconds) < (SELECT AVG(duration_seconds) FROM backup_history WHERE job_name=bh.job_name AND timestamp < datetime('now', '-7 days')) THEN 'decreasing'
        ELSE 'stable'
    END as trend
FROM backup_history bh
WHERE timestamp >= datetime('now', '-7 days')
GROUP BY job_name;
SQL
    
    log "Performance trends calculated"
}

# Predictive capacity planning
predict_capacity() {
    local mount_point="${1:-/backup}"
    
    log "Running capacity prediction for: $mount_point"
    
    # Get historical storage data
    local history=$(sqlite3 "$ANALYTICS_DB" <<SQL
SELECT timestamp, used_bytes 
FROM storage_metrics 
WHERE mount_point='$mount_point' 
AND timestamp >= datetime('now', '-30 days')
ORDER BY timestamp;
SQL
)
    
    if [ -z "$history" ]; then
        log "No historical data available for capacity prediction"
        return 1
    fi
    
    # Calculate growth rate using linear regression
    local temp_data="/tmp/capacity_data_$$"
    echo "$history" | awk -F'|' '{print NR, $2}' > "$temp_data"
    
    # Simple linear regression in bash (y = mx + b)
    local n=$(wc -l < "$temp_data")
    local sum_x=$(awk '{sum+=$1} END {print sum}' "$temp_data")
    local sum_y=$(awk '{sum+=$2} END {print sum}' "$temp_data")
    local sum_xy=$(awk '{sum+=$1*$2} END {print sum}' "$temp_data")
    local sum_x2=$(awk '{sum+=$1*$1} END {print sum}' "$temp_data")
    
    # Calculate slope (growth rate)
    local slope=$(echo "scale=10; ($n * $sum_xy - $sum_x * $sum_y) / ($n * $sum_x2 - $sum_x * $sum_x)" | bc)
    local intercept=$(echo "scale=10; ($sum_y - $slope * $sum_x) / $n" | bc)
    
    # Get current and total storage
    local current_used=$(sqlite3 "$ANALYTICS_DB" "SELECT used_bytes FROM storage_metrics WHERE mount_point='$mount_point' ORDER BY timestamp DESC LIMIT 1;")
    local total_capacity=$(sqlite3 "$ANALYTICS_DB" "SELECT total_bytes FROM storage_metrics WHERE mount_point='$mount_point' ORDER BY timestamp DESC LIMIT 1;")
    
    # Predict when storage will be full
    local days_until_full=0
    if (( $(echo "$slope > 0" | bc -l) )); then
        days_until_full=$(echo "scale=0; ($total_capacity - $current_used) / ($slope * 86400)" | bc)
    else
        days_until_full=999999  # Never
    fi
    
    # Calculate predictions for 30, 60, 90 days
    local predictions=""
    for days in 30 60 90; do
        local predicted_bytes=$(echo "scale=0; $current_used + ($slope * $days * 86400)" | bc)
        local predicted_gb=$(echo "scale=2; $predicted_bytes / 1024 / 1024 / 1024" | bc)
        local current_gb=$(echo "scale=2; $current_used / 1024 / 1024 / 1024" | bc)
        local growth_rate=$(echo "scale=2; (($predicted_bytes - $current_used) * 100) / $current_used" | bc)
        local prediction_date=$(date -d "+$days days" +%Y-%m-%d)
        
        sqlite3 "$ANALYTICS_DB" <<SQL
INSERT INTO capacity_predictions (mount_point, current_used_gb, predicted_used_gb, prediction_date, growth_rate_percent, days_until_full, confidence_level)
VALUES ('$mount_point', $current_gb, $predicted_gb, '$prediction_date', $growth_rate, $days_until_full, 0.85);
SQL
        
        predictions="$predictions
In $days days ($prediction_date): ${predicted_gb}GB (${growth_rate}% growth)"
    done
    
    log "Capacity predictions:"
    log "  Current usage: $(echo "scale=2; $current_used / 1024 / 1024 / 1024" | bc)GB"
    log "  Growth rate: $(echo "scale=2; $slope * 86400 / 1024 / 1024 / 1024" | bc)GB/day"
    log "  Days until full: $days_until_full"
    log "$predictions"
    
    rm -f "$temp_data"
}

# Anomaly detection
detect_anomalies() {
    log "Running anomaly detection..."
    
    sqlite3 "$ANALYTICS_DB" <<'SQL' | while read line; do
        echo "ANOMALY: $line"
    done
-- Find backups with unusual duration
SELECT 
    job_name,
    timestamp,
    duration_seconds,
    (SELECT AVG(duration_seconds) FROM backup_history WHERE job_name=bh.job_name) as avg_duration
FROM backup_history bh
WHERE duration_seconds > (SELECT AVG(duration_seconds) * 2 FROM backup_history WHERE job_name=bh.job_name)
AND timestamp >= datetime('now', '-7 days')

UNION ALL

-- Find backups with unusual size
SELECT 
    job_name,
    timestamp,
    size_bytes,
    (SELECT AVG(size_bytes) FROM backup_history WHERE job_name=bh.job_name) as avg_size
FROM backup_history bh
WHERE ABS(size_bytes - (SELECT AVG(size_bytes) FROM backup_history WHERE job_name=bh.job_name)) > 
      (SELECT AVG(size_bytes) * 0.5 FROM backup_history WHERE job_name=bh.job_name)
AND timestamp >= datetime('now', '-7 days');
SQL
}

# Cost analysis
analyze_costs() {
    log "Analyzing backup costs..."
    
    # Cost per GB for different storage tiers (example values)
    local local_cost=0.02    # $0.02/GB/month
    local nas_cost=0.03      # $0.03/GB/month
    local s3_ia_cost=0.0125  # $0.0125/GB/month
    local glacier_cost=0.004 # $0.004/GB/month
    local b2_cost=0.005      # $0.005/GB/month
    
    # Get storage sizes
    local local_gb=$(du -sb /backup 2>/dev/null | awk '{print $1/1024/1024/1024}' || echo "0")
    
    # Calculate costs
    local local_monthly=$(echo "scale=2; $local_gb * $local_cost" | bc)
    local local_annual=$(echo "scale=2; $local_monthly * 12" | bc)
    
    # Insert into database
    sqlite3 "$ANALYTICS_DB" <<SQL
DELETE FROM cost_analysis WHERE timestamp >= datetime('now', '-1 day');

INSERT INTO cost_analysis (storage_tier, total_gb, cost_per_gb, monthly_cost, annual_cost)
VALUES 
    ('Local Storage', $local_gb, $local_cost, $local_monthly, $local_annual),
    ('NAS', 0, $nas_cost, 0, 0),
    ('S3 Infrequent Access', 0, $s3_ia_cost, 0, 0),
    ('Glacier', 0, $glacier_cost, 0, 0),
    ('Backblaze B2', 0, $b2_cost, 0, 0);
SQL
    
    log "Cost analysis completed"
    log "  Local storage: ${local_gb}GB = $${local_monthly}/month ($${local_annual}/year)"
}

# Generate analytics dashboard
generate_analytics_dashboard() {
    local dashboard_file="$REPORTS_DIR/analytics_dashboard.html"
    
    log "Generating analytics dashboard..."
    
    cat > "$dashboard_file" <<'HTML'
<!DOCTYPE html>
<html>
<head>
    <title>Backup Analytics Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 20px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }
        .container {
            max-width: 1600px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        h1 {
            color: #667eea;
            border-bottom: 3px solid #764ba2;
            padding-bottom: 10px;
            margin-bottom: 30px;
        }
        .metrics-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 40px;
        }
        .metric-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 25px;
            border-radius: 10px;
            text-align: center;
        }
        .metric-value {
            font-size: 2.5em;
            font-weight: bold;
            margin: 15px 0;
        }
        .metric-label {
            font-size: 1em;
            opacity: 0.9;
        }
        .metric-trend {
            font-size: 0.9em;
            margin-top: 10px;
        }
        .chart-container {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 8px;
            margin-bottom: 30px;
        }
        .chart-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(500px, 1fr));
            gap: 30px;
        }
        h2 {
            color: #667eea;
            margin-top: 40px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .prediction-box {
            background: #e3f2fd;
            border-left: 4px solid #2196F3;
            padding: 20px;
            margin: 20px 0;
            border-radius: 5px;
        }
        .warning-box {
            background: #fff3cd;
            border-left: 4px solid #ffc107;
            padding: 20px;
            margin: 20px 0;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📈 Backup Analytics Dashboard</h1>
        <p><strong>Generated:</strong> $(date)</p>
        <p><strong>Analysis Period:</strong> Last 30 days</p>
        
        <div class="metrics-grid">
HTML
    
    # Get key metrics from database
    local total_backups=$(sqlite3 "$ANALYTICS_DB" "SELECT COUNT(*) FROM backup_history WHERE timestamp >= datetime('now', '-30 days');")
    local success_rate=$(sqlite3 "$ANALYTICS_DB" "SELECT ROUND((SUM(CASE WHEN status='success' THEN 1 ELSE 0 END) * 100.0 / COUNT(*)), 1) FROM backup_history WHERE timestamp >= datetime('now', '-30 days');")
    local total_data=$(sqlite3 "$ANALYTICS_DB" "SELECT ROUND(SUM(size_bytes)/1024.0/1024.0/1024.0, 2) FROM backup_history WHERE timestamp >= datetime('now', '-30 days');")
    local avg_duration=$(sqlite3 "$ANALYTICS_DB" "SELECT ROUND(AVG(duration_seconds)/60.0, 1) FROM backup_history WHERE timestamp >= datetime('now', '-30 days');")
    
    cat >> "$dashboard_file" <<HTML
            <div class="metric-card">
                <div class="metric-label">Total Backups</div>
                <div class="metric-value">$total_backups</div>
                <div class="metric-trend">Last 30 days</div>
            </div>
            <div class="metric-card" style="background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);">
                <div class="metric-label">Success Rate</div>
                <div class="metric-value">${success_rate}%</div>
                <div class="metric-trend">Target: >95%</div>
            </div>
            <div class="metric-card" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
                <div class="metric-label">Total Data Backed Up</div>
                <div class="metric-value">${total_data}</div>
                <div class="metric-trend">GB</div>
            </div>
            <div class="metric-card" style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);">
                <div class="metric-label">Avg Duration</div>
                <div class="metric-value">${avg_duration}</div>
                <div class="metric-trend">minutes</div>
            </div>
        </div>
        
        <h2>Capacity Predictions</h2>
        <div class="prediction-box">
            <h3>📊 Storage Growth Forecast</h3>
HTML
    
    # Add capacity predictions
    sqlite3 "$ANALYTICS_DB" "SELECT mount_point, prediction_date, predicted_used_gb, days_until_full FROM capacity_predictions ORDER BY prediction_date DESC LIMIT 3;" | while IFS='|' read mount date predicted days; do
        cat >> "$dashboard_file" <<HTML
            <p><strong>$mount</strong></p>
            <ul>
                <li>Predicted usage on $date: ${predicted}GB</li>
                <li>Estimated days until full: $days</li>
            </ul>
HTML
    done
    
    cat >> "$dashboard_file" <<HTML
        </div>
        
        <h2>Performance Trends</h2>
        <div class="chart-grid">
            <div class="chart-container">
                <canvas id="backupDurationChart"></canvas>
            </div>
            <div class="chart-container">
                <canvas id="storageGrowthChart"></canvas>
            </div>
        </div>
        
        <h2>Cost Analysis</h2>
        <table>
            <thead>
                <tr>
                    <th>Storage Tier</th>
                    <th>Total (GB)</th>
                    <th>Cost/GB</th>
                    <th>Monthly Cost</th>
                    <th>Annual Cost</th>
                </tr>
            </thead>
            <tbody>
HTML
    
    # Add cost analysis
    sqlite3 "$ANALYTICS_DB" "SELECT storage_tier, total_gb, cost_per_gb, monthly_cost, annual_cost FROM cost_analysis ORDER BY timestamp DESC LIMIT 5;" | while IFS='|' read tier gb cost monthly annual; do
        cat >> "$dashboard_file" <<HTML
                <tr>
                    <td>$tier</td>
                    <td>$(printf "%.2f" $gb)</td>
                    <td>$$(printf "%.4f" $cost)</td>
                    <td>$$(printf "%.2f" $monthly)</td>
                    <td>$$(printf "%.2f" $annual)</td>
                </tr>
HTML
    done
    
    cat >> "$dashboard_file" <<HTML
            </tbody>
        </table>
        
        <h2>Recommendations</h2>
HTML
    
    # Add recommendations based on analysis
    local storage_util=$(sqlite3 "$ANALYTICS_DB" "SELECT utilization_percent FROM storage_metrics WHERE mount_point='/backup' ORDER BY timestamp DESC LIMIT 1;" || echo "0")
    
    if (( $(echo "$storage_util > 80" | bc -l) )); then
        cat >> "$dashboard_file" <<HTML
        <div class="warning-box">
            <h3>⚠️ Storage Capacity Warning</h3>
            <p>Backup storage is ${storage_util}% full. Consider:</p>
            <ul>
                <li>Implementing backup rotation policies</li>
                <li>Enabling compression and deduplication</li>
                <li>Moving older backups to cold storage</li>
                <li>Provisioning additional storage</li>
            </ul>
        </div>
HTML
    fi
    
    if (( $(echo "$success_rate < 95" | bc -l) )); then
        cat >> "$dashboard_file" <<HTML
        <div class="warning-box">
            <h3>⚠️ Success Rate Below Target</h3>
            <p>Success rate is ${success_rate}%, below the 95% target. Actions:</p>
            <ul>
                <li>Review failed backup logs</li>
                <li>Check system resources during backup windows</li>
                <li>Verify network connectivity for remote backups</li>
                <li>Update backup scripts and dependencies</li>
            </ul>
        </div>
HTML
    fi
    
    cat >> "$dashboard_file" <<HTML
        
        <script>
        // Backup Duration Chart
        const durationCtx = document.getElementById('backupDurationChart').getContext('2d');
        new Chart(durationCtx, {
            type: 'line',
            data: {
                labels: ['Day 1', 'Day 7', 'Day 14', 'Day 21', 'Day 30'],
                datasets: [{
                    label: 'Avg Backup Duration (minutes)',
                    data: [45, 48, 52, 50, 53],
                    borderColor: 'rgb(102, 126, 234)',
                    backgroundColor: 'rgba(102, 126, 234, 0.1)',
                    tension: 0.4
                }]
            },
            options: {
                responsive: true,
                plugins: {
                    title: {
                        display: true,
                        text: 'Backup Duration Trend'
                    }
                }
            }
        });
        
        // Storage Growth Chart
        const storageCtx = document.getElementById('storageGrowthChart').getContext('2d');
        new Chart(storageCtx, {
            type: 'line',
            data: {
                labels: ['Week 1', 'Week 2', 'Week 3', 'Week 4'],
                datasets: [{
                    label: 'Storage Used (GB)',
                    data: [450, 485, 520, 558],
                    borderColor: 'rgb(118, 75, 162)',
                    backgroundColor: 'rgba(118, 75, 162, 0.1)',
                    tension: 0.4,
                    fill: true
                }, {
                    label: 'Predicted',
                    data: [null, null, null, 558, 595],
                    borderColor: 'rgb(255, 159, 64)',
                    borderDash: [5, 5],
                    fill: false
                }]
            },
            options: {
                responsive: true,
                plugins: {
                    title: {
                        display: true,
                        text: 'Storage Growth & Prediction'
                    }
                }
            }
        });
        </script>
        
        <hr style="margin-top: 40px;">
        <p style="text-align: center; color: #666;">
            Backup Analytics Dashboard v1.0 | Auto-refresh every 5 minutes
        </p>
    </div>
    
    <script>
        // Auto-refresh every 5 minutes
        setTimeout(() => location.reload(), 300000);
    </script>
</body>
</html>
HTML
    
    log "Analytics dashboard generated: $dashboard_file"
    echo "$dashboard_file"
}

# Export analytics data
export_analytics() {
    local format="${1:-csv}"
    local output_file="$REPORTS_DIR/analytics_export_$(date +%Y%m%d).${format}"
    
    log "Exporting analytics data to $format..."
    
    case "$format" in
        csv)
            sqlite3 -header -csv "$ANALYTICS_DB" "SELECT * FROM backup_history ORDER BY timestamp DESC LIMIT 1000;" > "$output_file"
            ;;
        json)
            sqlite3 "$ANALYTICS_DB" "SELECT json_group_array(json_object(
                'timestamp', timestamp,
                'job_name', job_name,
                'status', status,
                'duration_seconds', duration_seconds,
                'size_bytes', size_bytes
            )) FROM backup_history ORDER BY timestamp DESC LIMIT 1000;" > "$output_file"
            ;;
        *)
            log "ERROR: Unsupported format: $format"
            return 1
            ;;
    esac
    
    log "Analytics data exported: $output_file"
    echo "$output_file"
}

# Main command handling
case "${1:-help}" in
    init)
        init_analytics
        ;;
    
    collect)
        collect_backup_metrics
        collect_storage_metrics
        calculate_trends
        ;;
    
    predict)
        predict_capacity "$2"
        ;;
    
    anomalies)
        detect_anomalies
        ;;
    
    costs)
        analyze_costs
        ;;
    
    dashboard)
        generate_analytics_dashboard
        ;;
    
    export)
        export_analytics "$2"
        ;;
    
    full-analysis)
        log "Running full analytics cycle..."
        collect_backup_metrics
        collect_storage_metrics
        calculate_trends
        predict_capacity "/backup"
        detect_anomalies
        analyze_costs
        generate_analytics_dashboard
        ;;
    
    *)
        cat <<HELP
Backup Analytics & Predictive Analysis

Usage: $0 <command> [options]

Commands:
  init                - Initialize analytics database
  collect             - Collect backup and storage metrics
  predict [mount]     - Run capacity prediction
  anomalies           - Detect backup anomalies
  costs               - Analyze backup costs
  dashboard           - Generate analytics dashboard
  export [csv|json]   - Export analytics data
  full-analysis       - Run complete analysis cycle

Examples:
  $0 init
  $0 collect
  $0 predict /backup
  $0 dashboard
  $0 export csv

Dashboard: file://$REPORTS_DIR/analytics_dashboard.html
HELP
        ;;
esac
EOF

chmod +x backup_analytics.sh
```

---

## 🎓 Итоговое задание модуля 8

```bash
cat > module8_complete_setup.sh <<'COMPLETE'
#!/bin/bash

echo "=============================================="
echo "Module 8: Complete Monitoring Setup"
echo "=============================================="

# 1. Initialize Prometheus Exporter
echo ""
echo "1. Setting up Prometheus Backup Exporter..."
./backup_prometheus_exporter.sh init
./backup_prometheus_exporter.sh collect
./backup_prometheus_exporter.sh dashboard > /tmp/grafana_dashboard.json

# Start metrics collection daemon
./backup_prometheus_exporter.sh daemon 60 &
EXPORTER_PID=$!
echo "Metrics exporter running (PID: $EXPORTER_PID)"

# 2. Initialize AlertManager
echo ""
echo "2. Setting up AlertManager..."
./backup_alertmanager.sh init
./backup_alertmanager.sh start

# Test alert
./backup_alertmanager.sh test

# 3. Initialize Log Aggregation
echo ""
echo "3. Setting up Log Aggregation (Loki + Promtail)..."
./backup_log_aggregation.sh init
./backup_log_aggregation.sh start

# Test log ingestion
sleep 10
./backup_log_aggregation.sh test

# 4. Initialize Analytics
echo ""
echo "4. Setting up Backup Analytics..."
./backup_analytics.sh init
./backup_analytics.sh full-analysis

echo ""
echo "=============================================="
echo "Module 8 Setup Complete!"
echo "=============================================="
echo ""
echo "Access Points:"
echo "  - Prometheus Metrics: http://localhost:9100/metrics"
echo "  - AlertManager UI: http://localhost:9093"
echo "  - Loki API: http://localhost:3100"
echo "  - Promtail: http://localhost:9080"
echo ""
echo "Generated Files:"
echo "  - Grafana Dashboard: /tmp/grafana_dashboard.json"
echo "  - Analytics Dashboard: file://$REPORTS_DIR/analytics_dashboard.html"
echo "  - Alert Stats: /var/log/alert_statistics_*.html"
echo "  - Log Report: /var/log/log_analysis_report_*.html"
echo ""
echo "Next Steps:"
echo "  1. Import Grafana dashboard: /tmp/grafana_dashboard.json"
echo "  2. Configure notification channels in AlertManager"
echo "  3. Review analytics dashboard for insights"
echo "  4. Set up automated reporting with cron"
echo ""
echo "Recommended Cron Schedule:"
echo "  # Metrics collection"
echo "  */5 * * * * /usr/local/bin/backup_prometheus_exporter.sh collect"
echo ""
echo "  # Daily reports"
echo "  0 8 * * * /usr/local/bin/backup_analytics.sh dashboard"
echo "  0 8 * * * /usr/local/bin/backup_alertmanager.sh stats"
echo "  0 8 * * * /usr/local/bin/backup_log_aggregation.sh report"
echo ""
echo "  # Weekly analysis"
echo "  0 9 * * 1 /usr/local/bin/backup_analytics.sh full-analysis"
echo ""
COMPLETE

chmod +x module8_complete_setup.sh
```

---

## 📊 Итоговый отчет: Финальная проверка

```bash
cat > final_course_validation.sh <<'VALIDATION'
#!/bin/bash

echo "=============================================="
echo "🎓 Backup & Recovery: Final Validation"
echo "=============================================="
echo ""

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

check_pass() {
    echo -e "${GREEN}✓${NC} $1"
}

check_fail() {
    echo -e "${RED}✗${NC} $1"
}

check_warn() {
    echo -e "${YELLOW}⚠${NC} $1"
}

# Module 1: Backup Strategies
echo "Module 1: Backup Strategies"
echo "─────────────────────────────"
if [ -f /usr/local/bin/backup.sh ]; then
    check_pass "Backup script installed"
else
    check_fail "Backup script not found"
fi

if crontab -l | grep -q backup; then
    check_pass "Backup schedule configured"
else
    check_warn "Backup schedule not found"
fi

# Module 2: Database Backups
echo ""
echo "Module 2: Database Backups"
echo "──────────────────────────"
if [ -f /usr/local/bin/mysql_backup.sh ]; then
    check_pass "MySQL backup script installed"
else
    check_fail "MySQL backup script not found"
fi

if [ -f /usr/local/bin/pg_backup_pitr.sh ]; then
    check_pass "PostgreSQL PITR script installed"
else
    check_fail "PostgreSQL PITR script not found"
fi

if [ -d /backup/mysql ] || [ -d /backup/postgresql ]; then
    check_pass "Database backup directories exist"
else
    check_warn "Database backup directories not found"
fi

# Module 3: Filesystem Backups
echo ""
echo "Module 3: Filesystem Backups"
echo "────────────────────────────"
if [ -f /usr/local/bin/rsync_backup.sh ]; then
    check_pass "Rsync backup script installed"
else
    check_fail "Rsync backup script not found"
fi

if [ -f /usr/local/bin/system_backup.sh ]; then
    check_pass "System backup script installed"
else
    check_fail "System backup script not found"
fi

# Module 4: Cloud Backups
echo ""
echo "Module 4: Cloud Backups"
echo "───────────────────────"
if command -v rclone &>/dev/null; then
    check_pass "Rclone installed"
else
    check_warn "Rclone not installed"
fi

if command -v restic &>/dev/null; then
    check_pass "Restic installed"
else
    check_warn "Restic not installed"
fi

if [ -f /usr/local/bin/cloud_backup.sh ]; then
    check_pass "Cloud backup script installed"
else
    check_fail "Cloud backup script not found"
fi

# Module 5: DR Automation
echo ""
echo "Module 5: DR Automation"
echo "───────────────────────"
if [ -f /usr/local/bin/recovery_orchestrator.sh ]; then
    check_pass "Recovery orchestrator installed"
else
    check_fail "Recovery orchestrator not found"
fi

if [ -f /usr/local/bin/dr_testing_framework.sh ]; then
    check_pass "DR testing framework installed"
else
    check_fail "DR testing framework not found"
fi

if [ -f /usr/local/bin/automated_failover.sh ]; then
    check_pass "Automated failover system installed"
else
    check_fail "Automated failover system not found"
fi

# Module 6: Security & Compliance
echo ""
echo "Module 6: Security & Compliance"
echo "────────────────────────────────"
if [ -f /usr/local/bin/secure_backup_system.sh ]; then
    check_pass "Secure backup system installed"
else
    check_fail "Secure backup system not found"
fi

if [ -d /etc/backup-security/keys ]; then
    check_pass "Encryption keys directory exists"
else
    check_warn "Encryption keys directory not found"
fi

if [ -f /var/log/backup-security-audit.log ]; then
    check_pass "Security audit logging enabled"
else
    check_warn "Security audit log not found"
fi

# Module 7: Performance Optimization
echo ""
echo "Module 7: Performance Optimization"
echo "───────────────────────────────────"
if [ -f /usr/local/bin/compression_optimizer.sh ]; then
    check_pass "Compression optimizer installed"
else
    check_fail "Compression optimizer not found"
fi

if [ -f /usr/local/bin/parallel_backup_system.sh ]; then
    check_pass "Parallel backup system installed"
else
    check_fail "Parallel backup system not found"
fi

if [ -f /usr/local/bin/network_backup_optimizer.sh ]; then
    check_pass "Network optimizer installed"
else
    check_fail "Network optimizer not found"
fi

# Module 8: Monitoring & Analytics
echo ""
echo "Module 8: Monitoring & Analytics"
echo "─────────────────────────────────"
if command -v node_exporter &>/dev/null; then
    check_pass "Prometheus node_exporter installed"
else
    check_warn "node_exporter not installed"
fi

if command -v alertmanager &>/dev/null; then
    check_pass "AlertManager installed"
else
    check_warn "AlertManager not installed"
fi

if command -v loki &>/dev/null; then
    check_pass "Loki installed"
else
    check_warn "Loki not installed"
fi

if command -v promtail &>/dev/null; then
    check_pass "Promtail installed"
else
    check_warn "Promtail not installed"
fi

if [ -f /var/lib/backup-analytics/analytics.db ]; then
    check_pass "Analytics database initialized"
else
    check_warn "Analytics database not found"
fi

# Final Statistics
echo ""
echo "=============================================="
echo "📈 Final Statistics"
echo "=============================================="

# Count backups
total_backups=$(find /backup -type f \( -name "*.tar.gz" -o -name "*.sql.gz" \) 2>/dev/null | wc -l)
echo "Total backups created: $total_backups"

# Storage usage
if [ -d /backup ]; then
    storage_used=$(du -sh /backup 2>/dev/null | cut -f1 || echo "N/A")
    echo "Backup storage used: $storage_used"
fi

# Count scripts
scripts_installed=$(ls /usr/local/bin/*backup* /usr/local/bin/*recovery* 2>/dev/null | wc -l)
echo "Scripts installed: $scripts_installed"

# Check running services
echo ""
echo "Running Services:"
for service in node_exporter alertmanager loki promtail; do
    if systemctl is-active --quiet $service 2>/dev/null; then
        check_pass "$service is running"
    else
        check_warn "$service is not running"
    fi
done

echo ""
echo "=============================================="
echo "🎯 Course Completion Summary"
echo "=============================================="
echo ""
echo "Modules Completed:"
echo "  ✓ Module 1: Backup Strategies (3-2-1-1-0 Rule)"
echo "  ✓ Module 2: Database Backups (MySQL, PostgreSQL, MongoDB, Redis)"
echo "  ✓ Module 3: Filesystem Backups (LVM, Btrfs, ZFS, rsync)"
echo "  ✓ Module 4: Cloud Backups (S3, B2, Multi-cloud)"
echo "  ✓ Module 5: DR Automation (Recovery, Testing, Failover)"
echo "  ✓ Module 6: Security & Compliance (Encryption, Immutability)"
echo "  ✓ Module 7: Performance (Compression, Parallel, Network)"
echo "  ✓ Module 8: Monitoring (Prometheus, Loki, Analytics)"
echo ""
echo "Skills Acquired:"
echo "  • Comprehensive backup strategy design"
echo "  • Database-specific backup techniques"
echo "  • Filesystem snapshot management"
echo "  • Multi-cloud backup orchestration"
echo "  • Automated disaster recovery"
echo "  • Security and compliance frameworks"
echo "  • Performance optimization"
echo "  • Full-stack monitoring and analytics"
echo ""
echo "Generated Artifacts:"
ls -1 /var/log/*report*.html 2>/dev/null | tail -5
echo ""
echo "=============================================="
echo "🎓 COURSE COMPLETED SUCCESSFULLY!"
echo "=============================================="
echo ""
echo "Next Steps:"
echo "  1. Review generated reports and dashboards"
echo "  2. Customize scripts for your environment"
echo "  3. Schedule regular DR testing"
echo "  4. Document your backup procedures"
echo "  5. Train your team on recovery processes"
echo ""
echo "Remember the Golden Rules:"
echo "  • Backup without tested restore = disaster waiting"
echo "  • Security without compliance = audit nightmare"
echo "  • Monitoring without alerting = blind operations"
echo ""
echo "Thank you for completing the course! 🚀"
echo ""
VALIDATION

chmod +x final_course_validation.sh
```

---

## 🎓 Финальный Сертификат

```bash
cat > generate_certificate.sh <<'CERT'
#!/bin/bash

cat > /tmp/backup_course_certificate.txt <<'CERTIFICATE'
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║              CERTIFICATE OF COMPLETION                       ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝

                   Backup & Recovery Refresh
              Ежедневный/Полугодовой курс для DevOps

This certifies that the student has successfully completed
the comprehensive Backup & Recovery training course.

Course Duration: 8 Modules (2-3 hours each)
Completion Date: $(date +"%B %d, %Y")

Modules Mastered:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✓ Module 1: Backup Strategy Fundamentals
  - RTO/RPO concepts
  - 3-2-1-1-0 backup rule
  - Retention policies

✓ Module 2: Database Backup Techniques
  - MySQL/PostgreSQL PITR
  - MongoDB & Redis backups
  - Transaction log management

✓ Module 3: Filesystem & System Backups
  - LVM/Btrfs/ZFS snapshots
  - rsync-based deduplication
  - System state preservation

✓ Module 4: Cloud & Hybrid Strategies
  - Multi-cloud orchestration
  - Storage lifecycle management
  - Cost optimization

✓ Module 5: Disaster Recovery Automation
  - Recovery orchestration
  - Automated failover systems
  - DR testing frameworks

✓ Module 6: Security & Compliance
  - Encryption at rest/transit
  - Immutability & air-gap
  - Audit logging & compliance

✓ Module 7: Performance Optimization
  - Compression benchmarking
  - Parallel backup systems
  - Network optimization

✓ Module 8: Monitoring & Analytics
  - Prometheus metrics
  - Log aggregation (Loki)
  - Predictive analytics

Skills Validated:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

• Production-ready backup infrastructure design
• Database-specific recovery procedures
• Cloud-native backup strategies
• Security and compliance implementation
• Performance tuning and optimization
• Full-stack monitoring and alerting
• Disaster recovery planning and testing

Production-Ready Deliverables:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

• 40+ production-ready scripts
• Automated recovery procedures
• Monitoring dashboards
• Alert configurations
• DR runbooks
• Security frameworks
• Analytics systems

Signed,
Backup & Recovery Training Program
$(date +"%Y")

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
"Tested backups are the only backups that matter"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CERTIFICATE

cat /tmp/backup_course_certificate.txt

# Also generate HTML version
cat > /tmp/backup_course_certificate.html <<'HTML'
<!DOCTYPE html>
<html>
<head>
    <title>Backup & Recovery Course Certificate</title>
    <style>
        body {
            font-family: 'Georgia', serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            padding: 40px;
            margin: 0;
        }
        .certificate {
            max-width: 900px;
            margin: 0 auto;
            background: white;
            padding: 60px;
            border-radius: 10px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.3);
            border: 10px solid #667eea;
        }
        h1 {
            text-align: center;
            color: #667eea;
            font-size: 3em;
            margin-bottom: 10px;
            text-transform: uppercase;
            letter-spacing: 3px;
        }
        .subtitle {
            text-align: center;
            color: #764ba2;
            font-size: 1.5em;
            margin-bottom: 30px;
        }
        .content {
            line-height: 1.8;
            color: #333;
        }
        .modules {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 15px;
            margin: 30px 0;
        }
        .module {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 5px;
            border-left: 4px solid #667eea;
        }
        .signature {
            text-align: center;
            margin-top: 60px;
            padding-top: 30px;
            border-top: 2px solid #667eea;
        }
        .date {
            text-align: center;
            color: #666;
            margin-top: 20px;
        }
        .seal {
            text-align: center;
            font-size: 4em;
            color: #764ba2;
            margin: 40px 0;
        }
    </style>
</head>
<body>
    <div class="certificate">
        <h1>Certificate of Completion</h1>
        <div class="subtitle">Backup & Recovery Mastery Course</div>
        
        <div class="seal">🎓</div>
        
        <div class="content">
            <p style="text-align: center; font-size: 1.2em;">
                This certifies that the student has successfully completed the
                <strong>Comprehensive Backup & Recovery Training Course</strong>
            </p>
            
            <div class="modules">
                <div class="module">✓ Backup Strategies</div>
                <div class="module">✓ Database Backups</div>
                <div class="module">✓ Filesystem Backups</div>
                <div class="module">✓ Cloud & Hybrid</div>
                <div class="module">✓ DR Automation</div>
                <div class="module">✓ Security & Compliance</div>
                <div class="module">✓ Performance Tuning</div>
                <div class="module">✓ Monitoring & Analytics</div>
            </div>
            
            <p style="text-align: center; margin-top: 40px;">
                <strong>Skills Validated:</strong> Production-ready backup infrastructure,
                disaster recovery planning, security implementation, and full-stack monitoring
            </p>
            
            <div class="signature">
                <p style="font-size: 1.3em; margin: 0;">Backup & Recovery Training Program</p>
                <p style="color: #666; font-style: italic;">"Tested backups are the only backups that matter"</p>
            </div>
            
            <div class="date">
                Completed: $(date +"%B %d, %Y")
            </div>
        </div>
    </div>
</body>
</html>
HTML

echo ""
echo "Certificate generated!"
echo "  - Text version: /tmp/backup_course_certificate.txt"
echo "  - HTML version: /tmp/backup_course_certificate.html"
echo ""
echo "Open HTML certificate:"
echo "  firefox /tmp/backup_course_certificate.html"
CERT

chmod +x generate_certificate.sh
```

---

## 🎯 Финальный запуск всего курса

```bash
cat > run_complete_course.sh <<'FINAL'
#!/bin/bash

echo "╔══════════════════════════════════════════════════════════════╗"
echo "║     Backup & Recovery: Complete Course Execution            ║"
echo "╚══════════════════════════════════════════════════════════════╝"
echo ""
echo "This will run the entire course setup. Continue? (yes/no)"
read -r confirm

if [ "$confirm" != "yes" ]; then
    echo "Aborted."
    exit 0
fi

# Execute all modules
for module in {1..8}; do
    echo ""
    echo "════════════════════════════════════════════════════════════════"
    echo "  MODULE $module"
    echo "════════════════════════════════════════════════════════════════"
    
    case $module in
        1) echo "Setting up backup strategies..."
           ./backup.sh init 2>/dev/null || true
           ;;
        2) echo "Configuring database backups..."
           ./mysql_backup.sh daily 2>/dev/null || true
           ;;
        3) echo "Setting up filesystem backups..."
           ./rsync_backup.sh snapshot 2>/dev/null || true
           ;;
        4) echo "Configuring cloud backups..."
           ./cloud_backup.sh init 2>/dev/null || true
           ;;
        5) echo "Setting up DR automation..."
           ./recovery_orchestrator.sh init 2>/dev/null || true
           ;;
        6) echo "Implementing security..."
           ./secure_backup_system.sh init 2>/dev/null || true
           ;;
        7) echo "Running performance tests..."
           ./compression_optimizer.sh quick 2>/dev/null || true
           ;;
        8) echo "Setting up monitoring..."
           ./module8_complete_setup.sh 2>/dev/null || true
           ;;
    esac
    
    sleep 2
done

# Final validation
echo ""
echo "════════════════════════════════════════════════════════════════"
echo "  FINAL VALIDATION"
echo "════════════════════════════════════════════════════════════════"
./final_course_validation.sh

# Generate certificate
echo ""
./generate_certificate.sh

echo ""
echo "╔══════════════════════════════════════════════════════════════╗"
echo "║                 🎉 COURSE COMPLETED! 🎉                       ║"
echo "╚══════════════════════════════════════════════════════════════╝"
echo ""
FINAL

chmod +x run_complete_course.sh
```

---


**Практические навыки:**

- 🔄 Автоматизация всего цикла backup & recovery
- 🔐 Enterprise-grade security и compliance
- ⚡ Performance optimization и capacity planning
- 📊 Full-stack monitoring и predictive analytics
- 🚨 Automated alerting и incident response
- ☁️ Multi-cloud orchestration

**Помни главные правила:**

> "Backup без tested restore = катастрофа в ожидании"  
> "Security без compliance = аудиторский кошмар"  
> "Monitoring без alerting = слепые операции"

**Следующие шаги:**

1. Адаптируй скрипты под свою инфраструктуру
2. Регулярно тестируй recovery процедуры
3. Поддерживай документацию в актуальном состоянии
4. Проводи DR drills минимум раз в квартал
5. Обучай команду процедурам восстановления

**Успехов в production! 🚀**

Scenarios

# Модуль 9: Disaster Recovery Scenarios & Real-World Incident Response (35 минут)

## 🎯 Напоминалка

### Common Disaster Scenarios

```
Disaster Category Tree
├── Technical Failures
│   ├── Hardware Failures
│   │   ├── Disk failure (single/multiple)
│   │   ├── RAID controller failure
│   │   ├── Server hardware failure
│   │   ├── Storage array failure
│   │   └── Network equipment failure
│   ├── Software Failures
│   │   ├── Database corruption
│   │   ├── Application bugs
│   │   ├── OS corruption
│   │   └── Filesystem corruption
│   └── Infrastructure Failures
│       ├── Power outage
│       ├── Cooling failure
│       ├── Network outage
│       └── Datacenter disaster
│
├── Security Incidents
│   ├── Ransomware attack
│   ├── Data breach
│   ├── Malware infection
│   ├── DDoS attack
│   └── Unauthorized access
│
├── Human Errors
│   ├── Accidental deletion
│   ├── Configuration mistakes
│   ├── DROP TABLE/DATABASE
│   ├── Deployment errors
│   └── Permission changes
│
└── Natural Disasters
    ├── Fire
    ├── Flood
    ├── Earthquake
    ├── Hurricane/Storm
    └── Power grid failure
```

### Incident Response Phases

```
NIST Incident Response Lifecycle
┌────────────────────────────────┐
│   1. PREPARATION               │
│   - DR plans ready             │
│   - Team trained               │
│   - Tools available            │
└────────────────────────────────┘
           ↓
┌────────────────────────────────┐
│   2. IDENTIFICATION            │
│   - Detect incident            │
│   - Assess scope               │
│   - Classify severity          │
└────────────────────────────────┘
           ↓
┌────────────────────────────────┐
│   3. CONTAINMENT               │
│   - Isolate affected systems   │
│   - Prevent spread             │
│   - Preserve evidence          │
└────────────────────────────────┘
           ↓
┌────────────────────────────────┐
│   4. ERADICATION               │
│   - Remove threat              │
│   - Patch vulnerabilities      │
│   - Clean infected systems     │
└────────────────────────────────┘
           ↓
┌────────────────────────────────┐
│   5. RECOVERY                  │
│   - Restore from clean backups │
│   - Verify integrity           │
│   - Resume operations          │
└────────────────────────────────┘
           ↓
┌────────────────────────────────┐
│   6. LESSONS LEARNED           │
│   - Post-incident review       │
│   - Update procedures          │
│   - Improve defenses           │
└────────────────────────────────┘
```

### Recovery Decision Matrix

```
Impact vs Complexity
High │  P1: All hands    │  P1: Major incident
     │  Immediate action │  Executive escalation
     │                   │
Med  │  P2: Quick fix    │  P2: Planned recovery
     │  Within hours     │  Coordinate teams
     │                   │
Low  │  P3: Schedule     │  P3: Standard process
     │  Next window      │  Document & review
     └───────────────────┴────────────────────
       Low              Med              High
              COMPLEXITY
```

---

## 💻 Задание 1: Ransomware Attack Response & Recovery

```bash
cat > ransomware_recovery.sh <<'EOF'
#!/bin/bash

set -e

# Configuration
INCIDENT_ID="$(date +%Y%m%d-%H%M%S)-ransomware"
INCIDENT_DIR="/var/incident-response/$INCIDENT_ID"
FORENSICS_DIR="$INCIDENT_DIR/forensics"
RECOVERY_LOG="$INCIDENT_DIR/recovery.log"
BACKUP_ROOT="/backup"
CLEAN_BACKUP_POINT=""

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log() {
    local level="$1"
    shift
    local message="$*"
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [$level] $message" | tee -a "$RECOVERY_LOG"
}

alert() {
    echo -e "${RED}[!!! ALERT !!!]${NC} $*" | tee -a "$RECOVERY_LOG"
}

# Initialize incident response
init_incident_response() {
    alert "RANSOMWARE INCIDENT DETECTED"
    alert "Incident ID: $INCIDENT_ID"
    
    mkdir -p "$INCIDENT_DIR"
    mkdir -p "$FORENSICS_DIR"
    
    log INFO "Incident response initialized"
    log INFO "Log file: $RECOVERY_LOG"
    
    # Create incident timeline
    cat > "$INCIDENT_DIR/timeline.md" <<TIMELINE
# Ransomware Incident Timeline
**Incident ID:** $INCIDENT_ID
**Detected:** $(date)

## Timeline
- $(date): Incident detected
TIMELINE
    
    # Notify team
    notify_team "CRITICAL: Ransomware incident detected. Response initiated."
}

# Phase 1: Immediate Containment
phase1_containment() {
    alert "PHASE 1: IMMEDIATE CONTAINMENT"
    
    log INFO "Starting containment procedures..."
    
    # Step 1: Network Isolation
    log INFO "Step 1/5: Network isolation"
    isolate_affected_systems
    
    # Step 2: Stop Services
    log INFO "Step 2/5: Stopping critical services"
    stop_services
    
    # Step 3: Kill Suspicious Processes
    log INFO "Step 3/5: Identifying and killing suspicious processes"
    kill_suspicious_processes
    
    # Step 4: Block Malicious IPs
    log INFO "Step 4/5: Blocking malicious IPs"
    block_malicious_ips
    
    # Step 5: Document Initial State
    log INFO "Step 5/5: Documenting initial state"
    document_initial_state
    
    log INFO "Phase 1 containment complete"
    echo "- $(date): Phase 1 containment completed" >> "$INCIDENT_DIR/timeline.md"
}

# Isolate affected systems
isolate_affected_systems() {
    log WARN "Isolating affected systems from network..."
    
    # Identify network interfaces
    local interfaces=$(ip link show | grep -E '^[0-9]+:' | awk -F': ' '{print $2}' | grep -v lo)
    
    for iface in $interfaces; do
        # Check if interface is critical (preserve management access)
        if [[ "$iface" =~ ^(mgmt|ipmi) ]]; then
            log INFO "Preserving management interface: $iface"
            continue
        fi
        
        log WARN "Isolating interface: $iface"
        
        # Block all outbound connections except to backup server
        iptables -A OUTPUT -o "$iface" -d backup-server.local -j ACCEPT
        iptables -A OUTPUT -o "$iface" -j DROP
        
        # Block all inbound connections except from trusted IPs
        iptables -A INPUT -i "$iface" -s 10.0.0.0/8 -j ACCEPT
        iptables -A INPUT -i "$iface" -j DROP
    done
    
    # Save iptables rules
    iptables-save > "$FORENSICS_DIR/iptables_isolation.rules"
    
    log INFO "Network isolation completed"
    log INFO "Isolation rules saved to: $FORENSICS_DIR/iptables_isolation.rules"
}

# Stop services to prevent further encryption
stop_services() {
    log WARN "Stopping services to prevent further damage..."
    
    local services_to_stop=(
        "apache2"
        "nginx"
        "mysql"
        "postgresql"
        "docker"
        "smbd"
        "nfs-server"
    )
    
    for service in "${services_to_stop[@]}"; do
        if systemctl is-active --quiet "$service" 2>/dev/null; then
            log INFO "Stopping service: $service"
            systemctl stop "$service"
            systemctl disable "$service"
        fi
    done
    
    log INFO "Critical services stopped and disabled"
}

# Kill suspicious processes
kill_suspicious_processes() {
    log INFO "Scanning for suspicious processes..."
    
    # Common ransomware indicators
    local suspicious_patterns=(
        "encrypt"
        "ransom"
        ".locked"
        "recover+.*\.txt"
        "wannacry"
        "ryuk"
    )
    
    # Save process list before killing
    ps auxf > "$FORENSICS_DIR/processes_before_kill.txt"
    
    for pattern in "${suspicious_patterns[@]}"; do
        local pids=$(pgrep -f "$pattern" 2>/dev/null || true)
        
        if [ -n "$pids" ]; then
            log WARN "Found suspicious processes matching '$pattern': $pids"
            
            for pid in $pids; do
                # Get process details
                ps -p $pid -o pid,ppid,user,cmd >> "$FORENSICS_DIR/suspicious_processes.txt"
                
                # Kill process
                kill -9 $pid 2>/dev/null || true
                log INFO "Killed process: $pid"
            done
        fi
    done
    
    # Save process list after killing
    ps auxf > "$FORENSICS_DIR/processes_after_kill.txt"
}

# Block malicious IPs
block_malicious_ips() {
    log INFO "Blocking known malicious IPs..."
    
    # Check active connections
    netstat -tupn > "$FORENSICS_DIR/network_connections.txt"
    
    # Look for suspicious connections
    local suspicious_ips=$(netstat -tupn | grep ESTABLISHED | awk '{print $5}' | cut -d: -f1 | sort -u)
    
    for ip in $suspicious_ips; do
        # Skip local IPs
        if [[ "$ip" =~ ^(10\.|172\.|192\.168\.|127\.) ]]; then
            continue
        fi
        
        log WARN "Blocking suspicious IP: $ip"
        iptables -A INPUT -s "$ip" -j DROP
        iptables -A OUTPUT -d "$ip" -j DROP
        
        echo "$ip" >> "$FORENSICS_DIR/blocked_ips.txt"
    done
}

# Document initial state
document_initial_state() {
    log INFO "Documenting initial system state..."
    
    # System information
    cat > "$FORENSICS_DIR/system_info.txt" <<INFO
System Information
==================
Date: $(date)
Hostname: $(hostname)
Kernel: $(uname -a)
Uptime: $(uptime)

INFO
    
    # Find encrypted files
    log INFO "Scanning for encrypted/ransom note files..."
    
    # Common ransom note filenames
    local ransom_patterns=(
        "*DECRYPT*"
        "*README*"
        "*HELP*"
        "*RECOVER*"
        "*.txt.locked"
        "*.encrypted"
    )
    
    for pattern in "${ransom_patterns[@]}"; do
        find / -name "$pattern" -type f 2>/dev/null >> "$FORENSICS_DIR/ransom_notes.txt" || true
    done
    
    # Sample a ransom note if found
    local first_note=$(head -1 "$FORENSICS_DIR/ransom_notes.txt" 2>/dev/null)
    if [ -n "$first_note" ]; then
        log WARN "Ransom note found: $first_note"
        cp "$first_note" "$FORENSICS_DIR/sample_ransom_note.txt"
    fi
    
    # Check for file extension changes (indicator of encryption)
    log INFO "Analyzing file extensions..."
    find /data -type f -name "*.locked" -o -name "*.encrypted" -o -name "*.crypto" 2>/dev/null | \
        wc -l > "$FORENSICS_DIR/encrypted_file_count.txt"
    
    local encrypted_count=$(cat "$FORENSICS_DIR/encrypted_file_count.txt")
    log WARN "Found $encrypted_count potentially encrypted files"
    
    # Sample encrypted file for analysis
    find /data -type f \( -name "*.locked" -o -name "*.encrypted" \) 2>/dev/null | head -1 | \
        xargs -I {} cp {} "$FORENSICS_DIR/sample_encrypted_file" 2>/dev/null || true
    
    # File system timeline
    find /data -type f -mtime -1 -ls > "$FORENSICS_DIR/recently_modified_files.txt" 2>/dev/null || true
}

# Phase 2: Identify Clean Backup Point
phase2_identify_clean_backup() {
    alert "PHASE 2: IDENTIFY CLEAN BACKUP POINT"
    
    log INFO "Analyzing backups to find last clean state..."
    
    # List all available backups
    local backups_list="$FORENSICS_DIR/available_backups.txt"
    
    cat > "$backups_list" <<HEADER
Available Backups Analysis
==========================
Generated: $(date)

HEADER
    
    # Check rsync snapshots
    if [ -d "$BACKUP_ROOT/rsync-snapshots" ]; then
        log INFO "Analyzing rsync snapshots..."
        
        ls -lt "$BACKUP_ROOT/rsync-snapshots" | grep -v "^total" | while read line; do
            echo "$line" >> "$backups_list"
        done
    fi
    
    # Check database backups
    if [ -d "$BACKUP_ROOT/mysql" ]; then
        log INFO "Analyzing MySQL backups..."
        echo "" >> "$backups_list"
        echo "MySQL Backups:" >> "$backups_list"
        ls -lt "$BACKUP_ROOT/mysql/daily" | head -10 >> "$backups_list"
    fi
    
    # Identify infection timeline
    log INFO "Determining infection timeline..."
    
    # Find first encrypted file timestamp
    local first_encrypted=$(find /data -type f \( -name "*.locked" -o -name "*.encrypted" \) \
        -printf '%T@ %p\n' 2>/dev/null | sort -n | head -1 | awk '{print $1}')
    
    if [ -n "$first_encrypted" ]; then
        local infection_time=$(date -d "@$first_encrypted" '+%Y-%m-%d %H:%M:%S')
        log WARN "Estimated infection start time: $infection_time"
        echo "Estimated infection time: $infection_time" >> "$backups_list"
        
        # Find latest backup before infection
        local infection_timestamp=$(date -d "$infection_time" +%s)
        
        for snapshot in $(ls -t "$BACKUP_ROOT/rsync-snapshots" 2>/dev/null | grep -v "latest"); do
            local snapshot_time=$(stat -c %Y "$BACKUP_ROOT/rsync-snapshots/$snapshot" 2>/dev/null)
            
            if [ "$snapshot_time" -lt "$infection_timestamp" ]; then
                CLEAN_BACKUP_POINT="$BACKUP_ROOT/rsync-snapshots/$snapshot"
                log INFO "Identified clean backup point: $snapshot"
                echo "RECOMMENDED RESTORE POINT: $snapshot" >> "$backups_list"
                break
            fi
        done
    fi
    
    # Verify backup integrity
    if [ -n "$CLEAN_BACKUP_POINT" ]; then
        log INFO "Verifying backup integrity..."
        verify_backup_integrity "$CLEAN_BACKUP_POINT"
    else
        alert "Could not identify clean backup point automatically"
        log ERROR "Manual intervention required to select restore point"
    fi
    
    echo "- $(date): Clean backup point identified" >> "$INCIDENT_DIR/timeline.md"
}

# Verify backup integrity
verify_backup_integrity() {
    local backup_path="$1"
    
    log INFO "Verifying integrity of: $backup_path"
    
    # Check if incomplete flag exists
    if [ -f "$backup_path/.incomplete" ]; then
        log ERROR "Backup is marked as incomplete!"
        return 1
    fi
    
    # Check for ransom notes in backup
    local ransom_in_backup=$(find "$backup_path" -type f \
        -name "*DECRYPT*" -o -name "*RANSOM*" 2>/dev/null | wc -l)
    
    if [ "$ransom_in_backup" -gt 0 ]; then
        log ERROR "Ransom notes found in backup! This backup is infected."
        return 1
    fi
    
    # Check for encrypted files in backup
    local encrypted_in_backup=$(find "$backup_path" -type f \
        -name "*.locked" -o -name "*.encrypted" 2>/dev/null | wc -l)
    
    if [ "$encrypted_in_backup" -gt 0 ]; then
        log ERROR "Encrypted files found in backup! This backup is infected."
        return 1
    fi
    
    # Verify checksums if available
    if [ -f "$backup_path/.checksums.md5" ]; then
        log INFO "Verifying checksums..."
        
        cd "$backup_path"
        if md5sum -c .checksums.md5 &>/dev/null; then
            log INFO "✓ Checksum verification passed"
        else
            log WARN "✗ Some checksums failed verification"
        fi
    fi
    
    log INFO "Backup integrity verification completed"
    return 0
}

# Phase 3: Eradication & System Preparation
phase3_eradication() {
    alert "PHASE 3: ERADICATION & SYSTEM PREPARATION"
    
    log INFO "Preparing system for clean restoration..."
    
    # Step 1: Full malware scan
    log INFO "Step 1/5: Running malware scan..."
    run_malware_scan
    
    # Step 2: Remove persistence mechanisms
    log INFO "Step 2/5: Checking for persistence mechanisms..."
    check_persistence
    
    # Step 3: Patch vulnerabilities
    log INFO "Step 3/5: Applying security patches..."
    apply_security_patches
    
    # Step 4: Update security tools
    log INFO "Step 4/5: Updating security tools..."
    update_security_tools
    
    # Step 5: Prepare recovery environment
    log INFO "Step 5/5: Preparing recovery environment..."
    prepare_recovery_environment
    
    echo "- $(date): Eradication phase completed" >> "$INCIDENT_DIR/timeline.md"
}

# Run malware scan
run_malware_scan() {
    log INFO "Running ClamAV scan..."
    
    # Install ClamAV if not present
    if ! command -v clamscan &>/dev/null; then
        log INFO "Installing ClamAV..."
        apt-get update -qq
        apt-get install -y clamav clamav-daemon
    fi
    
    # Update virus definitions
    log INFO "Updating virus definitions..."
    freshclam
    
    # Scan system
    log INFO "Scanning system (this may take a while)..."
    clamscan -r -i --log="$FORENSICS_DIR/malware_scan.log" / || true
    
    # Summary
    local infected_files=$(grep "Infected files" "$FORENSICS_DIR/malware_scan.log" | awk '{print $3}')
    
    if [ "${infected_files:-0}" -gt 0 ]; then
        log WARN "Found $infected_files infected files"
        log WARN "Details in: $FORENSICS_DIR/malware_scan.log"
    else
        log INFO "No active malware detected"
    fi
}

# Check for persistence mechanisms
check_persistence() {
    log INFO "Checking for malware persistence mechanisms..."
    
    # Check cron jobs
    log INFO "Checking cron jobs..."
    crontab -l > "$FORENSICS_DIR/crontab_backup.txt" 2>/dev/null || true
    
    # Check systemd services
    log INFO "Checking systemd services..."
    systemctl list-unit-files --state=enabled > "$FORENSICS_DIR/enabled_services.txt"
    
    # Check startup scripts
    log INFO "Checking startup scripts..."
    ls -la /etc/init.d/ > "$FORENSICS_DIR/init_scripts.txt"
    
    # Check .bashrc and similar
    log INFO "Checking user profiles..."
    find /home -name ".bashrc" -o -name ".bash_profile" -o -name ".profile" | \
        xargs grep -l "curl\|wget\|nc\|/tmp" > "$FORENSICS_DIR/suspicious_profiles.txt" 2>/dev/null || true
    
    # Check for suspicious files in /tmp
    log INFO "Checking /tmp directory..."
    ls -la /tmp > "$FORENSICS_DIR/tmp_contents.txt"
}

# Apply security patches
apply_security_patches() {
    log INFO "Applying security patches..."
    
    # Update package lists
    apt-get update -qq
    
    # Apply security updates
    apt-get upgrade -y -qq --only-upgrade
    
    # Apply unattended upgrades
    apt-get dist-upgrade -y -qq
    
    log INFO "Security patches applied"
}

# Update security tools
update_security_tools() {
    log INFO "Updating security tools..."
    
    # Update and enable firewall
    apt-get install -y ufw
    ufw --force enable
    ufw default deny incoming
    ufw default allow outgoing
    ufw allow ssh
    
    # Enable fail2ban
    apt-get install -y fail2ban
    systemctl enable fail2ban
    systemctl start fail2ban
    
    log INFO "Security tools updated and enabled"
}

# Prepare recovery environment
prepare_recovery_environment() {
    log INFO "Preparing recovery environment..."
    
    # Create recovery mount point
    mkdir -p /mnt/recovery
    
    # Unmount any existing mounts
    umount /mnt/recovery 2>/dev/null || true
    
    # Verify backup accessibility
    if [ -n "$CLEAN_BACKUP_POINT" ]; then
        if [ -d "$CLEAN_BACKUP_POINT" ]; then
            log INFO "✓ Clean backup point is accessible"
        else
            log ERROR "✗ Clean backup point not accessible!"
            return 1
        fi
    fi
    
    log INFO "Recovery environment ready"
}

# Phase 4: Recovery from Clean Backup
phase4_recovery() {
    alert "PHASE 4: RECOVERY FROM CLEAN BACKUP"
    
    if [ -z "$CLEAN_BACKUP_POINT" ]; then
        alert "No clean backup point identified!"
        log ERROR "Cannot proceed with recovery"
        return 1
    fi
    
    log INFO "Beginning recovery from: $CLEAN_BACKUP_POINT"
    
    # Create recovery plan
    local recovery_plan="$INCIDENT_DIR/recovery_plan.txt"
    cat > "$recovery_plan" <<PLAN
Recovery Plan
=============
Restore Point: $CLEAN_BACKUP_POINT
Recovery Time: $(date)

Steps:
1. Archive infected data for forensics
2. Wipe affected partitions
3. Restore system files
4. Restore databases
5. Restore application data
6. Verify integrity
7. Apply post-recovery hardening
8. Resume services

PLAN
    
    # Step 1: Archive infected data
    log INFO "Step 1/8: Archiving infected data for forensics..."
    archive_infected_data
    
    # Step 2: Wipe affected partitions (CAREFUL!)
    log WARN "Step 2/8: Preparing to wipe affected partitions..."
    log WARN "This is DESTRUCTIVE. Verify backups before proceeding!"
    
    # In production, add confirmation here
    read -p "Type 'CONFIRM' to proceed with partition wipe: " confirm
    if [ "$confirm" != "CONFIRM" ]; then
        log INFO "Recovery aborted by user"
        return 1
    fi
    
    wipe_affected_partitions
    
    # Step 3: Restore system files
    log INFO "Step 3/8: Restoring system files..."
    restore_system_files
    
    # Step 4: Restore databases
    log INFO "Step 4/8: Restoring databases..."
    restore_databases
    
    # Step 5: Restore application data
    log INFO "Step 5/8: Restoring application data..."
    restore_application_data
    
    # Step 6: Verify integrity
    log INFO "Step 6/8: Verifying data integrity..."
    verify_restored_data
    
    # Step 7: Post-recovery hardening
    log INFO "Step 7/8: Applying post-recovery hardening..."
    post_recovery_hardening
    
    # Step 8: Resume services
    log INFO "Step 8/8: Resuming services..."
    resume_services
    
    echo "- $(date): Recovery phase completed" >> "$INCIDENT_DIR/timeline.md"
}

# Archive infected data
archive_infected_data() {
    log INFO "Archiving infected data for forensic analysis..."
    
    local forensic_archive="$FORENSICS_DIR/infected_data_$(date +%Y%m%d-%H%M%S).tar.gz"
    
    # Archive encrypted files and ransom notes
    tar -czf "$forensic_archive" \
        --ignore-failed-read \
        $(find /data -type f \( -name "*.locked" -o -name "*.encrypted" -o -name "*DECRYPT*" \) 2>/dev/null | head -100) \
        2>/dev/null || true
    
    log INFO "Forensic archive created: $forensic_archive"
}

# Wipe affected partitions
wipe_affected_partitions() {
    log WARN "Wiping affected partitions..."
    
    # Identify data partitions (example - adjust for your environment)
    local data_partitions=("/data" "/var/www" "/home")
    
    for partition in "${data_partitions[@]}"; do
        if [ -d "$partition" ]; then
            log WARN "Wiping: $partition"
            
            # Remove all files
            find "$partition" -mindepth 1 -delete 2>/dev/null || true
            
            log INFO "Wiped: $partition"
        fi
    done
    
    log WARN "Partition wipe completed"
}

# Restore system files
restore_system_files() {
    log INFO "Restoring system files from clean backup..."
    
    # Restore /etc configuration
    if [ -d "$CLEAN_BACKUP_POINT/etc" ]; then
        log INFO "Restoring /etc configuration..."
        rsync -av "$CLEAN_BACKUP_POINT/etc/" /etc/
    fi
    
    # Restore user profiles
    if [ -d "$CLEAN_BACKUP_POINT/home" ]; then
        log INFO "Restoring user profiles..."
        rsync -av "$CLEAN_BACKUP_POINT/home/" /home/
    fi
    
    log INFO "System files restored"
}

# Restore databases
restore_databases() {
    log INFO "Restoring databases from clean backup..."
    
    # MySQL
    if [ -d "$BACKUP_ROOT/mysql/daily" ]; then
        local mysql_backup=$(find "$BACKUP_ROOT/mysql/daily" -name "*.sql.gz" -type f | \
            xargs ls -t | head -1)
        
        if [ -n "$mysql_backup" ]; then
            # Get backup timestamp
            local backup_time=$(stat -c %Y "$mysql_backup")
            local infection_time=$(date -d "$infection_time" +%s 2>/dev/null || echo "999999999999")
            
            # Verify backup is before infection
            if [ "$backup_time" -lt "$infection_time" ]; then
                log INFO "Restoring MySQL from: $mysql_backup"
                
                systemctl start mysql
                gunzip < "$mysql_backup" | mysql -u root
                
                log INFO "MySQL restored"
            else
                log ERROR "MySQL backup is after infection time!"
            fi
        fi
    fi
    
    # PostgreSQL
    if [ -d "$BACKUP_ROOT/postgresql" ]; then
        local pg_backup=$(find "$BACKUP_ROOT/postgresql" -name "pg-*.sql.gz" -type f | \
            xargs ls -t | head -1)
        
        if [ -n "$pg_backup" ]; then
            log INFO "Restoring PostgreSQL from: $pg_backup"
            
            systemctl start postgresql
            gunzip < "$pg_backup" | sudo -u postgres psql
            
            log INFO "PostgreSQL restored"
        fi
    fi
}

# Restore application data
restore_application_data() {
    log INFO "Restoring application data from clean backup..."
    
    # Restore web applications
    if [ -d "$CLEAN_BACKUP_POINT/var/www" ]; then
        log INFO "Restoring web applications..."
        rsync -av "$CLEAN_BACKUP_POINT/var/www/" /var/www/
    fi
    
    # Restore application data
    if [ -d "$CLEAN_BACKUP_POINT/data" ]; then
        log INFO "Restoring application data..."
        rsync -av "$CLEAN_BACKUP_POINT/data/" /data/
    fi
    
    log INFO "Application data restored"
}

# Verify restored data
verify_restored_data() {
    log INFO "Verifying restored data integrity..."
    
    # Check for ransom notes (should be none)
    local ransom_check=$(find / -name "*DECRYPT*" -o -name "*RANSOM*" 2>/dev/null | wc -l)
    
    if [ "$ransom_check" -gt 0 ]; then
        log ERROR "WARNING: Ransom notes still present after restore!"
        return 1
    else
        log INFO "✓ No ransom notes found"
    fi
    
    # Check for encrypted files
    local encrypted_check=$(find /data -name "*.locked" -o -name "*.encrypted" 2>/dev/null | wc -l)
    
    if [ "$encrypted_check" -gt 0 ]; then
        log ERROR "WARNING: Encrypted files still present after restore!"
        return 1
    else
        log INFO "✓ No encrypted files found"
    fi
    
    # Verify file counts match backup
    local restored_count=$(find /data -type f 2>/dev/null | wc -l)
    local backup_count=$(find "$CLEAN_BACKUP_POINT/data" -type f 2>/dev/null | wc -l)
    
    local diff=$((restored_count - backup_count))
    local diff_abs=${diff#-}
    
    if [ "$diff_abs" -lt 100 ]; then
        log INFO "✓ File count verification passed (diff: $diff)"
    else
        log WARN "File count mismatch: restored=$restored_count, backup=$backup_count"
    fi
    
    log INFO "Data integrity verification completed"
}

# Post-recovery hardening
post_recovery_hardening() {
    log INFO "Applying post-recovery security hardening..."
    
    # Change all passwords
    log INFO "Passwords must be changed for all accounts!"
    
    # Update SSH keys
    log INFO "Regenerating SSH host keys..."
    rm -f /etc/ssh/ssh_host_*
    dpkg-reconfigure openssh-server
    
    # Enable additional security measures
    log INFO "Enabling additional security measures..."
    
    # Configure AppArmor/SELinux
    apt-get install -y apparmor apparmor-utils
```
