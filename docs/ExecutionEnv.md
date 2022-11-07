# Execution Environment

## Overview

For running production pipelines one shoudl use Docker container environment.

For debugging and experimental work, using Docker containers is inconvenient,
hence, one can use conda environment.

## Running in Docker container
  
### general notes on running pipelines in a docker container 

For production mode and for running CWL pipelines using either
Airflow user interface or command line, the best way is to run everything
in docker container.

See [Testing documentation](https://nsaph-data-platform.github.io/nsaph-platform-docs/common/platform-deployment/doc/Testing.html)
for more information on running your code inside CWL-Airflow container.

### Mapping between directories on NSAPH host and in the docker container
         
Mapping between host directories and paths within containers are defined in
[docker-compose.yaml](https://github.com/NSAPH-Data-Platform/nsaph-platform-deployment/blob/master/docker-compose.yaml)

The specific line that maps the path to output files to the docker is
[rundir mapping](https://github.com/NSAPH-Data-Platform/nsaph-platform-deployment/blob/1b652334782706a341bf2eafe8c98c91a2847361/docker-compose.yaml#L66)

```yaml
- /scratch/cwl/rundir:/opt/airflow/cwl_rundir
```

In other words, the container path `/opt/airflow/cwl_rundir` is mapped to
the host path `/scratch/cwl/rundir`. 

To simplify running pipelines in a container I use a script like the following
(for running Medicare processing):

```shell
#!/bin/bash 
dd=`date +%Y-%m-%d-%H-%M`
reldir=cms/medicare/add2018/${dd}
rundir=/opt/airflow/cwl_rundir/$reldir
echo /scratch/cwl/rundir/$reldir
docker exec  webserver bash -c "source /root/anaconda/etc/profile.d/conda.sh && conda activate nsaph && mkdir -p ${rundir} && cd ${rundir} && cwl-runner --leave-tmpdir  /opt/airflow/project/cms/src/cwl/medicare.cwl --database /opt/airflow/project/database.ini --connection_name nsaph2 --input /data/incoming/rce/ci3_d_medicare/original_data/cms_medicare/data/11836"
cp -R /scratch/cwl/rundir/$reldir .
```
   
I usually run this script with the following command:

```shell
nohup ./run_medicare.sh 2>&1 >  1medicare.log &
```

This script:
1. Prints the host path to output directory before starting the pipeline
2. Copies all outputs to the current directory after the pipeline has completed.

> Note: you might want to add `--leave-tmpdir` option to `cwl-runner` 
> command as in the script example above. This command preserves all
> intermediate outputs even in a case when the pipeline has failed to
> complete and the outputs have not been copied. Look for the paths
> to specific output files in the log file, like `1medicare.log` in
> the example above
 
## Using Airflow 

It is possible to run pipelines using Airflow. If you have used Airflow
to run a pipeline, the outputs will be in `CWL_OUTPUTS_FOLDER` and intermediate
results in `CWL_TMP_FOLDER`. These folders are mapped to the host in the
[docker-compose.yaml](https://github.com/NSAPH-Data-Platform/nsaph-platform-deployment/blob/master/docker-compose.yaml)

Mapping: [specific lines](https://github.com/NSAPH-Data-Platform/nsaph-platform-deployment/blob/1b652334782706a341bf2eafe8c98c91a2847361/docker-compose.yaml#L59-L63)

```yaml
- ${LOGS_DIR:-./airflow-logs}:/opt/airflow/logs
- ${CWL_TMP_FOLDER:-./cwl_tmp_folder}:/opt/airflow/cwl_tmp_folder
- ${CWL_INPUTS_FOLDER:-./cwl_inputs_folder}:/opt/airflow/cwl_inputs_folder
- ${CWL_OUTPUTS_FOLDER:-./cwl_outputs_folder}:/opt/airflow/cwl_outputs_folder
- ${CWL_PICKLE_FOLDER:-./cwl_pickle_folder}:/opt/airflow/cwl_pickle_folder
```

## Conda environment

Use 
    
    conda activate /opt/anaconda3/envs/nsaph

to activate NSAPH conda environment on NSAPH host.

The environment YAML file is located in:
                                       
    nsaph.rc.fas.harvard.edu:/data/environments/nsaph-2022-10-11.yml    
                
