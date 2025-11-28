## Tools for Hammercloud stress tests 


In this repo we collect recipes, scripts, notebooks for Hammercloud stress tests.
In order to test and evaluate the performance of NHR sites (HPC centers in DE) compared to standard T2 sites we have put together an HC test template which executes a simple analysis job onphyslite DAOD data. It is an IO heavy application which should be reasonably representative for analysis or derivation tasks in ATLAS.

We have prepared a dedicated dataset for the tests: `user.gduck:data24_13p6TeV.physics_Main.deriv.DAOD_PHYSLITE.NHR_test_1` which had been replicated to several RSEs in DE:
```
rucio list-dataset-replicas user.gduck:data24_13p6TeV.physics_Main.deriv.DAOD_PHYSLITE.NHR_test_1

DATASET: user.gduck:data24_13p6TeV.physics_Main.deriv.DAOD_PHYSLITE.NHR_test_1
+------------------------+---------+---------+
| RSE                    |   FOUND |   TOTAL |
|------------------------+---------+---------|
| FZK-LCG2_DATADISK      |    6748 |    6748 |
| UNI-FREIBURG_DATADISK  |    6748 |    6748 |
| DESY-HH_DATADISK       |    6748 |    6748 |
| LRZ-LMU_LOCALGROUPDISK |    6748 |    6748 |
| GOEGRID_DATADISK       |    6748 |    6748 |
+------------------------+---------+---------+
```
For the analysis of these tests there are several options available
* web page for HC test
* Open-search dashboard for jobs in test
* extraction of parameters from pandalogs and jupyter analysis notebook
* extraction of parameters from dCache billing logs and jupyter analysis notebook

### HC test start and monitoring
* HC Stress tests are listed [here](https://hammercloud.cern.ch/hc/app/atlas/testlist/stress/)
* [Instructions to start HC stress tests](https://docs.google.com/document/d/1RGQLAuJKtwqxY_Al6vYl1594OqKxzDqwYd4zn-jPhXo/edit?tab=t.0#heading=h.gyjfhbyjm5a)
* Example of a recent [test at LRZ-LMU](https://hammercloud.cern.ch/hc/app/atlas/test/20325020/)
* We keep track of major tests in **[shared table](https://cernbox.cern.ch/s/Kbn79hM2P1rAeva)**
  * *Please update!*  

### Open-search monitoring
* Sebastian provided a dashboard based on grafana/es entries with useful monitoring plots, see [example](https://os-atlas.cern.ch/dashboards/goto/7bf91b8a4328a6f7d1056172b7a80d89?security_tenant=global)
* to get that dashboard for specific test you have to set suitable timerange and set filter to point it to corresponding test, e.g.
field `destinationdblock.keyword` must be set to e.g. `hc_test.gangarbt.hc20322876.tid1313.LRZ-LMU.1`
  * *Watch out to remove blanks before/after string!*
* You can find the field for your test by looking for that parameter in a bigpanda joblog from that test
  * (and bigpanda joblog  you find from corresponding HC test page ([example](https://hammercloud.cern.ch/hc/app/atlas/test/20325020/)) -> Jobs -> Click `monitoring` for some job)
* **watch out: joblogs are only kept for ~4 weeks**

### notebook for pandajoblog analysis
* a much more flexible analysis can be done by extracting panda joblog data in json format and process it with jupyter notebook `AnalyzePandaJobpar.ipynb` 
* to produce json dump one can use `curl` to download:
  ```
  curl --capath $X509_CERT_DIR --cacert $X509_USER_PROXY --cert $X509_USER_PROXY --key $X509_USER_PROXY  -o qjlog.json \\
  'https://bigpanda.cern.ch/jobs/?username=gangarbt&computingsite=LRZ-LMU&destinationdblock=*tid1313*&date_from=2025-11-10T11:31&date_to=2025-11-10T22:31&json'
  ```
* in the query string adjust entries for `date_from, date_to, computingsite, destinationdblock` to your test
* you get the `$X509_` environment variables after ATLAS setup and `lsetup rucio`

### notebook for dCache billing log analysis
* in case data was read from dCache RSE one can analyze the individual file transfers with jupyter notebook `AnalyzeTransfers.ipynb`
* However, for that one needs admin access to the dCache system to  extract information from the dCache `billing logs`.
* Here example command used at LRZ-LMU to process billing log json file:
  ```
  cat /var/lib/dcache/billing/2025/11/billing-2025.11.17 | egrep 'DAOD_PHYSLITE.4363|DAOD_PHYSLITE.4193' | \\
  grep '"msgType":"transfer"' | egrep '"date":"2025-11-17T1[0123]' > billing-2025.11.17_DAOD_PHYSLITE.4363_4193.json
  ```
* dCache has no information about panda job params, so one has to do  `grepping` of matching transfer entries  based on patterns for file-name and time of test


  
