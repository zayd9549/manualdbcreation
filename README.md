# **Manual Database Creation in Oracle 19c**

Manual database creation in Oracle 19c is performed using **command-line tools**. Follow the steps below for creating a database without using DBCA. Each step includes a detailed explanation for clarity and understanding.

---

## **Steps to Manually Create an Oracle 19c Database**

---

### **1. Create PFILE at `$ORACLE_HOME/dbs`**

```bash
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
```

> 📘 **Explanation:** The PFILE (Parameter File) is a plain text initialization file used by Oracle to configure the instance. We place it in the `$ORACLE_HOME/dbs` directory with a unique name like `initORCL.ora` (where `ORCL` is your DB SID).
>
> * `audit_file_dest`: Location for audit trail logs
> * `compatible`: Database compatibility level
> * `control_files`: Locations of control files
> * `db_block_size`: Default block size for the database
> * `db_name`: Name of the database
> * `diagnostic_dest`: Root location for all diagnostic files
> * `undo_tablespace`: Tablespace used for rollback operations

> ✅ **Tip:** After the database is created, it's recommended to convert this PFILE into a binary SPFILE.

---

### **2. Create Necessary Directories**

```bash
mkdir -p /u01/app/oracle/admin/ORCL/adump
mkdir -p /u01/oradata/ORCL
mkdir -p /home/oracle/dbscripts
```

> 📘 **Explanation:**
>
> * `admin/ORCL/adump`: Directory to store audit logs.
> * `oradata/ORCL`: Main location for database datafiles, control files, redo logs.
> * `dbscripts`: Directory for keeping SQL scripts such as creation scripts for DB, tablespaces, etc.

---

### **3. Set ORACLE\_SID and Start the Instance in NOMOUNT**

```bash
export ORACLE_SID=ORCL
sqlplus / as sysdba

-- Inside SQL*Plus:
STARTUP NOMOUNT;
SELECT INSTANCE_NAME, STATUS FROM V$INSTANCE;
```

> 📘 **Explanation:**
>
> * `ORACLE_SID` defines the environment variable pointing to your instance.
> * `STARTUP NOMOUNT` brings the instance up without mounting the control files. This is necessary to run the `CREATE DATABASE` command.

---

### **4. Create the Database (dbcreate.sql)**

Create a file named `dbcreate.sql`:

```bash
vi /home/oracle/dbscripts/dbcreate.sql
```

Paste the following script:

```sql
CREATE DATABASE ORCL
USER SYS IDENTIFIED BY manager
USER SYSTEM IDENTIFIED BY manager
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

Then run it from SQL\*Plus:

```sql
@/home/oracle/dbscripts/dbcreate.sql
```

> 📘 **Explanation:** This script defines the structure of your database:
>
> * Redo logs: Required for recovery and logging.
> * Datafiles: Defines the physical storage of SYSTEM, SYSAUX, USERS, UNDO, TEMP tablespaces.
> * `CHARACTER SET`, `NATIONAL CHARACTER SET`: Unicode encoding settings.
> * `MAXLOGFILES`, `MAXDATAFILES`, etc.: Allow for future growth and scalability.

---

### **5. Run Post-Creation Scripts**

```sql
@/u01/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/catalog.sql
@/u01/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/catproc.sql
```

> 📘 **Explanation:**
>
> * `catalog.sql`: Creates data dictionary views.
> * `catproc.sql`: Installs Oracle supplied PL/SQL packages and procedures.

---

### **6. Build Product User Profile Table**

```sql
CONN system/manager
@/u01/app/oracle/product/19.0.0/dbhome_1/sqlplus/admin/pupbld.sql
```

> 📘 **Explanation:** `pupbld.sql` builds the product user profile table used by SQL\*Plus. It helps restrict access and control session attributes for users.

---

### **7. Compile Invalid Objects**

```sql
CONN / AS SYSDBA
@/u01/app/oracle/product/19.0.0/dbhome_1/rdbms/admin/utlrp.sql
```

> 📘 **Explanation:** This script recompiles all invalid PL/SQL packages, procedures, and views to ensure the database is fully functional.

---

### **8. Check for Invalid Objects**

```sql
SELECT COUNT(*) FROM dba_objects WHERE status='INVALID';
```

> 📘 **Explanation:** A zero count means everything is successfully compiled. Any remaining invalid objects might indicate issues needing resolution.

---

### **9. Add Entry to `/etc/oratab`**

```bash
vi /etc/oratab
```

Add:

```ini
ORCL:/u01/app/oracle/product/19.0.0/dbhome_1:N
```

> 📘 **Explanation:**
>
> * The `/etc/oratab` file is used by utilities like `dbstart`, `dbshut`, and `oraenv`.
> * `N` means don't automatically start DB on boot. Change to `Y` if desired.

To load environment via oraenv:


```bash
. oraenv

