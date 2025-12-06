Business Object Spectra Service.

export BOSS_MODEL_HOME=/scratch/$USER/esg-prototype/boss  
export PATH=/scratch/$USER/rwddev/rwd_dev/boss-tools/bin:$PATH

cd /usr/local/packages/aime/install  
./run_as_root su root

boss metadata package -a esg

curl -vLk --noproxy "*" -u jack:password -X POST '[http://localhost:8000/api/rwdinfra/apm/v1/applications?force=Enabled](http://localhost:8000/api/rwdinfra/apm/v1/applications?force=Enabled)' -d @$BOSS_MODEL_HOME/config/application.json

  

![[Screenshot_2023-06-07_at_10.53.03_AM.png]]

- Modules ⇒ collection of related objects and artifacts. It has the following components.  
    Modules are created in /scratch/$USER/app-packages/pkgName
    
    `boss module create -m <moduleName> pkgName`
    
      
    
      
    

> [!important] Each package must contain a single module

> [!important] Before accessing the apis, you need to package and deploy.

  

- Business Objects ⇒ objects based on the tables of DB. These are described in JSON.

- Datatypes ⇒ shared between various BOs and modules.

- View ⇒ It is the data that is shown to end user.

- An OpenAPI catalog is generated which documents the module.

- implemented as a microservice architecture with docker containers deployed over K8 with helidon and GraalVM

  

  

Each module has

- api
    
    - rest-api.json5 ⇒ has versions of the api
    
    - version.json5 ⇒ has the list of BOs that are exposed
    

- BOs  
    A Business Object represents a table in the database.
    
    - views
    
    - bo.json5
    

- enums

- module.json5 ⇒ has metadata reg the module.

  

  

### Creating BOs

`boss bo create -bo <BusinessObjectName> -t <tablename> -m <moduleName> <app-package>`

Each BO has :

- views

- bo.json5

```JSON
{
  $dt_version : "2307.0.510",
  accessModifier : "public",//public or module => who can access it
	//public => api can access
  creatable : "public",
  updatable : "public",
  deletable : "public",
  size : "small",
  allowInheritance : false,
  fields : {
    departmentId : {
      type : "int32",//datatype
      accessModifier : "public",
      dataStoreMapping : {//where to get the data from
        rdbms : {
		      column : "DEPARTMENT_ID",//column name 
// You can also write SQL expressions -> "FirstName || ' ' || SecondName"
          sortable : true,
          searchable : true//for keys of table
        }
      },
      readable : "public",
      nullable : false,
      creatable : "module",
      updatable : "never",
      fieldSecurityEnabled : false
    },
  },
  dataStoreMapping : {//where to get data from
    rdbms : {
      DEPARTMENTS : {//DB name
        setHistoryColumns : "always",
        table : "DEPARTMENTS",//table name
	      join:""//for joins
			}
    }
  },
  identifiedBy : {
    $primary : [ "departmentId" ],//specify Ids
		altKey1 : ["f1"], ....
  },
	defaultIdentifier:"altkey1",//sets default key
	
}
```

- security.json5  
    Defines the security level of the object.

  

  

