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
  i.Type = t.UUID
  AND
  t.Name = 'Table'
  AND
  i.definition = d.sde_xml_id;
```

In `Oracle` you can use the `XMLType` data type to convert the XML text that can be queried using `XPath` statements.

```sql
SELECT
  EXTRACTVALUE(XMLType(sde.sdexmltotext(d.xml_doc)), '/DEWorkspace/MajorVersion') AS "Major version",
  EXTRACTVALUE(XMLType(sde.sdexmltotext(d.xml_doc)), '/DEWorkspace/MinorVersion') AS "Minor version",
  EXTRACTVALUE(XMLType(sde.sdexmltotext(d.xml_doc)), '/DEWorkspace/BugfixVersion') AS "Bug fix version"
FROM
 sde.gdb_items i,
 sde.gdb_itemtypes t,
 sde.sde_xml_doc1 d
WHERE
  i.Type = t.UUID
  AND
  i.Name = 'Workspace'
  AND
  i.definition = d.sde_xml_id;
```


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
