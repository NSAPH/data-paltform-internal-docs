# Execution Environment

## Overview

For running production pipelines one shoudl use Docker container environment.

For debugging and experimental work, using Docker containers is inconvenient,
hence, one can use conda environment.

## Running in Docker container

For production mode and for running CWL pipelines using either
Airflow user interface or command line, the best way is to run everything
in docker container.

See [Testing documentation](https://nsaph-data-platform.github.io/nsaph-platform-docs/common/platform-deployment/doc/Testing.html)
for more information on running your code inside CWL-Airflow container.

## Conda environment

Use 
    
    conda activate /opt/anaconda3/envs/nsaph

to activate NSAPH conda environment on NSAPH host.

The environment YAML file is located in:
                                       
    nsaph.rc.fas.harvard.edu:/data/environments/nsaph-2022-10-11.yml    
                
