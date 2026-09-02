# Making software public

Publishing research software is more than making a repository visible. A useful
public release should be installable, documented, licensed, citable and free of
information that should remain private. The recommendations below assume that
the software has already been tested, reviewed and versioned as described in
[Software Development](software.md).

## Before publishing

Review the complete repository history as well as the current files. Removing a
secret from the latest commit does not remove it from earlier commits.

Before the first public release:

* Confirm that all contributors and, where relevant, their employers or funders
  agree that the software can be released.
* Remove credentials, access tokens, private URLs, personal data, patient data,
  confidential issue discussions and data that you do not have permission to
  redistribute. Revoke and replace any credential that has ever been committed.
* Check the licences of source code, models, example data, fonts, icons and
  other third-party material. Record their copyright and attribution notices.
* Add a `README`, a licence, installation and usage instructions, contributor
  guidance, a citation file and a way to report problems.
* Test installation in a clean environment and run the automated test suite.
  Test the built package, not only the source checkout.
* Choose a release version and prepare release notes describing important
  changes, known limitations and any compatibility breaks.

Avoid committing large datasets, model weights and generated build files to the
source repository. Deposit large research outputs in an appropriate public repository
and link to them using a persistent identifier.

## GitHub

A public [GitHub repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/about-repositories)
makes the source and its history discoverable and gives users a place to report
issues and contribute changes. Before changing a private repository to public,
inspect branches, tags, pull requests, Actions logs and release assets: some of
these also become visible. GitHub documents the
[consequences of changing repository visibility](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility).

Suggested contents for the repository:

* `README.md`: purpose, status, supported platforms, installation, a minimal
  example, links to full documentation, support and citation instructions.
* `LICENSE` or `LICENSE.md`: the full licence text.
* `CITATION.cff`: title, authors, ORCID identifiers, version, release date and
  DOI, when available. GitHub uses this file to display a **Cite this
  repository** link; see its
  [citation-file guidance](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-citation-files).
* `CONTRIBUTING.md`: how to set up a development environment, run tests and
  submit a change.
* `CODE_OF_CONDUCT.md`: expected behaviour and a route for confidential
  reporting.
* `SECURITY.md`: supported versions and how to report a vulnerability without
  first disclosing it publicly.
* A dependency or environment specification, with direct dependencies and
  supported language versions declared accurately.

Use an organisation-owned repository for collaborative projects when possible,
so that stewardship does not depend on one person's account. Give at least two
maintainers the permissions needed to make a release and enable multi-factor
authentication and branch protection.

### Create a release

Create a release from a reviewed commit on the default branch:

1. Update the version, changelog, citation metadata and documentation.
2. Run tests and build the documentation and distribution artefacts in a clean
   environment.
3. Create an annotated Git tag using the project's version convention, for
   example `v1.2.0`.
4. Create a GitHub Release from that tag and add release notes.
5. Publish packages and container images from the same tag, then test that
   users can install them using the documented commands.

[GitHub Releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases)
associate a fixed Git tag with release notes and downloadable assets. Do not
move or reuse a published tag. If a release is faulty, publish a new patch
release and explain the problem.

