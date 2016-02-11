# ArcGIS 10 Geodatabase Schema
The ArcGIS 10 Geodatabse Schema was altered so that all of the information related to the schema such as feature classes, domains, subtypes, etc..., that was previously stored in the 36+ geodatabase system tables (the tables prefixed with `GDB_`) were consolidated into four main tables:
- `GDB_ITEMS`
- `GDB_ITEMTYPES`
- `GDB_ITEMRELATIONSHIPTYPES`
- `GDB_ITEMRELATIONSHIPS`

The schema changes require detailed information to be read from blob data that is stored within the geodatabase tables as XML.

There is a unique XML definition for each of the types in `GDB_ITEMSTYPES` table. You can determine the definition by changing the `t.Name = 'Table'` to one of the static values in the table.

The following shows an example of the definitions for the `GDB_ITEMS` of type `Table`.

```sql
SELECT
  i.objectid,
  i.name,
  sde.sdexmltotext(d.xml_doc) AS DEFINITION
FROM
 sde.gdb_items i,
 sde.gdb_itemtypes t,
 sde.sde_xml_doc1 d
WHERE
  i.type = t.uuid
  AND
  t.name = 'Table'
  AND
  i.definition = d.sde_xml_id;
```

The following is an example of the contents of the `sde.sdexmltotext(d.xml_doc) AS DEFINITION` column returned from the above query.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<DETableInfo xsi:type='typens:DETableInfo' xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance' xmlns:xs='http://www.w3.org/2001/XMLSchema' xmlns:typens='http://www.esri.com/schemas/ArcGIS/10.1'>
  <CatalogPath>\SDE.MM_HIDDEN_PACKAGES</CatalogPath>
  <Name>SDE.MM_HIDDEN_PACKAGES</Name>
  <ChildrenExpanded>false</ChildrenExpanded>
  <DatasetType>esriDTTable</DatasetType>
  <DSID>180</DSID>
  <Versioned>false</Versioned>
  <CanVersion>true</CanVersion>
  <ConfigurationKeyword>
  </ConfigurationKeyword>
  <RequiredGeodatabaseClientVersion>10.0</RequiredGeodatabaseClientVersion>
  <HasOID>true</HasOID>
  <OIDFieldName>OBJECTID</OIDFieldName>
  <GPFieldInfoExs xsi:type='typens:ArrayOfGPFieldInfoEx'>
    <GPFieldInfoEx xsi:type='typens:GPFieldInfoEx'>
      <Name>OBJECTID</Name>
      <AliasName>OBJECTID</AliasName>
      <ModelName>OBJECTID</ModelName>
      <FieldType>esriFieldTypeOID</FieldType>
      <IsNullable>false</IsNullable>
      <Required>true</Required>
      <Editable>false</Editable>
    </GPFieldInfoEx>
    <GPFieldInfoEx xsi:type='typens:GPFieldInfoEx'>
      <Name>NAME</Name>
      <AliasName>NAME</AliasName>
      <ModelName>NAME</ModelName>
      <FieldType>esriFieldTypeString</FieldType>
      <DefaultValueString>ARCFM</DefaultValueString>
      <IsNullable>false</IsNullable>
      <Required>true</Required>
    </GPFieldInfoEx>
    <GPFieldInfoEx xsi:type='typens:GPFieldInfoEx'>
      <Name>CATEGORY</Name>
      <AliasName>CATEGORY</AliasName>
      <ModelName>CATEGORY</ModelName>
      <FieldType>esriFieldTypeInteger</FieldType>
      <DefaultValueNumeric>-1</DefaultValueNumeric>
      <IsNullable>false</IsNullable>
      <Required>true</Required>
    </GPFieldInfoEx>
    <GPFieldInfoEx xsi:type='typens:GPFieldInfoEx'>
      <Name>USERNAME</Name>
      <AliasName>USERNAME</AliasName>
      <ModelName>USERNAME</ModelName>
      <FieldType>esriFieldTypeString</FieldType>
      <DefaultValueString>ARCFM</DefaultValueString>
      <IsNullable>false</IsNullable>
      <Required>true</Required>
    </GPFieldInfoEx>
    <GPFieldInfoEx xsi:type='typens:GPFieldInfoEx'>
      <Name>CONTENTS</Name>
      <AliasName>CONTENTS</AliasName>
      <ModelName>CONTENTS</ModelName>
      <FieldType>esriFieldTypeBlob</FieldType>
      <IsNullable>true</IsNullable>
      <Required>true</Required>
    </GPFieldInfoEx>
  </GPFieldInfoExs>
  <CLSID>{EE2B3FD2-8DA8-43E2-AE88-4FFE21FFDC21}</CLSID>
  <EXTCLSID>
  </EXTCLSID>
  <RelationshipClassNames xsi:type='typens:Names'>
  </RelationshipClassNames>
  <AliasName>ArcFM Hidden Packages</AliasName>
  <ModelName>
  </ModelName>
  <HasGlobalID>false</HasGlobalID>
  <GlobalIDFieldName>
  </GlobalIDFieldName>
  <RasterFieldName>
  </RasterFieldName>
  <ExtensionProperties xsi:type='typens:PropertySet'>
    <PropertyArray xsi:type='typens:ArrayOfPropertySetProperty'>
    </PropertyArray>
  </ExtensionProperties>
  <ControllerMemberships xsi:type='typens:ArrayOfControllerMembership'>
  </ControllerMemberships>
  <EditorTrackingEnabled>false</EditorTrackingEnabled>
  <CreatorFieldName>
  </CreatorFieldName>
  <CreatedAtFieldName>
  </CreatedAtFieldName>
  <EditorFieldName>
  </EditorFieldName>
  <EditedAtFieldName>
  </EditedAtFieldName>
  <IsTimeInUTC>true</IsTimeInUTC>
  <ChangeTracked>false</ChangeTracked>