ORCL
```
Or

```bash
. oraenv <<< ORCL
```

Or create a custom script:

```bash
vi /home/oracle/setdb_ORCL.env
```

Contents:

```bash
export ORACLE_SID=ORCL
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```
Or

To activate:

```bash
. /home/oracle/setdb_ORCL.env
```

Or 

Instead of manually setting environment variables every time, you can add them to your `.bash_profile`:

```bash
vi ~/.bash_profile
```

Add the following lines:

```bash
export ORACLE_SID=ORCL
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

Then reload the profile:

```bash
. ~/.bash_profile
```

> 📘 **Explanation:**
>
> * This ensures that each new terminal session automatically loads the Oracle environment.
> * Especially useful for consistent login sessions and scripting tasks.

---

### ✅ **Convert PFILE to SPFILE**:

```sql
CREATE SPFILE FROM PFILE;
```

Restart DB to use the new SPFILE:

```sql
SHUTDOWN IMMEDIATE;
STARTUP;
```

> 📘 **Explanation:** SPFILE is a server parameter file—binary and persistent. It's recommended for production environments.

---

### 🔍 **Detailed Explanation of Key Parameters**

| Parameter                      | Description                                                                                       |
| ------------------------------ | ------------------------------------------------------------------------------------------------- |
| `audit_file_dest`              | Location for audit trail logs.                                                                    |
| `compatible`                   | Database compatibility level.                                                                     |
| `control_files`                | Locations of control files. Required to mount and open the database.                              |
| `db_block_size`                | Default block size of the database, typically 8KB.                                                |
| `db_name`                      | Database name (must match in control files).                                                      |
| `diagnostic_dest`              | Base directory for ADR (Automatic Diagnostic Repository).                                         |
| `undo_tablespace`              | Stores undo data for transactions. Required for consistent reads and rollbacks.                   |
| `MAXLOGFILES`                  | Max number of redo log groups allowed. Useful for future expansion; default is \~16, max is 255.  |
| `MAXLOGMEMBERS`                | Max number of members (copies) per redo log group. Provides redundancy—typical values are 2 or 3. |
| `MAXDATAFILES`                 | Max number of datafiles allowed in the DB. Controls scalability. Default is \~200.                |
| `MAXINSTANCES`                 | Maximum instances that can mount the DB (used for RAC). Keep as 1 for single-instance.            |
| `CHARACTER SET`                | Encoding for CHAR/VARCHAR fields. `AL32UTF8` is Oracle's recommended Unicode charset.             |
| `NATIONAL CHARACTER SET`       | For NCHAR/NVARCHAR fields. `AL16UTF16` supports multi-byte characters.                            |
| `DEFAULT TEMPORARY TABLESPACE` | Used for operations like joins, sorts, and temporary results.                                     |
| `DEFAULT TABLESPACE`           | Default storage for user-created objects. Keeps SYSTEM tablespace clean.                          |

---
