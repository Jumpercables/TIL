## Delete Multiversioned View

```bat
sdetable -o delete_mv_view -t CUSTOMERINFORMATION -i sde:oracle11g:/;LOCAL=[DATABASE] -u [USERNAME] -p [PASSWORD] -N                                
```

## Create  Multiversioned View

```bat
sdetable -o create_mv_view -T CUSTOMERINFORMATION_MV -t CUSTOMERINFORMATION -i sde:oracle11g:/;LOCAL=[DATABASE] -u [USERNAME] -p [PASSWORD]
sdetable -o grant -t CUSTOMERINFORMATION_MV -U [ROLE] -A SELECT,INSERT,UPDATE,DELETE -i sde:oracle11g:\;LOCAL=[DATABASE] -u [USERNAME] -p [PASSWORD]             

```