For long-term preservation and citation, connect the repository to a research
archive such as [Zenodo](https://help.zenodo.org/docs/github/). Archive each
release and add the resulting version-specific DOI to `CITATION.cff`. A concept
DOI can refer to the software as a whole, while a version DOI identifies the
exact software used in an analysis; prefer the version DOI when reproducibility
requires a particular release.

## Licence

Copyright exists even when no licence file is present. A public repository
without a licence can be viewed and forked under the hosting platform's terms,
but that does not generally give others permission to use, modify or distribute
the software. See [Choose a License](https://choosealicense.com)
for an accessible explanation.

Choose the licence before accepting external contributions, and obtain agreement
from all copyright holders. Consider:

* compatibility with the licences of dependencies and incorporated code;
* whether derived works may be distributed under other terms (a permissive
  licence) or must remain open under compatible terms (a copyleft licence);
* requirements imposed by an employer, funder, consortium or target ecosystem;
* patent provisions, attribution requirements and the intended commercial use.

Common choices include BSD-3-Clause, MIT and Apache-2.0 as permissive licences,
and GPL-3.0 as a strong copyleft licence. This list is not legal advice; consult
your institution's research software or legal team when ownership or licence
compatibility is unclear. Use a standard
[OSI-approved licence](https://opensource.org/licenses) without changing its
text, and include its exact SPDX identifier in package metadata.

A repository may need separate licences for software, documentation, data and
model weights. State clearly which files each licence covers. Creative Commons
licences may suit documentation and data but are
[not recommended for software](https://choosealicense.com/non-software/).
Retain third-party notices and make sure example data can legally be published.

## Statements about AI use

Generative-AI tools may influence source code, tests, documentation, images or
translations. The human contributors and maintainers remain responsible for
everything released: AI systems should not be listed as authors or copyright
holders.

Follow the policies of your institution, funder, publisher and software
community. When AI use was substantive, add a short statement to the repository,
release notes or associated paper. Record enough information to make the role of
the tool understandable, without publishing confidential prompts or sensitive
data. A statement can identify:

* the tool and model, including the version or access date where known;
* which parts of the work it assisted with;
* whether generated output was modified;
* how a human reviewed, tested and validated the result.

For example:

> Generative AI was used to suggest docstrings and unit-test cases. All
> suggestions were reviewed and edited by the maintainers, and the resulting
> software passed the project's automated tests. No confidential source code or
> research data was provided to the service.

If no generative-AI tool contributed to the release, a statement is normally
unnecessary unless a relevant policy explicitly requires one.

Do not submit personal, confidential, security-sensitive or unpublished
material to an external AI service unless its data handling terms and your
project approvals permit this. Treat generated code like any other external
contribution: understand it, review it for security and bias, test it, and check
for licence or attribution concerns before release. AI output can be plausible
but incorrect, and an AI-use statement does not replace quality assurance.


## PyPI package

Publishing a Python library or application on the
[Python Package Index (PyPI)](https://pypi.org/) allows users to install it with
`pip`. Follow the maintained
[Python Packaging User Guide](https://packaging.python.org/en/latest/tutorials/packaging-projects/)
rather than invoking `setup.py` directly.

Define the build system and package metadata in a single definition using `pyproject.toml`. Metadata should
include:

* a unique distribution name and version;
* a short description and `README` as the long description;
* the licence expression and licence files;
* required Python version and runtime dependencies;
* authors or maintainers;
* project URLs for source, documentation, issues and changelog;
* classifiers and optional dependencies where useful.

Build both a source distribution and a wheel, then check their metadata:

```console
python -m pip install --upgrade build twine
python -m build
python -m twine check --strict dist/*
```

Inspect the archive contents and install the wheel into a new virtual
environment. If the project has compiled extensions, provide wheels for the
supported combinations of operating system, architecture and Python version,
and test each build.

Use [TestPyPI](https://packaging.python.org/en/latest/guides/using-testpypi/)
to exercise a new packaging workflow before the production release. PyPI does
not permit a published file or version to be replaced, so corrections require a
new version.

For automated releases, prefer
[PyPI Trusted Publishing](https://docs.pypi.org/trusted-publishers/). It uses
short-lived identity tokens from the CI service instead of storing a long-lived
PyPI token in repository secrets. Restrict the publishing job to protected tags
or a protected release environment, and require review where appropriate.
Build once, test those exact artefacts, and promote the same files to the
production index.

After publishing, verify the package page and test the normal user workflow:

```console
python -m pip install your-package
python -c "import your_package"
```

## Containerisation

For maximal reproducibility, a key [FAIR](fair.md) principle, consider creating a
container definition that installs the software and its dependencies. A container
image bundles the interpreter, libraries, system packages and configuration that
the software needs, so users obtain the same environment as the developers. This
is particularly valuable for volume EM tools, which often combine Python
packages, compiled libraries, GPU frameworks and command-line utilities that are
difficult to install consistently across operating systems.

Containerisation complements, rather than replaces, a conventional package
release. Publish a [PyPI package](#pypi-package) or conda package for users who
want to work inside their own environment, and a container image for users who
want a fixed, ready-to-run environment or who work on shared infrastructure.

For an introduction to containers in this context, see
[Using Containers](https://rosalindfranklininstitute.github.io/volume-em-container-documentation/intro/containers/).

### Write the container definition

Keep the container definition, for example a `Dockerfile`, in the source
repository so that it is versioned and reviewed alongside the code. Build the
image from a released artefact or a specific commit rather than from uncommitted
local files.

Recommendations for the definition:

* Start from a minimal, actively maintained base image and record why it was
  chosen. Reuse an established base image, such as an official Python or CUDA
  image, instead of building an environment from scratch.
* Pinning dependecy versions confers reproducibility, but can overtime can lead to security vulnerabilities and build failures due to upstream availability.  We recommend a balanced approach whereby you fix the versions of dependencies your application depends on immediately,  leaving auxiliary system and build packages unpinned. If you need to guarantee binary reproducibility, consider minting an image in a public registry like Zenodo.
  system packages; and the versions of the software and its dependencies. An
  unpinned build produces a different environment every time it runs.
* Install the software as a package (`pip install .` or an equivalent) rather
  than copying source files into an arbitrary directory, so that version
  metadata and entry points are available inside the image.
* Use multi-stage builds to keep build tools out of the final image, and remove
  package-manager caches in the same layer that creates them.
* Run the application as a non-root user, and document any writable directories
  it needs.
* Define an `ENTRYPOINT` that runs the application, so that
  `docker run <image> --help` behaves as users expect.
* Add [OCI image labels](https://github.com/opencontainers/image-spec/blob/main/annotations.md)
  such as `org.opencontainers.image.source`, `.version`, `.revision`,
  `.licenses` and `.description`. These make an image traceable back to the
  exact commit that produced it.
* Keep large data, model weights and test datasets out of the image where
  possible; download or mount them at run time and document how.

Graphical applications, such as [napari](https://napari.org/) plugins, need
additional consideration: the container must be able to reach a display server,
either through the host's X11 socket or through a browser-based server such as
Xpra. For X server access with Docker, make sure the DISPLAY environmental variable is set and host display drivers are mounted at runtime — example docker commands can be found in the [RFI vEM container documentation](https://rosalindfranklininstitute.github.io/volume-em-container-documentation/). See [`muvis-align`](muvis-align.md#interactive-interface-with-xpra-container)
for a worked example of a napari image with an Xpra-enabled variant.

### Build and test in continuous integration

A container image can be built and tested in a continuous integration workflow,
which ensures the definition keeps working as dependencies change. At minimum,
build the image on pull requests and on release tags, and check that it starts
and reports its version without error.

For containerised research software we recommend:

* Building on every change to the container definition or dependency
  specification, and on a schedule to detect breakage caused by upstream
  updates.
* Running the test suite inside the built image, using a virtual display server
  such as `Xvfb` where a GUI is involved.
* Adding start-up or installation checks to the entrypoint, which warn the user
  when the runtime environment lacks something the application needs, such as a
  display, GPU access or a writable bind mount.
* Building for the architectures you intend to support, for example `amd64` and
  `arm64`, and testing each one.

See [Container Testing](container_testing.md) for a fuller discussion and
examples, including the start-up tests written for the napari Empanada plugin.

### Tag and publish

A container image can be built, properly tagged, and published to a public
registry such as [Docker Hub](https://hub.docker.com/),
[Quay.io](https://quay.io/) or the
[GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry).
Consider where your user community already looks for images, whether the
registry applies pull-rate limits or retention policies, and whether it will
still be available in several years.

The tags follow the same guidelines as release versioning. Likewise, a specific
tag can be referenced for reproducibility. A practical tagging scheme is:

* an immutable version tag matching the Git tag, for example `v1.2.0`, published
  once and never moved;
* optional moving tags such as `v1.2`, `v1` or `latest` for convenience;
* variant suffixes where several images are built from the same release, for
  example `v1.2.0-cuda` or `v1.2.0-xpra`.

Publish the image from the same tag as the corresponding
[GitHub Release](#create-a-release), and record the version in the image labels.
Where the registry supports it, generate build provenance and a software bill of
materials, so users can verify what an image contains and where it came from.

Automate publishing from CI using the registry's OpenID Connect or short-lived
credentials where available, rather than storing long-lived registry passwords in
repository secrets, and restrict the publishing job to protected tags or a
protected release environment.

### Document how to run the image

Include the container definition in the repository and provide instructions for
users to run it. Give a complete command that a new user can copy, and explain
each option that is not obvious:

```console
docker pull ghcr.io/your-org/your-tool:1.2.0
docker run --rm -v /path/to/data:/data ghcr.io/your-org/your-tool:1.2.0 --help
```

Document, as applicable:

* how to mount input and output data, and which paths the image expects;
* how to request GPU access, for example `--gpus all`, and which driver and
  CUDA versions are required;
* how to run a graphical interface, including any `DISPLAY` or X11 socket
  settings;
* the user and file-permission behaviour, for example `--user $(id -u):$(id -g)`
  when written files should be owned by the invoking user;
* resource expectations, such as memory, shared memory or disk space for large
  volumes.

### Shared infrastructure and archiving

Many users run analyses on HPC clusters, where Docker is usually unavailable
because it requires elevated privileges. Container runtimes such as
[Apptainer](https://apptainer.org/) (formerly Singularity) are commonly
installed instead, and can convert and run a published OCI image directly:

```console
apptainer pull your-tool_1.2.0.sif docker://ghcr.io/your-org/your-tool:1.2.0
apptainer run --bind /path/to/data:/data your-tool_1.2.0.sif --help
```

Test this route if you expect cluster use, since bind-mount behaviour, the home
directory, environment variables and the container user differ from Docker.

For work that must remain reproducible in the long term, reference images by
digest rather than by tag, since a digest identifies exact image content:

```console
docker run --rm your-tool@sha256:<digest>
```

Registries are not archives. Where a published analysis depends on a specific
image, record its digest in the methods or workflow description, and consider
depositing the container definition, the resolved dependency versions and, if
size permits, an exported image archive alongside the release in a research
archive such as [Zenodo](https://zenodo.org/). A container definition that can
be rebuilt from pinned inputs is often more durable than a stored image.