> [!important] Jaeger UI available at
> 
> [http://100.69.167.214:16686/](http://100.69.167.214:16686/search)

???labels???

  

## Defining Relationship between BOs

Relationship between BOs are when the column of another table is referenced in the current table.

Relation can be of three type:

- One to Many :
    
    - `boss bo addrelation -m departmentEmployees -bo Department -tbo Employee -fm departmentId:departmentId -c OneToMany -acc employees -macc department <app-package>`
    
    ```JSON
    //This field is in the DEPARTMENT BO.
    //The relation is always defined in the field the references the other table
    employees : {
      type : "object-collection",//custom data-type => Accessor Field
      accessModifier : "public",
      target : {
        module: "departmentEmployees",
        businessObject: "Employee"
      },
      mapping: {
        departmentId : "departmentId"//Identify the column that is referenced in the other table
      },
      mappedBy: "department"
    }
    
    //This field is in the EMPLOYEE BO. => Accessor Field
    department : {
      type: "object",//Each emoployee has just one Department.
    //Hence it is a many to one relation from employee side
      accessModifier: "public",
      target: { 
        module: "departmentEmployees",
        businessObject: "Department"
      },
      mappedBy: "employees"
    }
    ```
    

- One to One :
    
    - `boss bo addrelation -m studentInfo -bo Student -tbo StudentPII -fm id:studentId -c OneToOne -acc pii -j leftOuterJoin <app-package-name>`
    
    ```JSON
    pii : {//=> Accessor Field
      type : "object",//custom data type
      accessModifier : "public",
      target : {
        module : "studentInfo",
        businessObject : "StudentPII"
      },
      mapping : {
        id : "studentId"
      },
      joinType: "leftOuterJoin"
    }
    ```
    

- Many to Many :
    
    - This can be created by creating two Many one relations where a common third BO has the one end of the relation. This leads to Many-Many relation between the the other 2 BOs.
    
    `boss bo addrelation -m studentInfo -bo Student -tbo StudentContact -fm id:studentId -c OneToMany -acc students -macc contact <app-package-name>`
    
    `boss bo addrelation -m studentInfo -bo Contact -tbo StudentContact -fm id:contactId -c OneToMany -acc contacts -macc student <app-package-name>`
    
      
    
    ```JSON
    //Student
    students : {
      type : "object-collection",
      accessModifier : "public",
      target : {
        module : "studentInfo",
        businessObject : "StudentContact"
      },
      mappedBy : "contact"
    }
    //Contact
    contacts : {
      type : "object-collection",
      accessModifier : "public",
      target : {
        module : "studentInfo",
        businessObject : "StudentContact"
      },
      mappedBy : "student"
    }
    //Student Contact
    student : {
      type : "object",
      accessModifier : "public",
      target : {
        module : "studentInfo",
        businessObject : "Student"
      },
      mapping : {
        studentId : "id"
      },
      mappedBy : "students"
    },
    contact : {
      type : "object",
      accessModifier : "public",
      target : {
        module : "studentInfo",
        businessObject : "Contact"
      },
      mapping : {
        contactId : "id"
      },
      mappedBy : "contacts"
    }
    ```
    

  

  

## Views

These define the shape of data that is visible to the end user.

Any BO always has a default view, which can also be modified

```JSON
{
  $dt_version : "2307.0.510",
  dataStore : {
    type : "rdbms",
    useCache : false
  },
  fields : [ "departmentId" ]//what field will be visible in this view.
//This must only include fields which have access set to public.
}
```

collection is used to filter and sort the responses.

![[Screenshot_2023-06-13_at_12.49.55_AM.png]]

  

You can also construct query string to get access to fields which are public

$fields=f1,f2,f3..

$filter=cond1,cond2 ⇒ works like the where clause in sql, fields must me searchable

sortBy=f1:asc/desc,f2: ⇒ works like order by clause, fileds must be sortable

`/api/boss/data/objects/moduleName/v1/businessobjectname?$fields=lastName,firstName&$filter=studentId>200&$sortBy=birthDate:desc`

  

### Creating custom views

`boss view create -m moduleName -bo BusinessObjectName -v viewName <app-package-name>`

Or just create a file in the views directory.

“viewName.json5”

```JSON
{
  dataStore : {
    type : "rdbms"
  },
  fields : [ "id", "lastName", "email" ],
  accessors : {
    pii : {
      fields : ["studentId","birthDate"]
    }
  },
  collection : {
    sortBy : [{"pii.birthDate": "asc"}],
    limit : 5
  }
}
```

_**You need to repackage your application and deploy before you can access the new views created.**_

  

## Packaging and Deploying modules

`boss metadata package -m hellohr`

  

  

## APIs

[https://confluence.oraclecorp.com/confluence/display/BOSS/Business+Object+REST+API](https://confluence.oraclecorp.com/confluence/display/BOSS/Business+Object+REST+API)

You can access apis at 8000 on http and 8443 on https

You need to add all api versions in rest-api.json in the api folder.

```JSON
//rest-api.json5
{
  segments:["ora","scm","moduleName"],
//endPoint => /api/boss/data/objects/ora/scm/moduleName/v1/<businessObject
//corres module Name is oraScmModuleName
	versions : [ "v1","v2" ]
}
//v1.json5
{
  $dt_version : "2307.0.510",
  frameworkVersion : "1",
  resources : {
    employee : {
      allowList : true,
      allowGet : true,
      businessObject : "Employee"
    },...<List of all available BOs>
  },
  alias : "v1"
}
//v2.json5
{
  $dt_version : "2307.0.510",
  frameworkVersion : "1",
  resources : {
    employee : {
      allowList : true,
      allowGet : true,
      businessObject : "Employee"
    },...<List of all available BOs>
  },
  alias : "v1"
}
```

You can also specify segments in front of the api versions.

  

### POST APIs

To insert data into DB using BOs, set the _**“creatable”**_ property of the field to be _**“public” .**_

/api/boss/data/objects/moduleName/v1/businessObject

  

POST apis that do not create tables in the DB have $query attached at the end of the endpoint.

  

### Child BOs

It refers to objects within another object.

Such a relation type is called composition. It is generally a ont to many relation where the child object has a reference to the parent.

Other relations are reference types ⇒ they just point to another column dot own the object

Ex⇒ A building may have many classrooms. The existence of classrooms is conditional on the building

```JSON
//Building BO.
classrooms: {
  type: "object-collection",
  accessModifier: "public",
  target: {
    module: "college",
    businessObject: "Classroom"
  },
  mappedBy: "building", // refers to the Child BO
  relationshipType: "composition" // relation is composition
	//
}
//Classroom BO

building : {
  type : "object",
  accessModifier : "public",
  target : {
    module : "college",
    businessObject : "Building"
  },
  mapping : {
    buildingId : "id" //buildingId in classroom maps to id in Building
  },
  mappedBy : "classrooms" // refers to field in parent BO
}
```

  

### Update data using PATCH

The “updatable” field must be set to public

PATCH

`/api/boss/data/objects/studentInfo/v1/student/1000` ⇒ /student/<studId>

`{ "email":"jsmith@example.com"}`

  

Similarly to update a Child BO,

PATCH

`/api/boss/data/objects/college/v1/building/9000/classrooms/9020`

/building(BO)/<buildingId>/classroom(BO)/<classroomId>

```Plain
{
  "name": "Dat So La Lee"
}
```

  

### GET APIs

- `/api/boss/data/objects/studentInfo/v1/student/1000`

- `/api/boss/data/objects/studentInfo/v1/student?$filter=id=1000&$fields=id,lastName,email,pii.birthDate`

- `/api/boss/data/objects/studentInfo/v1/student/$views/youngestStudent`

- `http://<host-name>:8000/api/boss/data/objects/<moduleName>/v1/$openapi`  
    ⇒has all the metadata for the module, about its bos etc.

  

  

boss-cli-config.yaml

application.json

app-apckage.json5

  

## Custom Datatypes:

### Restriction Type

- dir ⇒ sources/model/self/modules/<moduleName>/restrictionTypes

```JSON
//Restriction Type => StudentCode.json5
{
    baseType: "string",
    format: "student-code",
    pattern: "[CDE][A-Z]{3}-[0-9]{5}"
}
//Rating.json5
{
    baseType: "int32",
    format: "rating",
    minimum: 1,
    maximum: 10
}
```

```JSON
studentCode: {
      type: "restriction",
      accessModifier : "public",
      target : {
        restriction: "StudentCode"
      },
      dataStoreMapping : {
        rdbms : {
          column: "STUDENT_CODE"
        }
      },
      nullable : false,
      creatable : "public",
      updatable : "public",
      readable : "public",
    }
}
```

  

### Enumeration

- dir ⇒ sources/model/self/modules/<moduleName>/enums

```JSON
//ContactType.json5
{
    values : [ {
        code: "MOTHER",//const value assigned
        display: "Mother", //what that value represents
        description: "Mother of the person's contact." 
    }, {      
        code: "FATHER",
        display: "Father"
    },
    .
    .
    .
    }, {
        code: "UNKNOWN",
        display: "Unknown"
    } ]
}
```

```JSON
contactType : {
    type: "enumeration",
    accessModifier: "public",
    target : {
      enumeration: "ContactType"//FIlename in enums folder
    },
    dataStoreMapping : {
      rdbms : {
        column: "RELATIONSHIP"
      }
    },
    nullable : false,
    creatable : "public",
    updatable : "public",
    readable : "public",
  }
```

  

## Composite

- dir ⇒ sources/model/self/modules/<moduleName>/compositeTypes

```JSON
//PersonName.json5
{
  fields: {
    first: {
      type: "string",
      required: true,
      nullable: false
    },
    last: {
      type: "string",
      required: true,
      nullable: false
    }
  }
}
```

```JSON
StudentName : {
  type: "composite",
  accessModifier: "public",
  target: {
    composite: "PersonName"
  },
  fields : {
    first: {
      creatable: "public",
      updatable: "public",
      dataStoreMapping: {
        rdbms: {
          column: "FIRST_NAME",
          searchable: true,
          sortable: true
        }
      }
    },
    last: {
      creatable: "public",
      updatable: "public",
      dataStoreMapping: {
        rdbms: {
          column: "LAST_NAME",
          searchable: true,
          sortable: true
        }
      }
    }
  }
}
```

  

### Joins

```JSON
LineType: {
	type: "object",
	target: {
		businessObject: "LookCode", 
		Filter: "LookupType = 'INVOICE LINE TYPE'"
	},
	mapping: {
		LineTypeLookupCode: "LookupCode"
	},
	JoinType: "leftOuterJoin", 
	dynamicLookup: {},
	accessModifier: "public"
}
```

  

  

Extractions:

See the doc.

For query over post POST

- Define datastore rod in module,bo and the referencing view.

- use end pt: http://host:8000/api/boss/extraction/extract/<module>/:<deploymentNumber>

For GET!!!

define the request in <moduleName>/extractions/<extName>

- Expose the extration in the api version file.

  

_CODE = lookup

_ID = id columns

versioning column ⇒