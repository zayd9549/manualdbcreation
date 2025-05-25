# 🛠️ **Manual Oracle 19c Database Creation via Command Line**

---

### 🧾 **1. Set Environment and Create PFILE**

```bash
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
cd $ORACLE_HOME/dbs
vi initORCL.ora
```

Inside `initORCL.ora`:

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

💡 **Explanation**:
This step sets up the Oracle environment and defines a basic PFILE (parameter file), which contains the minimal settings required to start the database in NOMOUNT mode.

---

### 📂 **2. Create Necessary Directories**

```bash
mkdir -p /u01/app/oracle/admin/ORCL/adump
mkdir -p /u01/oradata/ORCL
mkdir -p /home/oracle/dbscripts
```

💡 **Explanation**:
Oracle needs directories to store audit files, control/data files, and scripts. These paths must exist before database creation to avoid errors.

---

### 🚦 **3. Set ORACLE\_SID and Start NOMOUNT**

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

💡 **Explanation**:
The database instance is started in NOMOUNT mode, which reads the PFILE and initializes memory (SGA). It is the prerequisite state for creating a database.

---

### 🏗️ **4. Create the Database**

```bash
vi /home/oracle/dbscripts/dbcreate.sql
```

Insert:

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

Run the script:

```bash
sqlplus / as sysdba
@/home/oracle/dbscripts/dbcreate.sql
```

💡 **Explanation**:
The `CREATE DATABASE` command physically creates the Oracle database with essential files, user accounts, character sets, and tablespaces.

---

### 🛠️ **5. Run Required Post-Create Scripts**

```sql
@?/rdbms/admin/catalog.sql
@?/rdbms/admin/catproc.sql
EXIT;
```

💡 **Explanation**:
These scripts are critical for building the data dictionary (`catalog.sql`) and installing PL/SQL packages (`catproc.sql`), which support Oracle's core functionality.

---

### 👤 **6. Build Product User Profile Table**

```bash
sqlplus system/Oracle#123
@?/sqlplus/admin/pupbld.sql
EXIT;
```

💡 **Explanation**:
`pupbld.sql` sets up user-specific SQL*Plus settings and restrictions by creating the PRODUCT\_PROFILE table used by SQL*Plus.

---

### 🧹 **7. Compile Invalid Objects**

```bash
sqlplus / as sysdba
@?/rdbms/admin/utlrp.sql
```

💡 **Explanation**:
This script recompiles invalid PL/SQL and Java objects in the database to ensure system and application components function properly.

---

### 🔍 **8. Check for Invalid Objects**

```sql
SELECT COUNT(*) FROM dba_objects WHERE status='INVALID';
```

💡 **Explanation**:
Verifies the number of invalid objects remaining after recompilation. A clean database should ideally have zero invalid objects.

---

### 🔁 **9. Convert PFILE to SPFILE**

```sql
CREATE SPFILE FROM PFILE;
SHUTDOWN IMMEDIATE;
STARTUP;
EXIT;
```

💡 **Explanation**:
Converting to SPFILE enables dynamic parameter management and is required for features like automatic memory tuning.

---

### 📘 **10. Add Entry to `/etc/oratab`**

```bash
vi /etc/oratab
```

Add this line:

```ini
ORCL:/u01/app/oracle/product/19.0.0/dbhome_1:N
```

📝 Save and exit:

```
:wq!
```

💡 **Explanation**:
The `/etc/oratab` file helps automate database startup/shutdown using `dbstart` and `dbshut`. This entry is necessary for Oracle utilities to detect the instance.

---
