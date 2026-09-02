# Green computing

As part of efficient use of computing resources, and accountability for the environmental impact of computing, it is possible to measure the carbon footprint of heavy compute operations.
General recommendations on this are provided by the Green Algorithms initiative, see the link below for more information.
Although there are currently many tools to measure carbon footprint of compute, we explore the tools provided by the Cambridge Sustainable group.

We use the Green Algorithms for High Performance Computing (**GA4HPC**) software, as this is easily deployable on HPC clusters, supports SLURM, and can be retrospectively used without the need for pre-installed software or other components running on the cluster.

As a test case, we use [muvis-align](muvis-align.md), software for multi-view registration of microscopy images, and we measure the carbon footprint of the registration of a large dataset.

HPC hardware setup:
* CPU nodes: Dell PowerEdge R6525
* Two 2nd/3rd Generation AMD EPYCTM Processors with 64 cores per processor
* 280W for the CPU nodes processors (for 64 cores)

Power consumption / carbon factors:
* Power Usage Effectiveness (PUE): 1.12
* Average carbon intensity: 137 gCO2e/kWh


Dataset and operation:
* Operation: X/Y stitching of 2D tiles: registration and fusion including export of registration information and fused image data in OME-Zarr format
* Source data: 64 tiles (OME-Zarr format) of 6400 x 6400 x 16 bit = ~5 GB
* Resulting fused data: ~ 50500 x 50500 x 16 bit = ~5 GB

GA4HPC results:
* Energy used: 0.19 kWh
* CPU: 0 days 01:00:19 (1 hours)
* Total wallclock time: 0 days 00:41:13
* 26 gCO2e


Dataset and operation:
* Operation: X/Y stitching of 3D tiles: registration and fusion including export of registration information and fused image data in OME-Zarr format
* Source data: 4 tiles (OME-Zarr format) of 2048 x 2048 x 200 x 16 bit = ~1.7 GB
* Resulting fused data: ~ 4000 x 4000 x 220 x 16 bit = ~7 GB

GA4HPC results:
* Energy used: 0.18 kWh
* CPU: 0 days 00:58:24 (1 hours)
* Total wallclock time: 0 days 00:38:54
* 24 gCO2e


Relevant resources:

* Green Algorithms Initiative, Towards environmentally sustainable computational science: https://www.green-algorithms.org/
* GA4HPC tool: [github.com/Cambridge-Sustainable-Computing-Lab/GreenAlgorithms4HPC](https://github.com/Cambridge-Sustainable-Computing-Lab/GreenAlgorithms4HPC)
* Green DiSC https://www.software.ac.uk/GreenDiSC, https://github.com/Cambridge-Sustainable-Computing-Lab/greenDiSC