</DETableInfo>
```

In `Oracle` you can use the `XMLType` data type to convert the XML text that can be queried using `XPath` statements.

```sql
SELECT
  EXTRACTVALUE(XMLType(sde.sdexmltotext(d.xml_doc)), '/DETableInfo/Name') AS "Name",
  EXTRACTVALUE(XMLType(sde.sdexmltotext(d.xml_doc)), '/DETableInfo/AliasName') AS "Alias Name"
FROM sde.gdb_items i,
  sde.gdb_itemtypes t,
  sde.sde_xml_doc1 d
WHERE i.type     = t.uuid
AND t.name       = 'Table'
AND i.definition = d.sde_xml_id;
```

The following is an example of the outout of the above `sql`.

Name                       | Alias Name
:------------------------- | :---------------------------
SDE.MM_HIDDEN_PACKAGES     | ArcFM Hidden Packages
SDE.MM_SYSTEM_PACKAGES     | ArcFM System Packages
SDE.MM_PERSIST_INFO        | ArcFM User Information
SDE.MM_SYSTEM_PERSIST_INFO | ArcFM System Information
SDE.MM_SNAPPING            | ArcFM Snapping Configuration

In `.NET` you can read the definition data in a similar manner using the `XDocument` class.

```java
using (OracleConnection connection = new OracleConnection("User Id=sde;Password=sde;Data Source=sid;"))
{
    connection.Open();      

    using(OracleCommand cmd = connection.CreateCommand())
    {
      cmd.CommandText = @"SELECT sde.sdexmltotext(d.xml_doc) AS DEFINITION
                       FROM
                         sde.gdb_items i,
                         sde.gdb_itemtypes t,
                         sde.sde_xml_doc1 d
                        WHERE
                          i.Type = t.UUID
                          AND
                          t.Name = 'Table'
                          AND
                          i.definition = d.sde_xml_id;";

      using (OracleDataReader dr = cmd.ExecuteReader())
      {
          while (dr.Read())
          {
              string def = System.Convert.ToString(dr.GetValue(0));

              XDocument xd = XDocument.Parse(def);
              StringBuilder str = new StringBuilder();

              int i = 1;
              foreach (XElement ele in xd.Descendants("GPFieldInfoEx"))
              {
                  string name = ele.Element("Name").Value;
                  string code = ele.Element("FieldType").Value;
                  str.Append(i+"："+name + "," + code+"/n");
                  i++;
              }          
          }   
      }
    }
}
```
