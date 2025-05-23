# 🛠️ **Manual Database Creation in Oracle 19c**

Manual database creation in Oracle 19c is done via command-line tools. This guide walks you through the process step by step.

---

### 🔧 **1. Set Environment Variables and Create PFILE at `$ORACLE_HOME/dbs`**

```bash
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
cd $ORACLE_HOME/dbs
vi initORCL.ora
```

➡️ Type the following inside `vi`:

```ini
*.audit_file_dest='/u01/app/oracle/admin/ORCL/adump'
*.compatible='19.0.0'
*.control_files='/u01/oradata/ORCL/control01.ctl','/u01/oradata/ORCL/control02.ctl'
*.db_block_size=8192
*.db_name='ORCL'
*.diagnostic_dest='/u01/app/oracle'
*.undo_tablespace='UNDOTBS1'
*.sga_target=1g
```

📝 Save and exit:

```
:wq!
```

---

### 📁 **2. Create Necessary Directories**

```bash
mkdir -p /u01/app/oracle/admin/ORCL/adump
mkdir -p /u01/oradata/ORCL
mkdir -p /home/oracle/dbscripts
```

---

### 🏁 **3. Set ORACLE\_SID and Start the Instance in NOMOUNT**

```bash
export ORACLE_SID=ORCL
sqlplus / as sysdba
```

Inside `SQL*Plus`:

```sql
STARTUP NOMOUNT;
SELECT INSTANCE_NAME, STATUS FROM V$INSTANCE;
EXIT;
```

---

### 🏗️ **4. Create the Database (dbcreate.sql)**

```bash
vi /home/oracle/dbscripts/dbcreate.sql
```

➡️ Type the following inside `vi`:

```sql
CREATE DATABASE ORCL
USER SYS IDENTIFIED BY Oracle#123
USER SYSTEM IDENTIFIED BY Oracle#123
LOGFILE
  GROUP 1 '/u01/oradata/ORCL/redo1.log' SIZE 200M REUSE,
  GROUP 2 '/u01/oradata/ORCL/redo2.log' SIZE 200M REUSE,
  GROUP 3 '/u01/oradata/ORCL/redo3.log' SIZE 200M REUSE
MAXLOGFILES 10
MAXLOGMEMBERS 5
MAXDATAFILES 200
MAXINSTANCES 1
CHARACTER SET AL32UTF8
NATIONAL CHARACTER SET AL16UTF16
DATAFILE '/u01/oradata/ORCL/system01.dbf' SIZE 1G REUSE AUTOEXTEND ON
SYSAUX DATAFILE '/u01/oradata/ORCL/sysaux01.dbf' SIZE 600M REUSE AUTOEXTEND ON
UNDO TABLESPACE UNDOTBS1 DATAFILE '/u01/oradata/ORCL/undotbs01.dbf' SIZE 200M REUSE AUTOEXTEND ON
DEFAULT TEMPORARY TABLESPACE temp1 TEMPFILE '/u01/oradata/ORCL/temp01.dbf' SIZE 100M REUSE AUTOEXTEND ON
DEFAULT TABLESPACE USERS DATAFILE '/u01/oradata/ORCL/users01.dbf' SIZE 200M REUSE AUTOEXTEND ON;
```

📝 Save and exit:

```
:wq!
```

Run it:

```bash
sqlplus / as sysdba
```

```sql
@/home/oracle/dbscripts/dbcreate.sql
```

---

### 🧱 **5. Run Post-Creation Scripts**

```sql
@?/rdbms/admin/catalog.sql
@?/rdbms/admin/catproc.sql
EXIT;
```

---

### 🧩 **6. Build Product User Profile Table**

```bash
sqlplus system/Oracle#123
```

```sql
@?/sqlplus/admin/pupbld.sql
EXIT;
```

---

### 🛠️ **7. Compile Invalid Objects**

```bash
sqlplus / as sysdba
```

```sql
@?/rdbms/admin/utlrp.sql
```

---

### 🔎 **8. Check for Invalid Objects**

```sql
SELECT COUNT(*) FROM dba_objects WHERE status='INVALID';
EXIT;
```
---
### 🔄 **9. Convert PFILE to SPFILE**


```sql
CREATE SPFILE FROM PFILE;
SHUTDOWN IMMEDIATE;
STARTUP;
EXIT;
```
---

### 📝 **10. Add Entry to `/etc/oratab`**

```bash
vi /etc/oratab
```

➡️ Type the following:

```ini
ORCL:/u01/app/oracle/product/19.0.0/dbhome_1:N
```

📝 Save and exit:

```
:wq!
```
---

