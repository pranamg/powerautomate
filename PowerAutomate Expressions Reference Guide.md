# Power Automate Gymnastics Reference Guide #

_Sourced from:_
[Power Automate Gymnastics Reference Guide](https://crmtipoftheday.com/pages/power-automate-gymnastics-reference-guide/)

_Source Contributors: Amey “ABC” Holden, Antti Pajunen, Matt Collins-Jones._

> Have the favorite expression of your own? Send it straight to jar@crmtipoftheday.com

## List, Array, and Loop Walk Into A Bar ##

First items contact ID from List Rows

```python
first(outputs('List_Rows_Action_Name')?['body/value'])?['contactid']
```

First items OData ID (For UriHost()) from List Rows

```python
first(outputs('List_Rows_Action_Name')?['body/@odata'])?['id']
```

List record action returned no results – output = true/false

```python
empty(body('List_Rows_Action_Name')?['value'])
```

Choose a certain value from an HTTP request instead of using Parse JSON

```python
body('Invoke_an_HTTP_request')?['mail']
```

Get the first result for an array

```python
first(output('array'))

output('array')?[0]
```

Get the field from the first result of an array

```python
output('array')?[0]?['FirstName']
```

Check how many results are returned

```python
length(output('array'))
```

Check if array is blank

```python
empty(output('array'))
```

Skip lines for array

```python
Skip(('array'),6)
```

Split tab delimiter file into an array

```python
split(outputs('Get_File_from_SharePoint')?['body'], decodeUriComponent('%09'))
```

Split Array by carriage return

```python
split(string(item()), decodeUriComponent('%0A'))
```

## Conditions ##

Go through this list of stuff and return the first non-blank value

```python
coalesce(
   outputs('Get_Row_Action')?['body/description'], 
   outputs('Get_Row_Action')?['body/name'], 
   'Default value'
)
```

Check if something is blank and provide a different value, else use the first value

```python
if(
    empty(variables('varEmail')),
        'foobar@gmail.com',
        variables('varEmail')    
)
```

Check if something is blank and provide a different value, else use the first value (Mk II)

```python
coalesce(variables('varEmail'),  'foobar@gmail.com')
```

If Empty Condition

```python
if(empty(outputs('Get_Row_Action')?['body/industrycode']), 'Yes', 'No')
```

If Not Equal Condition

```python
if(not(equals(outputs('Get_Row_Action')?['body/industrycode'], 1)), 'Yes', 'No')
```

If Equals Condition

```python
if(equals(outputs('Get_Row_Action')?['body/industrycode'], 1), 'Yes', 'No')
```

Set the owner correctly regardless if it’s a user or a team

```python
concat(
  outputs('Get_Row_Action')?['body/_ownerid_value@Microsoft.Dynamics.CRM.lookuplogicalname'],
  's/',
  outputs('Get_Row_Action')?['body/_ownerid_value']
)
```

If empty do nothing, otherwise set the lookup (avoids failures when lookup value not found)

(Table is actually called lookup_table but in their wisdom designers of OData specifications introduced plural so it’s lookup_tables)

```python
if(empty(first(outputs('Get_Rows_Action')?['body'])?['GUID_Column']), 
    null,
    concat(
      'lookup_tables(',
      first(outputs('Get_Rows_Action')?['body'])?['GUID_Column'],
      ')'
    )
)
```

If customer lookup is to an account, set the lookup, otherwise do nothing

```python
if(equals(
  triggerOutputs()?['body/_customerid_value@Microsoft.Dynamics.CRM.lookuplogicalname'],
  'account'), 
    concat('accounts(',triggerOutputs()?['body/_customerid_value'],')'), 
    null
)
```

If Equals many conditions

```python
if(
  and(
    equals(triggerOutputs()?['body/firstname'], 'Mike'), 
    equals(triggerOutputs()?['body/lastname'], 'Ipsum')
  ), 
'Yes', 'No')

```

Use with the switch case value, so if its blank, your flow doesn’t fail

```python
coalesce(triggerBody()?['Choice']?['Value'],'Unknown')
```

## Data Versing ##

Check two string values are exact

```python
toLower(item()?['matt_email'])
```

Remove whitespace in string

```python
replace(triggerOutputs()?['body/productnumber'], ' ','')
```

Line breaks/new lines

```python
decodeUriComponent('%0A')
decodeUriComponent('%0D')

```

Remove last character of string

```python
substring(outputs('Compose_3'),0,sub(length(outputs('Compose_3')),1))
```

TimeZone Conversion for AUS

```python
convertTimeZone(utcNow(),'UTC','W. Australia Standard Time')
convertTimeZone(utcNow(),'UTC','AUS Eastern Standard Time')
```

Date/Time Formatting

<https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings>

Extracting a word from a sentence

```python
first(split(last(split(outputs('Compose_random_string'), 'word ')), ' is'))
```

Dataverse Email activity parties with lookup or email address (‘unresolved recipient’)
(2 = To 1 = From)

```python
[
  {   
    "participationtypemask": 2,    
    "partyid@odata.bind": "systemuser(E1AC0A77-C900-EC11-94EF-000D3ACC597E)" 
  },  
  {    
    "participationtypemask": 1,
    "addressused": "mailto:blah@email.com" 
  }
]
```

## Words not GUIDS ##

Expand: Get Contact first name from customer lookup

```python
customerid_contact($select=firstname)
```

Get Choice Label from List Rows Action

```python
item()?['choiceName@OData.Commuity.Display.V1.FormattedValue']

```

Get Choice Labels from Get A Row Action

```python
body('GetARow')?['choiceName@OData.Commuity.Display.V1.FormattedValue']
```

Name of lookup rather than GUID

```python
@OData.Community.Display.V1.FormattedValue
 
e.g. 
 
triggerOutputs()?['body/_customerid_value@OData.Community.Display.V1.FormattedValue'] 
   
  => Joe Bloggs 
        not 82373-43949-3948394-298424
```

Table logical name/type e.g. customer -> contact/account or owner -> team/systemuser

```python
@Microsoft.Dynamics.CRM.lookuplogicalname
 
e.g. 
 
triggerOutputs()?['body/_customerid_value@Microsoft.Dynamics.CRM.lookuplogicalname'] 
 
  => account OR contact
```

Expand: Get email address of owner and the person who last modified the record

```python
owninguser($select=internalemailaddress),modifiedby($select=internalemailaddress)
```

## Filtering ##

Trigger Filter expression

```python
firstname eq 'Mike' and lastname eq 'Ipsum'
```

Filter based on a lookup

```python
_logicalnamehere_value eq guidHere
```

Filter based on a record’s GUID

```python
entityPrimaryKey eq guidHere
```

Filter based on a string

```python
String inside single quotes, e.g.
 
firstname eq 'Mike'
```

Filter based on an integer (e.g. an option set)

```python
Integer without single quotes, e.g. 
 
preferredcontactmethodcode eq 1
```
ReportLevelFilters for filtering Power BI Reports by a Table/Column with no spaces in Table/Column name

```python
Table/ColumnNames without a space in the name, e.g. 
 
Reseller/Country-Region eq '@{items('Foreach')?['Country']}
```
ReportLevelFilters for filtering Power BI Reports by a Table/Column with spaces in Table/Column name

```python
Table/ColumnNames with a space in the name, e.g. 
 
Reseller/Country_x0020_Region eq '@{items('Foreach')?['Country']}'
```

OData filter for lookups – send a notification, about record update/assign, so long as the owner is not the same as the person who did it

```python
( _ownerid_value ne  _modifiedby_value)
( _ownerid_value ne _modifiedonbehalfby_value)
```
## [How to use Regex in Power Automate](https://tachytelic.net/2021/04/power-automate-regex/) ##

### Regex Match
Office script accepts three parameters:
- The string to perform the regex match on.
- The regex pattern to use.
- The flags to pass to the regex object.
    
```javascript
{
    "version": "0.2.0",
    "body": "function main\n(\n  workbook: ExcelScript.Workbook,\n  searchString: string, \n  regexPattern: string, \n  regexFlags: string\n) : Array<string> {\n  let matches: Array<string> = []\n  let re = new RegExp(regexPattern, regexFlags);\n  let matchArray: Array<string> = searchString.match(re);\n  if (matchArray) {\n    for (var i = 0; i < matchArray.length; i++) {\n      matches.push(matchArray[i]);\n    }\n  }\n  return matches;\n}",
    "description": "",
    "parameterInfo": "{\"originalParameterOrder\":[{\"name\":\"searchString\",\"index\":0},{\"name\":\"regexPattern\",\"index\":1},{\"name\":\"regexFlags\",\"index\":2}],\"parameterSchema\":{\"type\":\"object\",\"required\":[\"searchString\",\"regexPattern\",\"regexFlags\"],\"properties\":{\"searchString\":{\"type\":\"string\"},\"regexPattern\":{\"type\":\"string\"},\"regexFlags\":{\"type\":\"string\"}}},\"returnSchema\":{\"type\":\"object\",\"properties\":{\"result\":{\"type\":\"array\",\"items\":{\"type\":\"string\"}}}}}",
    "apiInfo": "{\"variant\":\"synchronous\",\"variantVersion\":1}"
}
```
### Regex Substitute
Office script accepts four parameters:
- The string to perform the regex substitute on.
- The regex pattern to use find.
- The flags to pass to the regex object.
- What matches will be replaced with

```javascript
{
    "version": "0.2.0",
    "body": "function main\n  (\n    workbook: ExcelScript.Workbook,\n    searchString: string,\n    regexPattern: string,\n    regexFlags: string,\n    replaceString?: string,\n  ) : string {\n  if (typeof (replaceString) === 'undefined') { replaceString = ''; }  \n  let re = new RegExp(regexPattern, regexFlags)\n  return searchString.replace(re, replaceString)\n}",
    "description": "",
    "parameterInfo": "{\"originalParameterOrder\":[{\"name\":\"searchString\",\"index\":0},{\"name\":\"regexPattern\",\"index\":1},{\"name\":\"regexFlags\",\"index\":2},{\"name\":\"replaceString\",\"index\":3}],\"parameterSchema\":{\"type\":\"object\",\"required\":[\"searchString\",\"regexPattern\",\"regexFlags\"],\"properties\":{\"searchString\":{\"type\":\"string\"},\"regexPattern\":{\"type\":\"string\"},\"regexFlags\":{\"type\":\"string\"},\"replaceString\":{\"type\":\"string\"}}},\"returnSchema\":{\"type\":\"object\",\"properties\":{\"result\":{\"type\":\"string\"}}}}",
    "apiInfo": "{\"variant\":\"synchronous\",\"variantVersion\":1}"
}
```
### [Flatten DLP JSON - Array of Arrays into a Flat Array](https://johnliu.net/blog/2025/3/flattening-an-array-of-arrays-in-power-automate)
This Office script takes a JSON string as input (representing the Power Platform DLP policies), flattens its structure, and returns an array of simplified objects.

```javascript
interface src {
  displayName: string;
  connectorGroups: {
    classification: string;
    connectors: {
      id: string;
      name: string;
      type: string;
    }[];
  }[];
}

interface tgt {
  policyName: string;
  classification: string;
  id: string;
  name: string;
  type: string;
}

function main(workbook: ExcelScript.Workbook, dlpValue: string): tgt[] {
  let dlp: src[] = JSON.parse(dlpValue);
  return flattenDLPJson(dlp);
}

function flattenDLPJson(dlp: src[]): tgt[] {
  let flattenedList: tgt[] = [];
  dlp.forEach(policy => {
    let policyName = policy.displayName;
    policy.connectorGroups.forEach(policyGroup => {
      let classification = policyGroup.classification;
      policyGroup.connectors.forEach(connector => {
        let flattenedItem: tgt = {
          policyName: policyName,
          classification: classification,
          id: connector.id,
          name: connector.name,
          type: connector.type
        };
        flattenedList.push(flattenedItem);
      });
    });
  });
  return flattenedList;
}
```
### Regex Patterns

[Regular Expression to Find Email Addresses](https://www.regular-expressions.info/email.html) with gi flag
```python
\b[A-Z0-9._%+-]+@[A-Z0-9.-]+.[A-Z]{2,}\b
```

[Regular Expression to Validate Credit Card Numbers](https://www.regular-expressions.info/creditcard.html) with gi flag
```python
^(?:4[0-9]{12}(?:[0-9]{3})?          # Visa
 |  (?:5[1-5][0-9]{2}                # MasterCard
     | 222[1-9]|22[3-9][0-9]|2[3-6][0-9]{2}|27[01][0-9]|2720)[0-9]{12}
 |  3[47][0-9]{13}                   # American Express
 |  3(?:0[0-5]|[68][0-9])[0-9]{11}   # Diners Club
 |  6(?:011|5[0-9]{2})[0-9]{12}      # Discover
 |  (?:2131|1800|35\d{3})\d{11}      # JCB
)$
```

[Regular Expression to Trim Extra Spaces within a String](https://tachytelic.net/2021/04/power-automate-regex/) with g flag
```python
\s\s+
```

[Regular Expression to match a date in mm/dd/yyyy format](https://www.regular-expressions.info/dates.html) with g flag
```python
^(0[1-9]|1[012])[- /.](0[1-9]|[12][0-9]|3[01])[- /.](19|20)\d\d$
```

[Regular Expression to match a date in dd-mm-yyyy format](https://www.regular-expressions.info/dates.html) with g flag
```python
^(0[1-9]|[12][0-9]|3[01])[- /.](0[1-9]|1[012])[- /.](19|20)\d\d$
```
