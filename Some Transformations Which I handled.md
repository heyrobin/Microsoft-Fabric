

### Following is for searching date time in table from pipelines

```sql
@concat('SELECT * FROM ', item().table_name, ' WHERE ', item().column_name, ' > ''',
 formatDateTime(item().watermark_value, 'yyyy-MM-dd HH:mm:ss'), '''')
```
