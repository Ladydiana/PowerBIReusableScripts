# PowerBI Reusable Scripts

- [Display Logged user](#display-logged-user)
- [DAX Studio - Get all measures](#dax-studio---get-all-measures)
- [Calendar](#calendar)
    + [M Generated Calendar](#m-generated-calendar)
	+ [Contiguous Calendar From Source](#contiguous-calendar-from-source)


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

# Calendar
### M Generated Calendar
```
let
    Source = #"Script to generate calendar"(Date.StartOfYear(DateTime.Date(Date.AddYears(KeyDateValue,YearOffset))), Date.StartOfYear(DateTime.Date(Date.AddYears(KeyDateValue,1))), null),
    #"Invoked Custom Function" = Table.AddColumn(Source, "ISOWeek", each ISOWeek([Date])),
    #"Invoked Custom Function1" = Table.AddColumn(#"Invoked Custom Function", "Week BW", each #"Week BW"([Date])),
    #"Invoked Custom Function2" = Table.AddColumn(#"Invoked Custom Function1", "WeekNumber", each WeekNumber([Date])),
    #"Changed Type" = Table.TransformColumnTypes(#"Invoked Custom Function2",{{"WeekNumber", Int64.Type}, {"Year", Int64.Type}, {"QuarterOfYear", Int64.Type}}),
    #"Sorted Rows" = Table.Sort(#"Changed Type",{{"Year", Order.Ascending}}),
    #"Duplicated Column" = Table.DuplicateColumn(#"Sorted Rows", "Year", "Year - Copy"),
    #"Renamed Columns" = Table.RenameColumns(#"Duplicated Column",{{"Year - Copy", "LabelYear"}}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Renamed Columns",{{"LabelYear", type text}}),
    #"Added YTD" = Table.AddColumn(#"Changed Type", "YTD", each if Date.Year([Date])=Date.Year(KeyDateValue) and [Date] <= DateTime.Date(KeyDateValue) then "YTD" else ""),
    #"Added CY" = Table.AddColumn(#"Added YTD", "CY", each if Date.Year([Date])=Date.Year(KeyDateValue) then "CY" else ""),
    #"Added LY" = Table.AddColumn(#"Added CY", "LY", each if Date.Year([Date])=Date.Year(Date.AddYears(KeyDateValue,-1)) then "LY" else ""),
    #"Added LM" = Table.AddColumn(#"Added LY", "LM", each if Date.Year([Date])=Date.Year(Date.AddMonths(KeyDateValue,-1)) and Date.Month([Date])=Date.Month(Date.AddMonths(KeyDateValue,-1))then "LM" else ""),
    #"Added YTLM" = Table.AddColumn(#"Added LM", "YTLM", each if  Date.Year([Date])=Date.Year(KeyDateValue) and [Date] <= DateTime.Date(Date.EndOfMonth(Date.AddMonths(KeyDateValue,-1))) then "YTLM" else ""),
	#"Added MTD" = Table.AddColumn(#"Added YTLM", "MTD", each if Date.Year([Date])=Date.Year(KeyDateValue) and Date.Month([Date])=Date.Month(KeyDateValue) and [Date] <= DateTime.Date(KeyDateValue) then "MTD" else ""),
    #"Added CM" = Table.AddColumn(#"Added MTD", "CM", each if Date.Year([Date])=Date.Year(KeyDateValue) and Date.Month([Date])=Date.Month(KeyDateValue) then "CM" else ""),
    #"Added QTD" = Table.AddColumn(#"Added CM", "QTD", each if Date.Year([Date])=Date.Year(KeyDateValue) and Date.QuarterOfYear([Date])=Date.QuarterOfYear(KeyDateValue) and [Date] <= DateTime.Date(KeyDateValue) then "QTD" else ""),
    #"Added WTD" = Table.AddColumn(#"Added QTD", "WTD", each if Date.Year([Date])=Date.Year(KeyDateValue) and Date.WeekOfYear([Date],Day.Monday)=Date.WeekOfYear(KeyDateValue) and [Date] <= DateTime.Date(KeyDateValue) then "WTD" else ""),
    #"TimePeriodKey Merged Column" = Table.AddColumn(#"Added WTD", "TimePeriodKey", each Text.Combine({[YTD], [CY],  [LY], [CM], [MTD], [QTD], [WTD], [LM], [YTLM]}, "|"), type text),
    #"Changed Type2" = Table.TransformColumnTypes(#"TimePeriodKey Merged Column",{{"DateInt", type text}}),
    #"Inserted Replaced Text" = Table.AddColumn(#"Changed Type2", "Replaced Text", each Text.Replace([ISOWeek], "W", ""), type text),
    #"Renamed Columns1" = Table.RenameColumns(#"Inserted Replaced Text",{{"Replaced Text", "IsoWeekNumber"}}),
    #"Changed Type3" = Table.TransformColumnTypes(#"Renamed Columns1",{{"IsoWeekNumber", Int64.Type}}),
    #"Renamed Columns2" = Table.RenameColumns(#"Changed Type3",{{"IsoWeekNumber", "ISOWeekNumber"}}),
    #"Duplicated Column1" = Table.DuplicateColumn(#"Renamed Columns2", "Week BW", "Week BW - Copy"),
    #"Split Column by Delimiter" = Table.SplitColumn(#"Duplicated Column1", "Week BW - Copy", Splitter.SplitTextByDelimiter(".", QuoteStyle.Csv), {"Week BW - Copy.1", "Week BW - Copy.2"}),
    #"Inserted Merged Column" = Table.AddColumn(#"Split Column by Delimiter", "Merged", each Text.Combine({[#"Week BW - Copy.2"], [#"Week BW - Copy.1"]}), type text),
    #"Removed Columns" = Table.RemoveColumns(#"Inserted Merged Column",{"Week BW - Copy.1", "Week BW - Copy.2"}),
    #"Renamed Columns3" = Table.RenameColumns(#"Removed Columns",{{"Merged", "Week BW Number"}}),
    #"Changed Type4" = Table.TransformColumnTypes(#"Renamed Columns3",{{"DayInWeek", Int64.Type}, {"QuarterInCalendarBW", type text}, {"QuarterInCalendar", type text}, {"MonthInCalendar", type text}}),
    #"Duplicated Column2" = Table.DuplicateColumn(#"Changed Type4", "QuarterInCalendar", "QuarterInCalendar - Copy"),
    #"Split Column by Delimiter1" = Table.SplitColumn(#"Duplicated Column2", "QuarterInCalendar - Copy", Splitter.SplitTextByDelimiter(" ", QuoteStyle.Csv), {"QuarterInCalendar - Copy.1", "QuarterInCalendar - Copy.2"}),
    #"Changed Type5" = Table.TransformColumnTypes(#"Split Column by Delimiter1",{{"QuarterInCalendar - Copy.1", type text}, {"QuarterInCalendar - Copy.2", Int64.Type}}),
    #"Replaced Value" = Table.ReplaceValue(#"Changed Type5","Q","0",Replacer.ReplaceText,{"QuarterInCalendar - Copy.1"}),
    #"Changed Type6" = Table.TransformColumnTypes(#"Replaced Value",{{"Week BW", type text}, {"ISOWeek", type text}}),
    #"Inserted Merged Column1" = Table.AddColumn(#"Changed Type6", "Merged", each Text.Combine({Text.From([#"QuarterInCalendar - Copy.2"], "cs-CZ"), [#"QuarterInCalendar - Copy.1"]}), type text),
    #"Renamed Columns4" = Table.RenameColumns(#"Inserted Merged Column1",{{"Merged", "QuarterInCalendar number"}}),
    #"Removed Columns1" = Table.RemoveColumns(#"Renamed Columns4",{"QuarterInCalendar - Copy.1", "QuarterInCalendar - Copy.2"}),
    #"Renamed Columns5" = Table.RenameColumns(#"Removed Columns1",{{"QuarterInCalendar number", "QuarterInCalendar Number"}}),
    #"Changed Type7" = Table.TransformColumnTypes(#"Renamed Columns5",{{"QuarterInCalendar Number", Int64.Type}, {"Week BW Number", Int64.Type}}),
    #"Duplicated Column3" = Table.DuplicateColumn(#"Changed Type7", "QuarterInCalendarBW", "QuarterInCalendarBW - Copy"),
    #"Split Column by Delimiter2" = Table.SplitColumn(#"Duplicated Column3", "QuarterInCalendarBW - Copy", Splitter.SplitTextByDelimiter(".", QuoteStyle.Csv), {"QuarterInCalendarBW - Copy.1", "QuarterInCalendarBW - Copy.2"}),
    #"Inserted Merged Column2" = Table.AddColumn(#"Split Column by Delimiter2", "Merged", each Text.Combine({"0", [#"QuarterInCalendarBW - Copy.2"]}), type text),
    #"Removed Columns2" = Table.RemoveColumns(#"Inserted Merged Column2",{"QuarterInCalendarBW - Copy.2"}),
    #"Inserted Merged Column3" = Table.AddColumn(#"Removed Columns2", "QuarterInCalendarBW Number", each Text.Combine({[#"QuarterInCalendarBW - Copy.1"], [Merged]}), type text),
    #"Removed Columns3" = Table.RemoveColumns(#"Inserted Merged Column3",{"QuarterInCalendarBW - Copy.1", "Merged"}),
    #"Inserted Merged Column4" = Table.AddColumn(#"Removed Columns3", "Month", each Text.Combine({"0", Text.From([MonthOfYear], "cs-CZ")}), type text),
    #"Duplicated Column4" = Table.DuplicateColumn(#"Inserted Merged Column4", "Year", "Year - Copy"),
    #"Changed Type9" = Table.TransformColumnTypes(#"Duplicated Column4",{{"Year - Copy", type text}}),
    #"Replaced Value1" = Table.ReplaceValue(#"Changed Type9","010","10",Replacer.ReplaceText,{"Month"}),
    #"Replaced Value2" = Table.ReplaceValue(#"Replaced Value1","011","11",Replacer.ReplaceText,{"Month"}),
    #"Replaced Value3" = Table.ReplaceValue(#"Replaced Value2","012","12",Replacer.ReplaceText,{"Month"}),
    #"Filtered Rows" = Table.SelectRows(#"Replaced Value3", each true),
    #"Inserted Merged Column5" = Table.AddColumn(#"Filtered Rows", "YearMonth Number", each Text.Combine({[#"Year - Copy"], [Month]}), type text),
    #"Removed Columns4" = Table.RemoveColumns(#"Inserted Merged Column5",{"Month", "Year - Copy"}),
    #"Changed Type8" = Table.TransformColumnTypes(#"Removed Columns4",{{"DayOfMonth", Int64.Type}, {"MonthOfYear", Int64.Type}, {"YTD", type text}, {"LY", type text}, {"MTD", type text}, {"QTD", type text}, {"WTD", type text}}),
    #"Removed Columns5" = Table.RemoveColumns(#"Changed Type8",{"DayInWeek"}),
    #"Inserted Day of Week" = Table.AddColumn(#"Removed Columns5", "Day of Week", each Date.DayOfWeek([Date], Day.Monday)),
    #"Renamed Columns6" = Table.RenameColumns(#"Inserted Day of Week",{{"Day of Week", "Day of Week Number"}}),
    #"Changed Type10" = Table.TransformColumnTypes(#"Renamed Columns6",{{"Day of Week Number", Int64.Type}, {"QuarterInCalendarBW Number", Int64.Type}, {"YearMonth Number", Int64.Type}}),
    #"Added FutureDate" = Table.AddColumn(#"Changed Type10", "Future Date", each if [Date] > Date.From(KeyDateValue) then "Future" else "Past"),
    #"Added CurOffSetMonth" = Table.AddColumn(#"Added FutureDate", "CurMonthOffSet", each (Date.Year([Date])- Date.Year(Date.From(KeyDateValue))) * 12 + Date.Month([Date])-Date.Month(Date.From(KeyDateValue))),
    #"Added CurWeekOffSet" = Table.AddColumn(#"Added CurOffSetMonth", "CurWeekOffSet", each (Date.Year([Date])- Date.Year(Date.From(KeyDateValue))) * 12 + Date.WeekOfYear([Date])-Date.WeekOfYear(Date.From(KeyDateValue))),
    #"Added IsInCurrentYear" = Table.AddColumn(#"Added CurWeekOffSet", "IsInCurrentYear", each if Date.Year([Date])=Date.Year(KeyDateValue) then "X" else ""),
    #"Added YTD-1" = Table.AddColumn(#"Added IsInCurrentYear", "YTD-1", each if Date.Year([Date])=Date.Year(Date.AddYears(KeyDateValue,-1))  and [Date] <= Date.AddYears(DateTime.Date(KeyDateValue),-1) then "X" else ""),
    #"Added MMM" = Table.AddColumn(#"Added YTD-1", "Month(MMM)", each Date.ToText([Date],"MMM"))
in
    #"Added MMM"
```

### Contiguous Calendar From Source
```
// DAX
Dates =
CALENDAR (
    DATE ( YEAR ( MIN ( Sales[Order Date] ) ), 1, 1 ),
    DATE ( YEAR ( MIN ( Sales[Order Date] ) ), 12, 31 )
)
```