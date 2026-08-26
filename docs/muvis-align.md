[muvis-align](https://github.com/ccp-volume-em/muvis-align) is an image registration package for aligning 2D and 3D images.
The package is implemented as a [napari](https://napari.org/stable/) plugin, notebooks using the package classes, and a command line interface (CLI) for batch processing.
The [napari](https://napari.org/stable/) plugin can also be run as a container on a local machine or on a cloud platform via an interactive web interface.

# muvis-align practices demonstrated in the source code

Repository: [https://github.com/ccp-volume-em/muvis-align](https://github.com/ccp-volume-em/muvis-align)

`muvis-align` demonstrates practical patterns for scalable, reproducible image registration workflows across a [napari](https://napari.org/stable/) plugin, notebooks, CLI execution, batch/HPC use, and containerized interactive access.

---

## Next Generation File Formats (NGFF)

`muvis-align` uses OME-Zarr / NGFF as a primary output format.

OME-Zarr / NGFF writing is delegated to `multiview_stitcher.ngff_utils.write_sim_to_ome_zarr` from [multiview-stitcher](https://pypi.org/project/multiview-stitcher/), including multiscale pyramid generation.

Source: [`src/muvis_align/image/ome_ngff_helper.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/image/ome_ngff_helper.py)

```python
from multiview_stitcher import ngff_utils

from src.muvis_align.constants import default_ome_zarr_version


def save_ome_ngff(filename, data, pyramid_downsample=2, ome_version=default_ome_zarr_version, verbose=False):
    pyramid_downsample_dict = {}
    for dim in data.dims:
        if dim in 'xyz':
            pyramid_downsample_dict[dim] = pyramid_downsample
        else:
            pyramid_downsample_dict[dim] = 1
    ngff_utils.write_sim_to_ome_zarr(data, filename,
                                     downscale_factors_per_spatial_dim=pyramid_downsample_dict,
                                     ngff_version=ome_version,
                                     overwrite=True,
                                     show_progressbar=verbose)
```

---

## Handling larger-than-memory data

The code uses [Dask](https://www.dask.org/)-backed image sources for lazy loading and chunked processing.

Source: [`src/muvis_align/image/source_helper.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/image/source_helper.py)

```python
def create_dask_source(filename, source_metadata=None):
    ext = os.path.splitext(filename)[1].lstrip('.').lower()
    if ext.startswith('tif'):
        dask_source = TiffDaskSource(filename, source_metadata)
    elif '.zar' in filename.lower():
        dask_source = ZarrDaskSource(filename, source_metadata)
    else:
        raise ValueError(f'Unsupported file type: {ext}')
    return dask_source
```

TIFF files are read lazily through [`dask.delayed`](https://docs.dask.org/en/stable/delayed.html), avoiding immediate full-volume loading.

Source: [`src/muvis_align/image/TiffDaskSource.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/image/TiffDaskSource.py)

```python
def get_data(self, level=0):
    if level < 0:
        dask_data = []
        for level in range(len(self.shapes)):
            lazy_array = dask.delayed(tifffile.imread)(self.filename, level=level)
            data = dask.array.from_delayed(lazy_array, shape=self.shapes[level], dtype=self.dtype)
            dask_data.append(data)
    else:
        lazy_array = dask.delayed(tifffile.imread)(self.filename, level=level)
        dask_data = dask.array.from_delayed(lazy_array, shape=self.shapes[level], dtype=self.dtype)
    return dask_data
```

OME-Zarr data is read through `ome_zarr.reader.Reader` from [ome-zarr](https://pypi.org/project/ome-zarr/), preserving multiscale data levels.

Source: [`src/muvis_align/image/ZarrDaskSource.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/image/ZarrDaskSource.py)

```python
reader = Reader(location)
nodes = list(reader())
image_node = nodes[0]
self.data = image_node.data
self.metadata = image_node.metadata

self.shapes = [level.shape for level in self.data]
self.shape = self.shapes[0]
self.dtype = self.data[0].dtype
```

`Fusion` from multiview-stitcher can write directly to Zarr during computation, rather than creating a full fused array in memory first.

Source: [`src/muvis-align/MVSRegistration.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis-align/MVSRegistration.py)

```python
with dask.config.set(scheduler='threads'):
    fused_image = fusion.fuse(
        sims,
        fusion_func=fuse_func,
        transform_key=transform_key,
        output_stack_properties=output_stack_properties,
        output_zarr_url=output_filename,
        zarr_options=zarr_options,
        output_chunksize=output_chunksize
    )
```

---

## FAIR implementation

Rather than only documenting FAIR principles, `muvis-align` implements concrete FAIR-supporting mechanisms:

* project template defined as workflow schema in Bilayers format
* structured parameters in YAML
* OME-Zarr / OME-TIFF image outputs
* RO-Crate generation
* persistent mappings and metrics files
* napari and CLI interfaces
* container metadata for reproducible execution


## Declarative UI definition based on workflow schema

Source: [`src/muvis_align/ui/project_template.yaml`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/ui/project_template.yaml)

The project_template.yaml file is written in the Bilayers format and serves as the declarative description of the muvis-align workflow.
The file is used to dynamically generate the [magicgui](https://pyapp-kit.github.io/magicgui/)-based [napari](https://napari.org/stable/) user interface, allowing workflow parameters, input definitions, and processing options to be defined in a structured configuration file rather than being hard-coded in the application.
This approach simplifies interface maintenance, improves reproducibility, and enables the same workflow definition to be reused across the [napari](https://napari.org/stable/) plugin, command-line execution, and containerized deployments.
Bilayers schemas are represented as a [Pydantic](https://docs.pydantic.dev/) model, which adds typed validation and structured serialization for this workflow definition layer.

Source: [`src/muvis_align/ui/bilayers_util.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/ui/bilayers_util.py)

```python
from pydantic import BaseModel, ConfigDict, Field, field_validator


class BilayersField(BaseModel):
    model_config = ConfigDict(extra='allow')

    name: str
    type: str
    label: str | None = None
    default: Any = None
    description: str | None = None
    section_id: str | None = None
    section_key: str | None = None
    cli_order: int = 0

    @field_validator('type', mode='before')
    @classmethod
    def _normalize_type(cls, value):
        return str(value).lower()

    @field_validator('cli_order', mode='before')
    @classmethod
    def _normalize_cli_order(cls, value):
        if value is None or value == '':
            return 0
        return int(value)


def get_section_dict(template, keys=None):
    sections = {}
    if keys is None:
        keys = [None]

    for key in keys:
        items = template.get(key, []) if key is not None else template
        for raw_item in items:
            item = BilayersField.model_validate({**raw_item, 'section_key': key})
            section_id = item.section_id or key
            sections.setdefault(section_id, []).append(item)

    return {
        section_id: [item.model_dump() for item in sorted(section_items, key=lambda item: item.cli_order)]
        for section_id, section_items in sections.items()
    }
```

The package creates RO-Crates for outputs and adds acquisition/workflow metadata.

Source: [`src/muvis_align/file/rocrate_utils.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/file/rocrate_utils.py)

```python
def create_ro_crate(dest_path, image_paths=[]):
    crate = ZarrCrate()

    for image_path in image_paths:
        crate.add_dataset(dest_path=image_path)

    properties = {"fbbi_id": {"@id": 'obo:FBbi_00000257'}}
    crate.add(ImageAcquistion(crate, properties=properties))

    workflow_schema_filename = os.path.join('src', 'muvis_align/', NAPARI_PROJECT_TEMPLATE)
    crate.add_workflow(workflow_schema_filename)

    crate.write(dest_path)
    return crate
```

OME-Zarr outputs receive a Zarr-specific RO-Crate.

Source: [`src/muvis_align/file/rocrate_utils.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/file/rocrate_utils.py)

```python
def create_zarr_ro_crate(dest_path):
    crate = ZarrCrate()

    properties = {}
    properties['name'] = get_filetitle(dest_path)
    dataset_entity = crate.add_dataset(dest_path='.', properties=properties)

    acquisition_properties = {
        '@type': 'image_acquisition',
        'fbbi_id': {'@id': 'obo:FBbi_00000257'},
    }
    acquisition_entity = ContextEntity(crate, '#acquisition-001', acquisition_properties)
    crate.add(acquisition_entity)

    dataset_entity['resultOf'] = acquisition_entity

    crate.write(dest_path)
    return crate
```

---

## Persistent metadata

Input metadata is read from OME-TIFF or OME-Zarr and converted into physical units, positions, rotations, channels, scales, and transforms.

### Interoperable NGFF RFC 6 transforms

Registration transforms are written in the [OME-NGFF RFC 6 coordinate-transform format](https://github.com/ome/ngff-spec/blob/main/index.md#coordinateTransformations-metadata) rather than as application-specific matrix arrays. Each affine identifies its image path and names its input (`source_metadata`) and output (`registered`) coordinate systems. Using the `ome-zarr-models` v0.6 model for both serialization and validation makes the saved transforms portable to other tools that implement the NGFF coordinate-system and transformation model.

Source: [`src/muvis_align/file/transforms.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/file/transforms.py)

```python
def metadata_models(path, transform, source_key='source_metadata', transform_key='registered'):
    affine = Affine(
        path=path,
        affine=transform,
        input=CoordinateSystemIdentifier(name=source_key),
        output=CoordinateSystemIdentifier(name=transform_key),
    )
    return affine.model_dump(exclude_none=True, exclude_defaults=True, exclude_unset=True)


def read_transforms(filename):
    transforms = import_json(filename)
    return {label: Affine.model_validate(transform).affine for label, transform in transforms.items()}
```

Source: [`src/muvis_align/image/ZarrDaskSource.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/image/ZarrDaskSource.py)

```python
axes = self.metadata['axes']
dims = ''.join([axis['name'] for axis in axes])
self.dimension_order = dims
units = {axis['name']: axis['unit'] for axis in axes if 'unit' in axis}

for ct_index, transforms in enumerate(self.metadata.get('coordinateTransformations', [])):
    scale = {}
    position = {}
    for transform in transforms:
        if transform['type'] == 'scale':
            scale = {dim: value for dim, value in zip(dims, transform['scale']) if dim in dims_used}
        if transform['type'] == 'translation':
            position = {dim: value for dim, value in zip(dims, transform['translation']) if dim in dims_used}
```

The workflow converts SIMs into `MSIM` objects and performs pairwise/global registration on these multiscale spatial images.

Source: [`src/muvis_align/MVSRegistration.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/MVSRegistration.py)

```python
self.msims = [msi_utils.get_msim_from_sim(sim) for sim in self.sims]
results = self.register_global(sims, self.msims, register_indices=register_indices, params=params)
```

The parsed OME metadata (axes, units, voxel scale, translations/positions, channels, and coordinate transforms) is preserved in this [`xarray`](https://docs.xarray.dev/)-based `MSIM` (multiscale-spatial-image) representation, where image data remains in labeled dimensions and spatial context is stored as coordinates/attributes. This metadata is propagated through image operations, updated during pairwise/global registration, and carried into fusion/export (for example, OME-Zarr) to preserve physical-space interpretation and multiscale provenance.

The CLI runs the pipeline from a YAML parameter file.

Source: [`run.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/run.py)

```python
parser = argparse.ArgumentParser(description=f'muvis-align')
parser.add_argument('params',
                    help='The parameters file',
                    default='resources/params.yml')

args = parser.parse_args()
print(f'Parameters file: {args.params}')
with open(args.params, 'r', encoding='utf8') as file:
    params = yaml.safe_load(file)

pipeline = Pipeline(params)
pipeline.run()
```

---

## Performance metrics


The project calculates registration metrics such as NCC, SSIM, normalized mutual information, MSE-derived score, match rate, and distance-based feature metrics.
### Registration quality metrics

`muvis-align` computes several complementary image similarity metrics to evaluate registration quality. Each metric captures a different aspect of alignment quality, making it useful to evaluate multiple metrics together rather than relying on a single score.

| Metric | Full name                                 | Measures                                             | Typical range |
| ------ | ----------------------------------------- | ---------------------------------------------------- | ------------- |
| NCC    | Normalized Cross-Correlation              | Intensity correlation between images                 | -1 to 1       |
| SSIM   | Structural Similarity Index Measure       | Similarity of local image structures                 | 0 to 1        |
| ONMI   | Overlapping Normalized Mutual Information | Shared information between overlapping image regions | 0 to 1+       |

The metrics are implemented in `metrics.py`:

```python
all_metric_funcs = {
    'ncc': multiview_stitcher.metrics.normalized_cross_correlation,
    'ssim': lambda im1, im2: structural_similarity(
        np.nan_to_num(im1),
        np.nan_to_num(im2),
        data_range=data_range,
        channel_axis=reg_channel),
    'onmi': lambda im1, im2:
        normalized_mutual_information(
            np.nan_to_num(im1),
            np.nan_to_num(im2)) - 1,
    'mse': lambda im1, im2:
        1 / mean_squared_error(im1, im2),
}
```

Source: `src/muvis_align/metrics.py`

#### NCC — Normalized Cross-Correlation

NCC measures the correlation of image intensities between two images. It is particularly useful when corresponding structures have similar brightness values.

* `NCC = 1` indicates perfect correlation.
* `NCC = 0` indicates no correlation.
* Negative values indicate inverse correlation.

NCC is computationally efficient and widely used for rigid image registration, although it can be sensitive to intensity variations between acquisitions.

#### SSIM — Structural Similarity Index Measure

SSIM evaluates the similarity of local image structure by comparing:

* intensity
* contrast
* local texture

Unlike pixel-wise metrics, SSIM is sensitive to structural preservation and is therefore useful for assessing whether biological features remain aligned after registration.

* `SSIM = 1` indicates identical structure.
* Values close to zero indicate poor structural agreement.

SSIM is often more representative of perceived image quality than correlation-based metrics.

#### ONMI — Overlapping Normalized Mutual Information

ONMI measures the amount of shared information between overlapping image regions.

Mutual information is particularly valuable when images originate from different modalities, channels, or acquisition conditions because it does not assume similar intensity values.

Higher ONMI values indicate that:

* corresponding image regions contain similar information,
* overlapping structures are well aligned,
* registration improves the statistical dependence between views.

Because ONMI is based on image information content rather than intensity similarity, it is often more robust for multimodal registration problems.

Using NCC, SSIM, and ONMI together provides complementary measures of registration quality:

* **NCC** evaluates intensity agreement.
* **SSIM** evaluates structural similarity.
* **ONMI** evaluates shared image information.

This combination gives a more robust assessment of registration performance than any single metric alone.

---

## Storing data for resumable workflows

`muvis-align` stores intermediate registration state in mapping and metrics files, allowing workflows to continue from pairwise registration, global registration, or fused output.

Source: [`src/muvis-align/constants.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis-align/constants.py)

```python
default_pair_mappings_name = 'pair_mappings.json'
default_mappings_name = 'mappings.json'
metrics_name = 'metrics.json'
metrics_tabular_name = 'mappings.csv'
```

The registration state machine tracks progress through the workflow.

Source: [`src/muvis-align/MVSRegistration.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis-align/MVSRegistration.py)

```python
class RegState(Enum):
    UNINIT = auto()
    INIT = auto()
    SIMS_INIT = auto()
    PAIRS_REG = auto()
    GLOBAL_REG = auto()
    FUSED = auto()
```

Existing output files are detected before re-running expensive steps.

Source: [`src/muvis-align/MVSRegistration.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis-align/MVSRegistration.py)

```python
def check_progress(self, output_filename, output_format):
    pair_mappings_filename = self.output + self.output_params.get('pair_mappings', default_pair_mappings_name)
    mappings_filename = self.output + self.output_params.get('mappings', default_mappings_name)
    if self.output_exists(output_filename, output_format):
        self.state = RegState.FUSED
    elif os.path.exists(mappings_filename):
        self.state = RegState.GLOBAL_REG
    elif os.path.exists(pair_mappings_filename):
        self.state = RegState.PAIRS_REG
```

Pair mappings, final mappings, and metrics are saved separately.

Source: [`src/muvis-align/MVSRegistration.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis-align/MVSRegistration.py)

```python
self.save_pair_mappings(pair_results['pair_mappings'], qualities, bboxes)
results = self.register_global(sims, self.msims, register_indices=register_indices, params=params)
self.save_mappings(results['mappings'])
self.save_metrics(results['metrics'])
```

### Restoring registered transforms from intermediate output

Registered alignment transforms are persisted as JSON intermediate outputs (`pair_mappings.json` and `mappings.json`) and reloaded on later runs when these files already exist.

Source: [`src/muvis_align/MVSRegistration.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/MVSRegistration.py)

```python
def check_progress(self, output_filename, output_format):
    pair_mappings_filename = self.output + self.output_params.get('pair_mappings', default_pair_mappings_name)
    mappings_filename = self.output + self.output_params.get('mappings', default_mappings_name)
    if self.output_exists(output_filename, output_format):
        self.state = RegState.FUSED
    elif os.path.exists(mappings_filename):
        self.state = RegState.GLOBAL_REG
    elif os.path.exists(pair_mappings_filename):
        self.state = RegState.PAIRS_REG
```

When progress indicates pairwise/global registration already exists, `init_progress` loads saved mappings and reapplies transforms to SIMs instead of recomputing registration.

Source: [`src/muvis_align/MVSRegistration.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/MVSRegistration.py)

```python
if self.is_global_registered():
    mappings = import_json(mappings_filename)
    for sim, label in zip(sims, self.file_labels):
        mapping = param_utils.affine_to_xaffine(np.array(mappings[label]))
        value = mapping
        si_utils.set_sim_affine(sim, value, transform_key=self.reg_transform_key)
```

JSON persistence/reload is handled by shared utilities:

Source: [`src/muvis_align/util.py`](https://github.com/ccp-volume-em/muvis-align/blob/main/src/muvis_align/util.py)

```python
def import_json(filename):
    with open(filename, encoding='utf8') as file:
        data = json.load(file)
    return data


def export_json(filename, data):
    with open(filename, 'w', encoding='utf8') as file:
        json.dump(data, file, indent=4)
```

---

## Interactive interface with Xpra container

The Dockerfile defines a standard napari container and a second Xpra-enabled image for browser-based interactive use.

Source: [`Dockerfile`](https://github.com/ccp-volume-em/muvis-align/blob/main/Dockerfile)

```dockerfile
FROM python:3.11-slim-bookworm AS muvis-align

COPY requirements.txt .
COPY run.py .
COPY pyproject.toml .
COPY src/ src/

RUN pip install -r requirements.txt
RUN pip install napari[all]
RUN pip install .

ENTRYPOINT ["python3", "-m", "napari", "--plugin", "muvis-align"]
```

The Xpra stage installs Xpra and launches napari through a browser-accessible session.

Source: [`Dockerfile`](https://github.com/ccp-volume-em/muvis-align/blob/main/Dockerfile)

```dockerfile
FROM muvis-align AS muvis-align-xpra

RUN apt-get update && \
    apt-get install -yqq \
        xpra \
        xvfb \
        menu-xdg \
        xdg-utils \
        xterm \
        sshfs && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

ENV DISPLAY=:100
ENV XPRA_PORT=9876
ENV XPRA_START="python3 -m napari --with muvis-align"
ENV XPRA_EXIT_WITH_CLIENT="yes"
ENV XPRA_XVFB_SCREEN="1920x1080x24+32"
EXPOSE 9876
```

```dockerfile
CMD echo "Launching napari on Xpra. Connect via http://localhost:$XPRA_PORT or $(hostname -i):$XPRA_PORT"; \
    xpra start \
    --bind-tcp=0.0.0.0:$XPRA_PORT \
    --html=on \
    --start="$XPRA_START" \
    --exit-with-client="$XPRA_EXIT_WITH_CLIENT" \
    --daemon=no \
    --xvfb="/usr/bin/Xvfb +extension Composite -screen 0 $XPRA_XVFB_SCREEN -nolisten tcp -noreset" \
    --pulseaudio=no \
    --notifications=no \
    --bell=no \
    $DISPLAY
```

---

## Summary table

| Practice                | Source-code implementation                                                  |
| ----------------------- | --------------------------------------------------------------------------- |
| NGFF                    | OME-Zarr default extension, NGFF v0.5, `write_sim_to_ome_zarr`              |
| Larger-than-memory data | Dask-backed TIFF and Zarr sources, chunked fusion, direct Zarr output       |
| FAIR implementation     | package metadata, YAML configuration, RO-Crate generation, OME-Zarr outputs |
| Persistent metadata     | OME metadata parsing, `xarray`/`MSIM` spatial transforms, positions, scales, channels; interoperable NGFF RFC 6 affine transforms |
| Performance metrics     | NCC, SSIM, ONMI, MSE-derived score, match metrics, timing logs              |
| Resumable workflow      | state machine, pair mappings, global mappings, metrics JSON                 |
| Xpra interface          | Docker stage exposing browser-based napari over Xpra                        |

