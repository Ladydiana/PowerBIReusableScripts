# PowerBI Reusable Scripts

- [Display Logged user](#display-logged-user)
- [DAX Studio - Get all measures](#dax-studio---get-all-measures)


-------------------------------------------------


# Display Logged user
```
// DAX
_UserLogin = USERNAME()
```

# DAX Studio - Get all measures
```sql
SELECT MEASUREGROUP_NAME as TABLE_NAME, MEASURE_NAME, EXPRESSION  FROM $SYSTEM.MDSCHEMA_MEASURES
WHERE MEASURE_IS_VISIBLE
```