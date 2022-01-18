# Measure Selector Example

```
// Step 1. Create Power Query table
/*
		TABLE SCHEMA
General 2x2 table with Name and Code columns. 
For the purpose of this example, it is populated statically with "Queued" and "Successful" values.
 _______________________________
| 		Name	|		Code	|
|_______________|_______________|
|   Queued		|	  	0		|
| 	Successful	|	  	1 		|
|_______________|_______________|

*/

let
    Source = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("i45WCi5NTk4tLk4rzVHSUTJUitWJVgosTS1NTQFyDZRiYwE=", BinaryEncoding.Base64), Compression.Deflate)), let _t = ((type nullable text) meta [Serialized.Text = true]) in type table [Name = _t, Code = _t]),
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Name", type text}, {"Code", Int64.Type}})
in
    #"Changed Type"
```

```
// Step 2. Create measures

Selected Measure Code= SELECTEDVALUE('Measure Selection'[Code],1)
Selected Measure Name = SELECTEDVALUE('Measure Selection'[Name])

Functional All Measures = 
SWITCH(
[Selected Measure Code],
0,[_CalculationForSuccessful],
1,[_CalculationForQueued]
)

// where _CalculationForSuccessful and _CalculationForQueued are additional measures created to calculate the results which should be switched between

```