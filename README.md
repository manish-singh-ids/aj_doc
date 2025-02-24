# I. AJ Related Open-Items Grid

Main Api for OpenItems - 
POST https://twdev.repfabric.com/rfnextgenapi/api/v1/open-items

Request Body Example : 

```
{
    "pagination": {
        "page": 1,
        "size": 10
    },
    "filter": {
        "ABC": [
            12
        ]
    },
    "columnFilters": {
        "items": [
            {
                "field": "type",
                "operator": "contains",
                "value": "op"
            },
            {
                "field": "customer",
                "operator": "contains",
                "value": "ww"
            }
        ],
        "logicOperator": "OR",
        "quickFilterValues": ["A"], //Optional Search; Use when you want to search in the result which is filtered by column Already Or Make a global search without ColumFilter (Both Works)
        "quickFilterLogicOperator": "AND"
    },
    "grid": {
        "openItems": {
            "ajDtlId": 11439
        }
    },
    "gridSort": [
        {
            "field": "type",
            "sort": "asc"
        }
    ]
}

```


## Requirements

For detailed task requirements, refer to the following link:  
[Task 15271](https://pms.indeadesignsystems.com/task_management/tasks/15271)

## Overview

The AJ Related Open-Items should be displayed together in a grid format. The grid includes the following record types:

- **Opportunity**
- **Job**
- **Quote**

<hr>

## II. Grid Functionality

Each row in the grid represents an open item and includes the following fields:

| Field       | Editable | Opportunity | Job | Quote |
|------------|----------|------------|-----|-------|
| `topic`     | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| `customer`  | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| `distributor` | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes |
| `stage`     | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| `value`     | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| `type`      | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes |

### Editing Behavior
- Inline editing is supported for individual fields.
- All fields except `type` are editable.
- `type` is a hardcoded string to determine the type (hence, no edits).
- The `stage` field provides a dropdown menu for users to select from available options, updating only the respective row.
- **Job** - The `distributor` field is not available and will be `NULL` by default (hence, no edits).

<hr>

## III. Functionalities
- The data is retrieved based on the POST request body.
- Open-Items data is fetched using `AJ_DTL_ID` from *(Table: AJ_LINKS)*.
- The *(Table: AJ_LINKS)* contains the `AJ_LINK_ID` column, which maps to different tables:
  - **Opportunity** *(Table: OPPORTUNITIES)*: Mapped using `OPP_ID`
  - **Job** *(Table: JOBS)*: Mapped using `JOB_ID`
  - **Quote** *(Table: QUOTE_HDR)*: Mapped using `REC_ID`
-  The total number of unique tables used in these queries is **8**. Here’s the list:
    - **AJ_LINKS**
    - **OPPORTUNITIES**
    - **COMPANIES**
    - **OPP_ACTIVITIES_MST**
    - **JOBS**
    - **JOB_ACTIVITIES_MST**
    - **QUOTES_HDR**
	- **DATA_OPTIONS**

Each query uses a combination of these tables:

- **Opportunity Query** uses: `AJ_LINKS`, `OPPORTUNITIES`, `COMPANIES` (twice as C and C1), `OPP_ACTIVITIES_MST` (twice as OAM and OAM_Fallback_Default).
- **Job Query** uses: `AJ_LINKS`, `JOBS`, `COMPANIES`, `JOB_ACTIVITIES_MST`.
- **Quote Query** uses: `AJ_LINKS`, `QUOTES_HDR`, `COMPANIES` (twice as C and C1), `DATA_OPTIONS`.

<hr>

### IV. **Stage Logic**

#### **Opportunity Stage**
- The stage is retrieved from *(Table: OPP_ACTIVITIES_MST)*.
- It tries to match the opportunity's activity (`OP.OPP_ACTIVITY`) with `OAM.ACT_NAME`, but it also considers the **principal ID** (`OP.OPP_PRINCIPAL = OAM.ACT_COMP_ID`).
- If no match is found, a **fallback** is used (`OAM_Fallback_Default`), where the `ACT_COMP_ID` is **0**.

#### **Job Stage**
- The stage is retrieved from *(Table: JOB_ACTIVITIES_MST)*.
- It directly matches the `JOB.JOB_ACTIVITY` field with `JAM.REC_ID`.

#### **Quote Stage**
- The stage is retrieved from *(Table: DATA_OPTIONS)*.
- It matches `Q.QUOT_DELIV_STATUS` with `D.REC_ID`.

<hr>

### V. **Dropdown List Retrieval Logic**

#### **Tables Involved**
- *(Table: DATA_OPTIONS)* (For Quote stages)
- *(Table: JOB_ACTIVITIES_MST)* (For Job stages)
- *(Table: OPP_ACTIVITIES_MST)* (For Opportunity stages)

#### **Quote Stage List**
- **Table Queried**: *(Table: DATA_OPTIONS)*
- **Query Condition**: `OPT_DATA_SOURCE = 'QUOTE'`

#### **Job Stage List**
- **Table Queried**: *(Table: JOB_ACTIVITIES_MST)*
- **Sorting**: Ordered by `ACT_SORT_ID ASC`

#### **Opportunity Stage List**
- **Table Queried**: *(Table: OPP_ACTIVITIES_MST)*
- **Condition**: `ACT_COMP_ID = :principalId`, fallback `ACT_COMP_ID = 0`.

### **Error Handling**
- If `gridName` is missing, it returns a **Bad Request** error.
- If `gridName = OPPORTUNITY` but `principalId` is missing, it also returns a **Bad Request** error.
- If an unexpected error occurs, it returns an **Internal Server Error**.

<hr>

### VI. **Updating Fields (Inline Edit)**

update API 
PATCH https://twdev.repfabric.com/rfnextgenapi/api/v1/open-items

request Body Example 

```
{
    "id": 22,
    "type": "Opportunity",
    "topic": "aaa-U1",
    "customer": "Synccustomer-U1",
    "customerId": 35,
    "distributor": "Syncdistributor-U2",
    "distributorId": 36,
    "stage": {
        "recId": 1849,
        "name": "qwe1"
    },
    "value": "10.00"
}
```


#### **Opportunity Update**
- **Tables Involved**: *(Table: OPPORTUNITIES)*, *(Table: COMPANIES)*, *(Table: OPP_ACTIVITIES_MST)*.
- **Stage Update Logic**: Updates `OPP_ACTIVITY`.

#### **Job Update**
- **Tables Involved**: *(Table: JOBS)*, *(Table: COMPANIES)*, *(Table: JOB_ACTIVITIES_MST)*.
- **Stage Update Logic**: Updates `JOB_ACTIVITY`.

#### **Quote Update**
- **Tables Involved**: *(Table: QUOTES_HDR)*, *(Table: COMPANIES)*, *(Table: DATA_OPTIONS)*.
- **Stage Update Logic**: Updates `QUOT_DELIV_STATUS`.

---






