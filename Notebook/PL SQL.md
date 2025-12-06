Limitations of SQL :  
Performs one operations at a time.  
Lacks the feature of grouping similar instructions together.  

Advantages of PL/SQL :  
Enables modularizations, provides better security, enables maintainability, exception handling  

Procedural Language Extension to SQL = PLSQL  

Procedural constructs ⇒ Loops, if else, variables, data types. ⇒ Programming constructs in SQL  
Also supports OOPs. ⇒ Composite Data Types.(Refer Extra)  

PL/SQL Block Structure:  
DECLARE(optional) ⇒ BEGIN ⇒ EXCEPTION(optional) ⇒ END ⇒ /(indicates termination of block)

  
VARCHAR2⇒ oracle standard of varchar.

  

DIsplaying output in PL/SQL:

  

```SQL
SET SERVEROUTPUT ON
BEGIN
DBMS_OUTPUT.PUT_LINE(’First name of employee is ‘||var_fname);
END;
```

  

Variable Declarations:

must start with character, must not be longer than 30 characters.  
var_name [CONSTANT] datatype [NOT_NULL] [ := or DEFAULT expression];

```SQL
SET SERVEROUTPUT ON
DECLARE
	v_sal NUMBER(8,2);--total 8 out of which 2 are decimals.
	v_s1 CONSTANT VARCHAR2(20) NOT_NULL := 'HK';
	v_s2 VARCHAR2(20) DEFAULT 'HK';
BEGIN
//Inputing
SELECT f_name INTO v_s2 from employees where id = 100;
DBMS_OUTPUT.PUT_LINE(’First name of employee is ‘||var_fname);
END;
```

Input:

```SQL
SELECT f_name INTO v_s2 from employees where id = 100;
```

Data types String:

CHAR, NCHAR, VARCHAR, NVARCHAR. CLOB, NCLOB(For Large files )  
** For using apostrope in string:::  
v_event = q’[Father’s]’;

v_event = q’!Mother’s!’;

  

Integer types:

NUMBER  
SIMPLE_INTEGER  
BINARY_INTEGER(PLS_INTEGER)

  

%TYPE ⇒ copies the datatype

```SQL
v_nm employees.fname%TYPE
v_xy v_nm%TYPE
```

BOOLEAN ⇒ (True, False or NULL)

LOB(Large Objects) ⇒  
Contains data upto 4GBs of Data  
Char,Bin LOBS, Bin File(BFILE)

Composite DateTypes: ⇒  

Bind Variables :

```SQL
VARIABLE b_result NUMBER --this is a bind variable
-- it has a global scope. can be used in multiple blocks
-- it has to be used with a : preceeding,
:b_result := 10
-- can be printed using print command
PRINT b_result
```

SET AUTOPRINT ON  
used for debugging.  

Predefined functions like MAX, MIN,LNGTH<MONTHS_BETWEEN etc can be used.

  

SEQUENCES :  
these are auto-incremented values in tables.  
CREATE SEQUENCE emp INCREMENT BY 1 NOMAXVALUE;  

v_id = emp.NEXTVAL;

Only DML,TCL commands are generally used in PL/SQL.

```SQL
IF cond1 THEN 
ELSIF cond2 THEN
ELSE
END IF;

CASE v_1
	WHEN 20 THEN
		
	WHEN 30 THEN

	ELSE

END CASE;

v3 := CASE v_1
	WHEN 20 THEN 123
	WHEN 30 THEN 456
	ELSE 789
END CASE;
```

Handlling Nulls:

comparision with null yields null, Not of a null is also null.

  

```SQL
LOOP
	statements;
	EXIT WHEN exit_condition;
END LOOP;

WHILE condition LOOP
	statements;
END LOOP;

FOR counter IN [REVERSE] lower_bound..upper_bound LOOP
	statements;
END LOOP;
```

### PROCEDURES AND FUNCTIONS

```SQL
/*
Sub Programs ==> Functions of PL SQL ==> Named Block
*/

```