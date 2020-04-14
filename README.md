# ClickHouse Replication

![banner](banner.jpg)

Replication is only supported for tables in the `MergeTree` family. `MergeTree` is the main engine in [ClickHouse](https://clickhouse.tech). It allows to store, query data on one server and experience all the advantages of ClickHouse without requiring any special configuration.‌‌

However, data is not fault-tolerance, do not improve reliability or accessibility on one server. Maybe, a server has issues(downtime, network, etc). That’ why, the system need to have the replica(s). It means another server(s) have a copy data which can 'substitute' the data of original server any moment.

One of ClickHouse's features uses `ReplicatedMergeTree` engine to have an extra copy(replica) of data. The system condition is running [ZooKeeper](https://zookeeper.apache.org/).‌‌

## Install

```bash
docker-compose up -d
```

## Test

You can test your cluster by creating a simple table :

#### Connect using ClickHouse client

```bash
docker exec -it CH-Replica-client /usr/bin/clickhouse-client --host server-1
```

#### Create sample table

```sql
CREATE TABLE ontime
(
    Year UInt16,
    Quarter UInt8,
    Month UInt8,
    DayofMonth UInt8,
    DayOfWeek UInt8,
    FlightDate Date,
    UniqueCarrier FixedString(7),
    AirlineID Int32,
    Carrier FixedString(2),
    TailNum String,
    FlightNum String,
    OriginAirportID Int32,
    OriginAirportSeqID Int32,
    OriginCityMarketID Int32,
    Origin FixedString(5),
    OriginCityName String,
    OriginState FixedString(2),
    OriginStateFips String,
    OriginStateName String,
    OriginWac Int32,
    DestAirportID Int32,
    DestAirportSeqID Int32,
    DestCityMarketID Int32,
    Dest FixedString(5),
    DestCityName String,
    DestState FixedString(2),
    DestStateFips String,
    DestStateName String,
    DestWac Int32,
    CRSDepTime Int32,
    DepTime Int32,
    DepDelay Int32,
    DepDelayMinutes Int32,
    DepDel15 Int32,
    DepartureDelayGroups String,
    DepTimeBlk String,
    TaxiOut Int32,
    WheelsOff Int32,
    WheelsOn Int32,
    TaxiIn Int32,
    CRSArrTime Int32,
    ArrTime Int32,
    ArrDelay Int32,
    ArrDelayMinutes Int32,
    ArrDel15 Int32,
    ArrivalDelayGroups Int32,
    ArrTimeBlk String,
    Cancelled UInt8,
    CancellationCode FixedString(1),
    Diverted UInt8,
    CRSElapsedTime Int32,
    ActualElapsedTime Int32,
    AirTime Int32,
    Flights Int32,
    Distance Int32,
    DistanceGroup UInt8,
    CarrierDelay Int32,
    WeatherDelay Int32,
    NASDelay Int32,
    SecurityDelay Int32,
    LateAircraftDelay Int32,
    FirstDepTime String,
    TotalAddGTime String,
    LongestAddGTime String,
    DivAirportLandings String,
    DivReachedDest String,
    DivActualElapsedTime String,
    DivArrDelay String,
    DivDistance String,
    Div1Airport String,
    Div1AirportID Int32,
    Div1AirportSeqID Int32,
    Div1WheelsOn String,
    Div1TotalGTime String,
    Div1LongestGTime String,
    Div1WheelsOff String,
    Div1TailNum String,
    Div2Airport String,
    Div2AirportID Int32,
    Div2AirportSeqID Int32,
    Div2WheelsOn String,
    Div2TotalGTime String,
    Div2LongestGTime String,
    Div2WheelsOff String,
    Div2TailNum String,
    Div3Airport String,
    Div3AirportID Int32,
    Div3AirportSeqID Int32,
    Div3WheelsOn String,
    Div3TotalGTime String,
    Div3LongestGTime String,
    Div3WheelsOff String,
    Div3TailNum String,
    Div4Airport String,
    Div4AirportID Int32,
    Div4AirportSeqID Int32,
    Div4WheelsOn String,
    Div4TotalGTime String,
    Div4LongestGTime String,
    Div4WheelsOff String,
    Div4TailNum String,
    Div5Airport String,
    Div5AirportID Int32,
    Div5AirportSeqID Int32,
    Div5WheelsOn String,
    Div5TotalGTime String,
    Div5LongestGTime String,
    Div5WheelsOff String,
    Div5TailNum String
)
ENGINE = MergeTree(FlightDate, (Year, FlightDate), 8192);
```

#### Import data

Download [data](https://yadi.sk/d/pOZxpa42sDdgm) and run :

```bash
xz -v -c -d < ~./ontime.csv.xz | docker exec -i CH-Replica-client /usr/bin/clickhouse-client --host server-1 --query="INSERT INTO ontime FORMAT CSV"
```

## Production

- For tests, it can use one standalone ZooKeeper instance.‌‌
- For production use, it should have ZooKeeper ensemble **at least of 3 servers**. And tips to deployment is **not run ZooKeeper on the same servers as ClickHouse**. Because ZooKeeper is very sensitive for latency and ClickHouse may utilize all available system resources.‌‌
