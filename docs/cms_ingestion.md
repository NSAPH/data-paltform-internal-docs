# Ingestion guide: Loading data into the `cms` and `medicaid/medicare` schemas

It is good practice to test the medicare pipeline when new CMS data is going to be added into the main DB. Testing requires a sandbox DB that contains a small subset of data from the main DB.

Run the following SQL query in `superset` to print the number of entries in the sandbox DB.

```
select 
    public.count_rows('medicare', 'beneficiaries') As NumBeneficiaries,
    public.count_rows('medicare', 'enrollments') As CountEnrollments,
    public.get_year('medicare'::varchar, 'enrollments'::varchar) As EnrollmentYears,
    public.count_rows('medicare', 'admissions') As NumAdmissions,
    public.count_rows('medicare_audit', 'admissions') As InvalidAdmissions,
    public.get_year('medicare'::varchar, 'admissions'::varchar) As AdmissionsYears,
    public.get_year('medicare_audit'::varchar, 'admissions'::varchar) InvalidAdmissionYears;
```

## Testing the medicare pipeline

### Extracting a sample of the CMS data

The first steps is to extract a sample of the new data using the NSAPH utilities. Access the `nsaph host` in FASSE and create a folder `$HOME/testing_ingestion` to do the tesing. Inside the `HOME/testing_ingestion` folder create a `test_data` folder. Activate the `nsaph` environment.

```
cd $HOME/testing_ingestion/test_data
conda activate nsaph
```

Identify the location of the new data. It should be stored in a subdirectory of `/n/dominici_nsaph_l3/Lab/data/ci3_d_medicare/original_data/cms_medicare/data/yyyy`. Use the `cms.tools.mcr_create_test_data` utility to create the sample. Note: you can only create the test data if there are .fts and .dat files available. For example, this wouldn't work for 2010 data because neither .fts or .dat files exist for that year.

```
python -u -m cms.tools.mcr_create_test_data --in /n/dominici_nsaph_l3/Lab/data/ci3_d_medicare/original_data/cms_medicare/data/2018 --out . --selector 0.0005
```

In this case, a sample of 0.05% is going to be created in `$HOME/testing_ingestion/test_data`.

### Loading the CMS sample data into the sanbox DB

The next step is to execute the `medicare.cwl` pipeline using the sample data. 

The `medicare.cwl` pipeline needs to be executed inside a docker container named `webserver`. To validate the existence of the container in the `nsaph host` use.

```
docker ps
```

You should see an `myairflow-conda` image named `webserver` in the printed list. If `webserver` container is not in the list, accesss has to be granted by the owner. (A note for the owner: you can grant access with `usermod -a -G docker <username>`)

> The code that is used to create the conda image is in the [/NSAPH-Data-Platform/nsaph-platform-deployment](https://github.com/NSAPH-Data-Platform/nsaph-platform-deployment) repository. The instructions to build the docker container are in the [Data Platform Internals](https://nsaph-data-platform.github.io/nsaph-platform-docs/common/platform-deployment/doc/index.html) section of the Data Platform documentation.

Create a bash file `ingest_medicare.sh` and copy the code below.

```
#!/bin/bash 
dd=`date +%Y-%m-%d-%H-%M`
reldir=sandbox/medicare/add2018/${dd}
rundir=/opt/airflow/cwl_rundir/$reldir
echo /scratch/cwl/rundir/$reldir
docker exec  webserver bash -c "source /root/anaconda/etc/profile.d/conda.sh && conda activate nsaph && mkdir -p ${rundir} && cd ${rundir} && cwl-runner --debug  --leave-tmpdir  /opt/airflow/project/cms/src/cwl/medicare.cwl --database /opt/airflow/project/sandbox.ini --connection_name sandbox --input $HOME/testing_ingestion/test_data"
```

Remember that a container has an independent file structure from the `nsaph host`. `rundir` concatenated with `reldir` is a directory inside the docker container. `reldir` has a mapping to the `nsaph host`. The `sandbox.ini` file is stored inside the container to connect to the database. The input directory is mapped to the `nsaph host`.

Run the bash file. This might take a while to run. Therefore, you can either go with the below command,

```
bash ingest_medicare.sh
```

or,

```
nohup ./ingest_medicare.sh 2>&1 >  1medicare.log &
```

Once the pipeline finishes you should see message __Final process status is success__ in either terminal or the log file depending on which command you chose.

If the format of the new data satisfies the properties that are defined in the pipeline. The pipeline should run smoothly. Once the process is complete, check the final number of entries in the sandbox DB using the query that is included in the beginning of the ingestion guide. The number of entries should have increased. Inconsistensies in column names or data types in the new data are examples of issues that may be encountered during the test.

For docker related documentation, please see:
* https://nsaph-data-platform.github.io/nsaph-platform-docs/common/platform-deployment/doc/UsefulCommands.html
* https://github.com/NSAPH/data-paltform-internal-docs/blob/master/docs/ExecutionEnv.md#mapping-between-directories-on-nsaph-host-and-in-the-docker-container

<!-- ### Fixing column name/data type issues -->

## Running the pipeline

Once the test is completed successfully, the `medicare.cwl` pipeline can be used to load new data into the main DB.

The docker container that is housed in the `nsaph host` does not have direct access to the FASSE file system. As such, the raw data has to be copied from FASSE into the `nsaph host`. You can place it in the temporary folder of the `nsaph host` file system `/tmp/cms_data/yyyy`.

Create a folder `$HOME/ingestion/` and a bash script `ingest_medicare.sh` inside that folder with the following content.

```
#!/bin/bash 
dd=`date +%Y-%m-%d-%H-%M`
reldir=cms/medicare/add2018/${dd}
rundir=/opt/airflow/cwl_rundir/$reldir
echo /scratch/cwl/rundir/$reldir
docker exec  webserver bash -c "source /root/anaconda/etc/profile.d/conda.sh && conda activate nsaph && mkdir -p ${rundir} && cd ${rundir} && cwl-runner --leave-tmpdir  /opt/airflow/project/cms/src/cwl/medicare.cwl --database /opt/airflow/project/database.ini --connection_name nsaph2 --input /tmp/cms_data/yyyy"
```

Run the bash file.

```
bash ingest_medicare.sh
```

Happy data loading!
