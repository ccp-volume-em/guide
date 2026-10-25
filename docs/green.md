# Green computing

As part of efficient use of computing resources, and accountability for the environmental impact of computing, it is possible to measure the carbon footprint of heavy compute operations.
General recommendations on this are provided by the Green Algorithms initiative, see the link below for more information.
Although there are currently many tools to measure carbon footprint of compute, we explore the tools provided by the Cambridge Sustainable group.

We use the Green Algorithms for High Performance Computing (**GA4HPC**) software, as this is easily deployable on HPC clusters, supports SLURM, and can be retrospectively used without the need for pre-installed software or other components running on the cluster.

## Test cases

As a test case, we use [muvis-align](muvis-align.md), software for multi-view registration of microscopy images, and we measure the carbon footprint of the registration of a large dataset.

HPC hardware setup:

* CPU nodes: Dell PowerEdge R6525
* Two 2nd/3rd Generation AMD EPYCTM Processors with 64 cores per processor
* 280W for the CPU nodes processors (for 64 cores)

Power consumption / carbon factors:

* Power Usage Effectiveness (PUE): 1.12
* Average carbon intensity: 137 gCO2e/kWh

___

Dataset and operation:

* Operation: X/Y stitching of 2D tiles: registration and fusion including export of registration information and fused image data in OME-Zarr format
* Source data: 64 tiles (OME-Zarr format) of 6400 x 6400 x 16 bit = 5 GB
* Resulting fused data: ~ 50500 x 50500 x 16 bit = 5 GB

GA4HPC results:

* Energy used: 0.19 kWh
* CPU: 0 days 01:00:19 (1 hours)
* Total wallclock time: 0 days 00:41:13
* 26 gCO2e

___

Dataset and operation:

* Operation: X/Y stitching of 3D tiles: registration and fusion including export of registration information and fused image data in OME-Zarr format
* Source data: 4 tiles (OME-Zarr format) of 2048 x 2048 x 200 x 16 bit = 1.7 GB
* Resulting fused data: ~ 4000 x 4000 x 220 x 16 bit = 7 GB

GA4HPC results:

* Energy used: 0.18 kWh
* CPU: 0 days 00:58:24 (1 hours)
* Total wallclock time: 0 days 00:38:54
* 24 gCO2e

___

Dataset and operation:

* Operation: stitching of 2D tiles and low resolution overviews: registration and fusion to OME-Zarr format
* Improved efficiency of the registration and fusion process resulting in reduced energy consumption and faster processing times
* Lower node memory
* Source data:
  * 32915 tiles of 4096 x 3072 x 8 bit (10 um pixel size OME-Tiff format)
  * 1081 tiles of 4096 x 3072 x 8 bit (250 um pixel size OME-Tiff format)
* Fused data shape: 169114 x 147520 x 1081 x 8 bit = 24.5 TB
* Resulting multi-size fused data: 950 GB

GA4HPC results:

* Total processing wallclock time: 18.3 hours
* Memory: 640 GB
* Energy used: 7.97 kWh
* CPU: 14 days 10:26:19 (346 hours)
* 1 kgCO2e

GA4HPC results of improved registration and fusion process:

* Total processing wallclock time: 8.5 hours
* Memory: 640 GB
* Energy used: 3.10 kWh
* CPU: 6 days 19:00:31 (163 hours)
* 424 gCO2e

GA4HPC results of improved registration and fusion process & lower memory:

* Total processing wallclock time: 8.1 hours
* Memory: 160 GB
* Energy used: 1.47 kWh
* CPU: 6 days 12:48:00 (157 hours)
* 201 gCO2e

GA4HPC results of further improvements & even lower memory:

* Total processing wallclock time: 8.4 hours
* Memory: 64 GB
* Energy used: 1.34 kWh
* CPU: 7 days 15:29:04 (183 hours)
* 183 gCO2e

___


## Relevant resources:

* Guide for software developers [software.ac.uk/guide/introductory-guide-green-computing-software-developers](https://www.software.ac.uk/guide/introductory-guide-green-computing-software-developers)
* Green Algorithms Initiative, Towards environmentally sustainable computational science: [green-algorithms.org](https://www.green-algorithms.org/)
* GA4HPC tool: [github.com/Cambridge-Sustainable-Computing-Lab/GreenAlgorithms4HPC](https://github.com/Cambridge-Sustainable-Computing-Lab/GreenAlgorithms4HPC)
* Green Digital Sustainability Certification (DiSC) [software.ac.uk/GreenDiSC](https://www.software.ac.uk/GreenDiSC), [github.com/Cambridge-Sustainable-Computing-Lab/greenDiSC](https://github.com/Cambridge-Sustainable-Computing-Lab/greenDiSC)
