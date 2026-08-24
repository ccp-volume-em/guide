# FAIR principles

## Findable

Make all project components discoverable by capturing metadata as early as possible in the workflow. Metadata should be recorded not only for image datasets, but also for analyses, workflows, software, protocols, models, and project-level information.

Practical recommendations include:

* Store acquisition metadata directly alongside image data whenever possible.
* Record workflow provenance, including software versions, parameters, and processing steps.
* Assign persistent identifiers to datasets and published outputs.
* Maintain a consistent project structure so that raw data, processed data, analyses, and documentation can be located easily.
* Include metadata describing relationships between datasets, workflows, and derived results.
* Use NGFF OME-Zarr format which includes standardized metadata, also supporting efficient metadata exploration for large datasets.
* Use RO-Crate metadata as a single entry point to datasets, workflows, software, and documentation.

Relevant resources:

* OME-Zarr: [OME-Zarr specification and documentation](https://ngff.openmicroscopy.org/)
* RO-Crate: [RO-Crate specification](https://www.researchobject.org/ro-crate/)

## Accessible

Data and metadata should remain accessible through clearly defined access mechanisms. While not all datasets can be openly shared, metadata should generally remain available.

Implementation guidelines:

* Define access permissions at the project or dataset level.
* Separate access control from metadata availability, allowing users to discover datasets even when data access is restricted.
* Use standard protocols for data access rather than custom solutions.
* Clearly document licensing, embargo periods, and access requirements.
* Preserve metadata even if datasets are moved or archived.

For collaborative projects, role-based access control can provide secure access while maintaining a consistent metadata layer across users and systems.

## Interoperable

Interoperability depends on adopting community standards for both data and metadata.

Recommended practices include:

* Store image data using OME-Zarr whenever possible.
* Use community vocabularies and standardized metadata fields.
* Represent project metadata using RO-Crate.
* Maintain explicit links between datasets, workflows, software, and results.
* Avoid custom metadata formats when existing standards are available.

OME-Zarr enables data exchange between acquisition, visualization, and analysis tools, while RO-Crate provides standardized descriptions of the complete research workflow. Together, these standards facilitate submission to repositories such as the BioImage Archive and support interoperability with external tools and services.

Relevant resources:

* OME-Zarr: [OME-Zarr and NGFF documentation](https://ngff.openmicroscopy.org/)
* RO-Crate: [RO-Crate technical overview](https://www.researchobject.org/ro-crate/technical_overview)
* BioImage Archive: [BioImage Archive](https://www.ebi.ac.uk/bioimage-archive/)

## Reusable

Reusability requires sufficient context to reproduce analyses and interpret results.

Practical measures include:

* Package datasets, workflows, software information, and documentation together.
* Record analysis parameters and software versions.
* Include provenance information describing how outputs were generated.
* Provide clear usage licenses.
* Deposit completed datasets in public repositories when possible.

RO-Crate packaging can be used to bundle all relevant project components, including image data, workflows, protocols, software, and results. At minimum, a package should contain:

* Dataset metadata.
* Workflow descriptions.
* Software and version information.
* Provenance information.
* Licensing information.
* Links between inputs, outputs, and analyses.

Correct RO-Crate packaging improves reproducibility, simplifies data exchange, and supports deposition in public archives.

Relevant resources:

* RO-Crate specification: [RO-Crate specification](https://www.researchobject.org/ro-crate/specification/)
* Workflow RO-Crate: [Workflow RO-Crate profile](https://about.workflowhub.eu/Workflow-RO-Crate/)
* BioImage Archive submission guidance: [OME-Zarr submission guidance for BioImage Archive](https://www.ebi.ac.uk/bioimage-archive/help-zarr/)
