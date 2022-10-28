Code To Run
======

```
# Install latest Strategus
remotes::install_github("ohdsi/Strategus")


# load Strategus
library(Strategus)
library(dplyr)

# Inputs to run (edit these for your CDM):
# ========================================= #

database <- 'databaseName'
connectionDetailsReference <- "databaseName"
outputFolder <- '/Users/username/Documents/saveLocation'
cdmDatabaseSchema <- keyring::key_get('cdmDatabaseSchema', database)
workDatabaseSchema <- "schema where you can read/write to"
cohortTable <- "plp_pca"
  
connectionDetails <- DatabaseConnector::createConnectionDetails(
  dbms = keyring::key_get('dbms', database),
  server = keyring::key_get('server', database),
  user = keyring::key_get('user', database),
  password = keyring::key_get('pw', database),
  port = keyring::key_get('port', database)
)

# end inputs

storeConnectionDetails(connectionDetails = connectionDetails,
                       connectionDetailsReference = connectionDetailsReference)

executionSettings <- createCdmExecutionSettings(
  connectionDetailsReference = connectionDetailsReference,
  workDatabaseSchema = workDatabaseSchema,
  cdmDatabaseSchema = cdmDatabaseSchema,
  cohortTableNames = CohortGenerator::getCohortTableNames(cohortTable = cohortTable),
  workFolder = file.path(outputFolder, "strategusWork"),
  resultsFolder = file.path(outputFolder, "strategusOutput"),
  minCellCount = 5
)

# Note: this environmental variable should be set once for each compute node
Sys.setenv("INSTANTIATED_MODULES_FOLDER" = file.path(outputFolder,"StrategusInstantiatedModules"))

url <- "https://raw.githubusercontent.com/yazhilin/plp-pca/main/inst/analysisSpecifications.json"
json <- readLines(file(url))
json2 <- paste(json, collaplse = '\n')
analysisSpecifications <- ParallelLogger::convertJsonToSettings(json2)

execute(
  analysisSpecifications = analysisSpecifications,
  executionSettings = executionSettings,
  executionScriptFolder = file.path(outputFolder,"strategusExecution")
)
```

After running this code, you'll find a results.zip file in the "strategusOutput" > "PatientLevelPredictionValidationModule_3" subfolders in the location you specified as your "outputFolder".

Inspect the zipped folder contents (should be a set of csv files with the performance settings and results) and once happy with the content, send the results.zip to the study administrator.
