# I. AJ Related Open-Items Grid

Main API for Open Items:
POST https://twdev.repfabric.com/rfnextgenapi/api/v1/open-items

### Request Body Example:
```json
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
           // {
            //     "field": "type",
            //     "operator": "contains",
            //     "value": "o"
            // },
            // {
            //     "field": "topic",
            //     "operator": "contains",
            //     "value": "68"
            // },
            //  {
            //     "field": "topic",
            //     "operator": "isAnyOf",
            //     "value": ["newssd","test"]
            // }
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

---

## II. Grid Functionality
Each row in the grid represents an open item and includes the following fields:

| Field       | Editable | Opportunity | Job | Quote |
|------------|----------|------------|-----|-------|
| `topic`     | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| `distributor` | ❌ No | ✅ Yes | ❌ No | ✅ Yes |
| `stage`     | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| `value`     | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| `type`      | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes |

### Editing Behavior
- Inline editing is supported for individual fields.
- All fields except `type` & `distributor` are editable.
- `type` is a hardcoded string to determine the type (hence, no edits).
- The `stage` field provides a dropdown menu for users to select from available options, updating only the respective row.
- **Job** - The `distributor` field is not available and will be `NULL` by default (hence, no edits).

---

## III. Functionalities
- The data is retrieved based on the POST request body.
- Open-Items data is fetched using `principalId` and `customerId` from *(Table: Opportunities, Job_Related_Comps, Quotes_Hdr)*.
- The *(Table: Job_Related_Comps)* contains the `JOB_RELT_COM_ID` column, which is `principalId` and also `customerId`. Using these two IDs, we can get Job IDs.
  - **Opportunity** *(Table: OPPORTUNITIES)*: Mapped using `OPP_ID`
  - **Job** *(Table: JOBS)*: Mapped using `JOB_ID`
  - **Quote** *(Table: QUOTE_HDR)*: Mapped using `REC_ID`
- The total number of unique tables used in these queries is **8**. Here’s the list:
    - **OPPORTUNITIES**
    - **COMPANIES**
    - **OPP_ACTIVITIES_MST**
    - **JOB_RELT_COM_ID**
    - **JOBS**
    - **JOB_ACTIVITIES_MST**
    - **QUOTES_HDR**
    - **DATA_OPTIONS**

---

### IV. **Stage Logic**

#### **Opportunity Stage**
- The stage is retrieved from *(Table: OPP_ACTIVITIES_MST)*.
- It tries to match the opportunity's activity (`OP.OPP_ACTIVITY`) with `OAM.ACT_NAME`, but it also considers the **principal ID** (`OP.OPP_PRINCIPAL = OAM.ACT_COMP_ID`).
- If no match is found, two fallback mechanisms are used:
  1. **Fallback 1**: Matches the activity (`OP.OPP_ACTIVITY`) but without the principal ID constraint (`OAM_Fallback.ACT_COMP_ID = 0`).
  2. **Fallback 2**: If the first fallback does not find a match, it selects the first available record where `ACT_COMP_ID = 0`, ordered by `REC_ID ASC` (`OAM_Fallback_Default`).

#### **Job Stage**
- The stage is retrieved from *(Table: JOB_ACTIVITIES_MST)*.
- It directly matches the `JOB.JOB_ACTIVITY` field with `JAM.REC_ID`.

#### **Quote Stage**
- The stage is retrieved from *(Table: DATA_OPTIONS)*.
- It matches `Q.QUOT_DELIV_STATUS` with `D.REC_ID`.

---

### V. **Dropdown List Retrieval Logic**

GET https://twdev.repfabric.com/rfnextgenapi/api/v1/open-items/stage-dropdown/opportunity/34
GET https://twdev.repfabric.com/rfnextgenapi/api/v1/open-items/stage-dropdown/job
GET https://twdev.repfabric.com/rfnextgenapi/api/v1/open-items/stage-dropdown/quote

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

---

### VI. **Updating Fields (Inline Edit)**

Update API:
PATCH https://twdev.repfabric.com/rfnextgenapi/api/v1/open-items

#### Request Body Example:
```json
{
    "id": 22,
    "type": "Opportunity",
    "topic": "aaa-U1",
    "distributor": "Syncdistributor-U2",
    "distributorId": 36,
    "stage": {
        "recId": 1849,
        "name": "qwe1"
    },
    "value": "10.00"
}
```

---

After any PATCH update, the following GET API must be called to update various tables:
```
https://twdev.repfabric.com/rf_api/activityActionHandler.php?opp_id=${id}&user_id=${userId}
```
---

