# **🛠️ Manual Database Creation in Oracle 19c**

Manual database creation in Oracle 19c is performed using **command-line tools**. Follow the steps below to create a database without using DBCA. Each step includes detailed explanations for better understanding.

---

## **📋 Steps to Manually Create an Oracle 19c Database**

---

### **1️⃣ Set Environment Variables & Create PFILE at `$ORACLE_HOME/dbs`**

```bash
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
cd $ORACLE_HOME/dbs
vi initORCL.ora
```

Sample `initORCL.ora`:

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

> 📘 **Explanation:**
> The PFILE (Parameter File) is a plain-text initialization file Oracle uses to configure the instance.
> 💡 Place it under `$ORACLE_HOME/dbs` as `initORCL.ora`.

---

### **2️⃣ Create Necessary Directories**

```bash
mkdir -p /u01/app/oracle/admin/ORCL/adump
mkdir -p /u01/oradata/ORCL
mkdir -p /home/oracle/dbscripts
```

> 📁 These directories are required for audit logs, datafiles, and script management.

---

### **3️⃣ Set `ORACLE_SID` and Start Instance in NOMOUNT Mode**

```bash
export ORACLE_SID=ORCL
sqlplus / as sysdba
```

Inside SQL\*Plus:

```sql
STARTUP NOMOUNT;
SELECT INSTANCE_NAME, STATUS FROM V$INSTANCE;
EXIT;
```

> ⚙️ `STARTUP NOMOUNT` starts the Oracle instance without mounting the control files.
> Use `EXIT;` to return to Linux prompt.

---

### **4️⃣ Create the Database using SQL Script**

Create script:

```bash
vi /home/oracle/dbscripts/dbcreate.sql
```

Paste the following content:

```sql
CREATE DATABASE ORCL
USER SYS IDENTIFIED BY Oracle#123
USER SYSTEM IDENTIFIED BY Oracle#123
LOGFILE
  GROUP 1 '/u01/oradata/ORCL/redo1.log' SIZE 200M,
  GROUP 2 '/u01/oradata/ORCL/redo2.log' SIZE 200M,
  GROUP 3 '/u01/oradata/ORCL/redo3.log' SIZE 200M
MAXLOGFILES 10
MAXLOGMEMBERS 5
MAXDATAFILES 200
MAXINSTANCES 1
CHARACTER SET AL32UTF8
NATIONAL CHARACTER SET AL16UTF16
DATAFILE '/u01/oradata/ORCL/system01.dbf' SIZE 1G REUSE AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED
SYSAUX DATAFILE '/u01/oradata/ORCL/sysaux01.dbf' SIZE 600M REUSE AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED
UNDO TABLESPACE UNDOTBS1 DATAFILE '/u01/oradata/ORCL/undotbs01.dbf' SIZE 200M AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED
DEFAULT TEMPORARY TABLESPACE temp1 TEMPFILE '/u01/oradata/ORCL/temp01.dbf' SIZE 100M REUSE AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED
DEFAULT TABLESPACE USERS DATAFILE '/u01/oradata/ORCL/users01.dbf' SIZE 200M AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED;
```

Then run:

```bash
sqlplus / as sysdba
```

```sql
@/home/oracle/dbscripts/dbcreate.sql
EXIT;
```

> 🛠️ This script creates your Oracle database structure, including log files and tablespaces.

---

### **5️⃣ Run Oracle Catalog & Catproc Scripts**

```bash
sqlplus / as sysdba
```

```sql
@?/rdbms/admin/catalog.sql
@?/rdbms/admin/catproc.sql
EXIT;
```

> 📚 These scripts set up essential system views and PL/SQL packages.

---

### **6️⃣ Build Product User Profile Table**

```bash
sqlplus system/Oracle#123
```

```sql
@?/sqlplus/admin/pupbld.sql
EXIT;
```

> 🔒 Creates the `PRODUCT_USER_PROFILE` table used by SQL\*Plus for session restrictions.

---

### **7️⃣ Compile Invalid Objects**

```bash
sqlplus / as sysdba
```

```sql
@?/rdbms/admin/utlrp.sql
EXIT;
```

> 🧹 This recompiles invalid PL/SQL objects to ensure DB health.

---

### **8️⃣ Check for Invalid Objects**

```bash
sqlplus / as sysdba
```

```sql
SELECT COUNT(*) FROM dba_objects WHERE status='INVALID';
EXIT;
```

> ✅ Zero invalid objects means everything compiled successfully.

---

### **9️⃣ Add ORCL Entry to `/etc/oratab`**

```bash
vi /etc/oratab
```

Add this line:

```ini
ORCL:/u01/app/oracle/product/19.0.0/dbhome_1:N
```

> 🗂️ This file is used by `oraenv`, `dbstart`, and `dbshut`.

To use `oraenv`:

```bash
. oraenv <<< ORCL
```

Or create a custom env file:

```bash
vi /home/oracle/setdb_ORCL.env
```

```bash
export ORACLE_SID=ORCL
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

Activate:

```bash
. /home/oracle/setdb_ORCL.env
```

Or add it to `.bash_profile`:

```bash
vi ~/.bash_profile
```

Append:

```bash
export ORACLE_SID=ORCL
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

Then:

```bash
. ~/.bash_profile
```

> 🧠 Ensures Oracle environment is loaded for every new session.

---

### 🔁 **Convert PFILE to SPFILE**

```bash
sqlplus / as sysdba
```

```sql
CREATE SPFILE FROM PFILE;
SHUTDOWN IMMEDIATE;
STARTUP;
EXIT;
```

> 💾 SPFILE is binary and preferred for persistent configurations.

---

### 🔍 **Quick Recap of Key Parameters**

| 🧩 Parameter             | 🔎 Description                                  |
| ------------------------ | ----------------------------------------------- |
| `audit_file_dest`        | Directory for audit trail logs                  |
| `compatible`             | Ensures compatibility with Oracle version       |
| `control_files`          | Paths to control files                          |
| `db_block_size`          | Data block size in bytes (usually 8K)           |
| `db_name`                | Name of your database                           |
| `diagnostic_dest`        | Root ADR directory for diagnostics              |
| `undo_tablespace`        | Tablespace for managing undo data               |
| `sga_target`             | Total memory allocated for SGA                  |
| `MAXLOGFILES`            | Maximum redo log groups                         |
| `MAXLOGMEMBERS`          | Max members (copies) per redo log group         |
| `MAXDATAFILES`           | Max number of datafiles                         |
| `MAXINSTANCES`           | Max RAC instances (1 for single instance)       |
| `CHARACTER SET`          | Default encoding (AL32UTF8 is standard)         |
| `NATIONAL CHARACTER SET` | Encoding for NCHAR/NVARCHAR (usually AL16UTF16) |

---
