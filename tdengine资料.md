```
sudo systemctl start taosd
sudo systemctl status taosd
● taosd.service - TDengine server service
   Loaded: loaded (/etc/systemd/system/taosd.service; enabled; vendor preset: en
   Active: active (running) since Wed 2020-03-04 08:50:17 CST; 15min ago
 Main PID: 10791 (taosd)
    Tasks: 23 (limit: 4915)
   CGroup: /system.slice/taosd.service
           └─10791 /usr/bin/taosd

3月 04 08:50:17 zjf-B85 systemd[1]: Started TDengine server service.
3月 04 08:50:17 zjf-B85 TDengine:[10791]: Starting TDengine service...
3月 04 08:50:17 zjf-B85 TDengine:[10791]: Started TDengine service successfully.
```

```
zjf@zjf-B85:~$ taosdemo
Creating meters super table...
meters created!
Creating 10000 table(s)......
Table(s) created!
Inserting data......
SYNC Insert with 10 connections:
Spent 745.5895 seconds to insert 1000000000 records with 1000 record(s) per request: 1341220.66 records/second
```


    查询超级表下记录总条数：

    taos>select count(*) from test.meters;

    查询10亿条记录的平均值、最大值、最小值等：

    taos>select avg(f1), max(f2), min(f3) from test.meters;

    查询loc="beijing"的记录总条数：

    taos>select count(*) from test.meters where loc="beijing";

    查询areaid=10的所有记录的平均值、最大值、最小值等：

    taos>select avg(f1), max(f2), min(f3) from test.meters where areaid=10;

    对表t10按10s进行平均值、最大值和最小值聚合统计：

    taos>select avg(f1), max(f2), min(f3) from test.t10 interval(10s);
