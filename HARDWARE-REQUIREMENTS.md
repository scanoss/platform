# SCANOSS Platform Hardware Requirements

The document contains details of the hardware requirements for running the SCANOSS platform on-premise.

## Engine/LDB

The following is recommended for running the SCANOSS Engine and OSSKB (LDB):

| Component | Size                      |
|-----------|---------------------------|
| CPU       | 8 Core x64                |
| RAM       | 32GB                      |
| HDD       | 10TB SSD (NVMe preferred) |


## API/Load Balancer

The following is recommended for running the SCANOSS API and Load Balancer:

| Component | Size                      |
|-----------|---------------------------|
| CPU       | 4 Core x64                |
| RAM       | 16GB                      |
| HDD       | 100GB                     |


API/LB need to be in the same subnet as the Engine/LDB.

## Minr

The following is recommended for running the SCANOSS Mining software:

| Component | Size                      |
|-----------|---------------------------|
| CPU       | 4 Core x64                |
| RAM       | 32GB                      |
| HDD       | 2TB SSD (NVMe preferred) |


Note: The Minr servers are optional, only required if local mining is being conducted.
