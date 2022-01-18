# Last Refresh of the Dataset

```
// Step 1: Power Query
// Generate new date table LastRefreshedUTCNow which refreshes at every dataset refresh
let
    Source = DateTimeZone.FixedUtcNow()
in
    Source
	
// Step 2: DAX
TimeFromLastRefreshedDate = 
var TimeDiff = ABS(DATEDIFF(UTCNOW(),MAX(LastRefreshedUTCNow[LastRefreshedUTCNow]), SECOND))
// or use this for max date in case this is coming from a table, in order to not be impacted by any filters:
// MAXX(ALL(LastRefreshedUTCNow[LastRefreshedUTCNow]), OPENTEXT_AGGREGATION[LastRefreshedUTCNow[LastRefreshedUTCNow])
var TimeMin = ROUNDDOWN(DIVIDE(TimeDiff,60),0)
var TimeHour = ROUNDDOWN(DIVIDE(TimeDiff,60*60),0)
var TimeDay = ROUNDDOWN(DIVIDE(TimeDiff,60*60*24),0) 
var TimeFormat = SWITCH( TRUE(), 
    TimeDiff<60, TimeDiff & " seconds ago",
    TimeDiff<3600,  IF(TimeMin = 1, TimeMin & " minute ago", TimeMin & " minutes ago"),
    TimeDiff<86400, IF(TimeHour = 1, TimeHour & " hour ago", TimeHour & " hours ago"),
    IF(TimeDay = 1, TimeDay & " day ago", TimeDay & " days ago")
)
return "Last Refresh: " & TimeFormat
```