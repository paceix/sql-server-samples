# Loading Instructions

### SQL Server Via WSL

Prepare Data Dir
```bash
export VOLUMEDIR=/home/kishan/dev/sqlserver/datavolume
sudo rm -rf $VOLUMEDIR && mkdir -p $VOLUMEDIR
sudo chown -R 10001:10001 $VOLUMEDIR
sudo chmod -R 755 $VOLUMEDIR
```

Build mssql with Full Text Search using below dockerfile 
```bash
docker build -t mssql-with-fts .
docker run -v /mnt/c/Users/kisha/windev/Power\ BI/sql-server-samples/:/sql-server-samples:ro -v /home/kishan/dev/sqlserver/datavolume:/var/opt/mssql -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=SecurePassword@33" -e "MSSQL_PID=Developer" -p 1433:1433  --name sqlpreview --hostname sqlpreview -d mssql-with-fts
```

NOTE: don't use this docker image in production, this duplicates the server install as installed from deb and also in base
```Dockerfile
FROM mcr.microsoft.com/mssql/server:2022-latest

USER root

# Install Full-Text Search
RUN apt-get update -y && \
    apt-get install -y curl gnupg2 && \
    curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor -o /usr/share/keyrings/microsoft-prod.gpg && \
    curl -fsSL https://packages.microsoft.com/config/ubuntu/22.04/mssql-server-2022.list | tee /etc/apt/sources.list.d/mssql-server-2022.list && \
    apt-get update -y && \
    apt-get install -y mssql-server-fts && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER mssql
```

Logging in:
- For with password: `sqlcmd -U sa -P SecurePassword@33`
- For empty password `sqlcmd -U sa` and press enter on password prompt

## Adventure Data Works
- Open SSMS copy, create a new query, Activate SQLCMD mode from Query options (top bar)
- Copy and execute the scripts `instawdb.sql` and `instawdbdw.sql`

## Wide World Importers
### Prerequsites
- Visual Studio with SQL components
- WSL2 + Docker setup
- Install [SSIS Extension](https://marketplace.visualstudio.com/items?itemName=SSIS.MicrosoftDataToolsIntegrationServices) in Visual Studio, restart if needed

### Modifying Data Simulation

Edit `Script.PostDeployment1.sql` to update the start and end date at dataloadsimulation procedure
```sql
EXEC DataLoadSimulation.DailyProcessToCreateHistory
    @StartDate = '20130201',
    @EndDate = '20130301',
```

### Run Instructions
- Run the db creations (ww-ssdt and ww-dw-ssdt) first with password set. Clean / Build / Publish the Solution packages
- Check that data is populated in WideWorldImporters and tables created in DW variant
- Make password blank for ETL to run (not sure why connection managers fail to connect even after asking to save password)
```sql
ALTER LOGIN [sa] WITH CHECK_POLICY=OFF;
GO
ALTER LOGIN [sa] WITH PASSWORD='';
GO
```
- Configure the Connection Managers with valid provider and blank password, choose one of the valid OLE DB Native providers and make sure to set credentials **and catalog**
- Execute the `DailyETLMain.dtsx` from SSIS Packages in Daily ETL, once successfull should populate the data for 2013 feb based on data simulation range set (previous section)