The following is a `SQL` package that allows for editing tables and features in the geodatabase from `SQL` statements.

```sql
/******************************************************************************
*    PACKAGE NAME : GDB_VERSION_UTILS
*    SYNOPSIS     : This package contains functions and stored procedures
*        for interacting with versioned geodatabase tables in SDE.
*    AUTHOR       : Kyle Baesler
*    AMENDMENTS   :
*    DATE        NAME                REASON
*
*****************************************************************************/
CREATE OR REPLACE PACKAGE GDB_VERSION_UTILS IS

  FUNCTION DELTA_TABLE(OWNERNAME IN VARCHAR2, TABLENAME IN VARCHAR2, PREFIX IN VARCHAR2)
    RETURN VARCHAR2;


END;

CREATE OR REPLACE PACKAGE BODY GDB_VERSION_UTILS AS

  /******************************************************************************
  *    FUNCTION NAME : DELTA_TABLE
  *    SYNOPSIS      : This function returns the registration id of the specified
  *         table and owner.
  *    AUTHOR        : Kyle Baesler
  *    AMENDMENTS    :
  *    DATE        NAME                REASON
  *
  *****************************************************************************/
  FUNCTION DELTA_TABLE(OWNERNAME IN VARCHAR2, TABLENAME IN VARCHAR2, PREFIX IN VARCHAR2)
    RETURN VARCHAR2
  IS

    id NUMBER := 0;

    BEGIN

      SELECT REGISTRATION_ID
      INTO id
      FROM SDE.TABLE_REGISTRY
      WHERE UPPER(TABLE_NAME) = UPPER(TABLENAME)
            AND UPPER(OWNER) = UPPER(OWNERNAME);

      RETURN PREFIX || id;

    END;

  /******************************************************************************
  *    PROCEDURE NAME: ANALYZE_TABLE
  *    SYNOPSIS      : This procedure runs statistics on the specified
  *             table as well as its associated Adds and Deletes tables.
  *    AUTHOR        : Kyle Baesler
  *    AMENDMENTS    :
  *    DATE        NAME                REASON
  *
  ******************************************************************************/
  PROCEDURE ANALYZE_TABLE
    (OWNERNAME IN VARCHAR2, TABLENAME IN VARCHAR2)
  AS

    BEGIN

      SYS.DBMS_STATS.GATHER_TABLE_STATS(OWNNAME=>OWNERNAME, TABNAME=>TABLENAME, CASCADE=>TRUE);
      SYS.DBMS_STATS.GATHER_TABLE_STATS(OWNNAME=>OWNERNAME,
                                        TABNAME=> GDB_VERSION_UTILS.DELTA_TABLE(OWNERNAME=>OWNERNAME, TABLENAME=>TABLENAME,
                                                                 PREFIX=>'A'),
                                        CASCADE=>TRUE);
      SYS.DBMS_STATS.GATHER_TABLE_STATS(OWNNAME=>OWNERNAME,
                                        TABNAME=> GDB_VERSION_UTILS.DELTA_TABLE(OWNERNAME=>OWNERNAME, TABLENAME=>TABLENAME,
                                                                 PREFIX=>'D'),
                                        CASCADE=>TRUE);

      EXCEPTION
      WHEN OTHERS THEN

      RAISE;

    END;

  /******************************************************************************
  *    PROCEDURE NAME: UPDATE_TABLE
  *    SYNOPSIS      : This procedure that will execute the update statement
  *                    against the business and A (or ADD) table.
  *    AUTHOR        : Kyle Baesler
  *    AMENDMENTS    :
  *    DATE        NAME                REASON
  *
  ******************************************************************************/
  PROCEDURE UPDATE_TABLE
    (OWNERNAME IN VARCHAR2, TABLENAME IN VARCHAR2, SETWHERE IN VARCHAR2)
  AS

    BEGIN

      EXECUTE IMMEDIATE 'UPDATE ' || OWNERNAME || '.' || TABLENAME || ' ' || SETWHERE;
      EXECUTE IMMEDIATE
      'UPDATE ' || OWNERNAME || '.' || GDB_VERSION_UTILS.DELTA_TABLE(OWNERNAME=>OWNERNAME, TABLENAME=>TABLENAME, PREFIX=>'A') ||
      ' ' || SETWHERE;

      EXCEPTION
      WHEN OTHERS THEN

      RAISE;

    END;

  /******************************************************************************
  *    PROCEDURE NAME: SET_VERSION
  *    SYNOPSIS      : This procedure that notify that the version should be
  *         set as the contents of the subsequent queries.
  *    AUTHOR        : Kyle Baesler
  *    AMENDMENTS    :
  *    DATE        NAME                REASON
  *
  ******************************************************************************/
  PROCEDURE SET_VERSION
    (VERSIONNAME IN VARCHAR2)
  AS

    BEGIN

      SDE.VERSION_UTIL.SET_CURRENT_VERSION(VERSIONNAME);

    END;

  /******************************************************************************
  *    PROCEDURE NAME: START_VERSION_EDITS
  *    SYNOPSIS      : This procedure that notify that the version should be
  *         set as the contents of the subsequent queries.
  *    AUTHOR        : Kyle Baesler
  *    AMENDMENTS    :
  *    DATE        NAME                REASON
  *
  ******************************************************************************/
  PROCEDURE START_VERSION_EDITS
    (VERSIONNAME IN VARCHAR2)
  AS

    BEGIN

      GDB_VERSION_UTILS.SET(VERSIONNAME=>VERSIONNAME);
      SDE.VERSION_USER_DDL.EDIT_VERSION(VERSIONNAME, 1);

    END;
  /******************************************************************************
  *    PROCEDURE NAME: STOP_VERSION_EDITS
  *    SYNOPSIS      : This procedure that notify that the version should be
  *         set as the contents of the subsequent queries.
  *    AUTHOR        : Kyle Baesler
  *    AMENDMENTS    :
  *    DATE        NAME                REASON
  *
  ******************************************************************************/
  PROCEDURE STOP_VERSION_EDITS
    (VERSIONNAME IN VARCHAR2)
  AS

    BEGIN

      SDE.VERSION_USER_DDL.EDIT_VERSION(VERSIONNAME, 2);

    END;
END;
/
```
