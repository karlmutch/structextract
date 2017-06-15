[![Coverage Status](https://coveralls.io/repos/github/intelligentpos/structextract/badge.svg)](https://coveralls.io/github/intelligentpos/structextract)

# structextract
A very small package that extracts a given struct to an array or to a map.
There is option to ignore fields or to use the tag names as key on the struct.

## Install

```bash
go get github.com/intelligentpos/structextract
```

## Examples 

#### Basic Usage
```go
       type SampleStruct struct {
		Field1 string `json:"field1" db:"field_1_db"`
		Field2 string `json:"field2" db:"field_2_db"`
		Field3 bool   `json:"field3" db:"field_3_db"`
		Field4 int    `json:"field4"`
	}
	

	ss := SampleStruct{
		Field1: "value 1",
		Field2: "value 2",
		Field3: true,
		Field4: 123,
	}
	
    // Create a new extractor,we have to pass a pointer to a struct
	extract := New(&ss)
	

	//Values will return the values of every field, []interface
	//["value 1", "value 2", true, 123]
	values, _ := extract.Values()
	

	//Names will return an array of the field names, []string
	//["Field1","Field2","Field3","Field4"]
	names, _ := extract.Names()
	

	//NamesFromTag will return an array of the tag value for the given tag, map[string]interface{}
	//["field_1_db","field_2_db","field_3_db"]
	tagnames, _ := extract.NamesFromTag("db")
	

	//FieldValueMap will return a map of field name to value, map[string]interface{}
	//{"Field1":"value 1","Field2":"value 2","Field3":true,"Field4":123}
	valuesmap, _ := extract.FieldValueMap()
	

	//FieldValueFromTagMap will return a map of tag value to value, map[string]interface{}
	//{"field1":"value 1","field2":"value 2","field3":true,"field4":123}
	tagmap, _ := extract.FieldValueFromTagMap("json")
	
```
#### Ignore Fields
```go
       ss := SampleStruct{
		Field1: "value 1",
		Field2: "value 2",
		Field3: true,
		Field4: 123,
	}
	

	// Set all the struct fields that we need to ignore
	extract := New(&ss).
		IgnoreField("Field2","Field3")

	
	//The result will be {"Field1":"value 1","Field4":123},
	//all the fields that are ignored are not present.
	valuesmap, _ := extract.FieldValueMap()
	
```

#### Use cases

We found that is very convenient to use structextract when we want to create sql statements 
or maps with data to update or create.

A sample example that we use the structextract with [Squirrel](https://github.com/Masterminds/squirrel).

```go
       type SampleStruct struct {
		Field1 string `json:"field1" db:"field_1_db"`
		Field2 string `json:"field2" db:"field_2_db"`
		Field3 bool   `json:"field3" db:"field_3_db"`
		Field4 int    `json:"field4"`
	}
	

	ss := SampleStruct{
		Field1: "value 1",
		Field2: "value 2",
		Field3: true,
		Field4: 123,
	}        
        
        //Create a map with all the fields a user can update 
	ext := structextract.New(&ss).
		IgnoreField("Field2")
		
    
	bm, _ := ext.FieldValueFromTagMap("db")
	
	//Build the query
	query, args, _ := squirrel.Update("DBTable").
		SetMap(bm).
		Where(squirrel.Eq{"id": ss.Field1}).
		ToSql()
		
	//Make the sql request..	
```

