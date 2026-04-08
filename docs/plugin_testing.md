# Writing Tests For Plugins
Well-written tests ensure your plugin is maintainable for future developers, and help locate the source of issues a user might encounter.
We recommend using PyTest to write tests for plugins, please refer to the documentation: https://docs.pytest.org/en/stable/getting-started.html

## Installation Tests
Installation tests are used to check our package was installed in the current environment, and that the module can be imported without errors.

### Test: Package is installed and detected by Python:
```
from importlib.metadata import distributions

def test_module_is_installed():
    packages = [dist.metadata.get("Name") for dist in distributions()]
    required = ["empanada-napari", "torch", "napari"]
    missing = [pkg for pkg in required if pkg not in packages]
    assert not missing, f"Missing packages: {', '.join(missing)}"
```
In the above test, we're checking that our package name `empanada-napari` exists in the list of package distributions that Python detects.

### Test: Package module (and some main dependencies) can import without error:
```
def test_module_imports():
    try:
        import napari
        import torch
        import empanada_napari
    except ImportError as e:
        pytest.fail(f"Failed to import required module: {e}")
```
This test can catch initial errors in importing `empanada_napari`, and some of its main dependencies such as `napari` and `torch`. This test can fail if the packages weren't installed correctly, or if there are version mismatches between the packages and their common dependencies.

Installation testing is also a useful way to check that dependencies will interact correctly.  
For example, many Napari plugins rely on packages like PyTorch or TensorFlow to run computations on the GPU, so we might want to check that PyTorch can access the GPU.  
If the machine does not have a GPU, we may want to skip running this test automatically.

In Pytest, we can add markers to our tests which tell it to run (or skip) on specific conditions.  
In this example, we will create a marker that will tell pytest to skip a test if the system doesn't have a CUDA-compatible installation of PyTorch.
In `tests/conftest.py`, we override the function `pytest_collection_modifyitems(config, items)` to register our custom marker(s):
```
def pytest_collection_modifyitems(config, items):
    try:
        import torch
        has_gpu = torch.cuda.is_available()
    except Exception:
        has_gpu = False

    if not has_gpu:
        skip_gpu = pytest.mark.skip(reason="No GPU available")
        for item in items:
            if "gpu" in item.keywords:
                item.add_marker(skip_gpu)
```
If the environment does not have CUDA-compatible PyTorch, we skip any test containing a marker with "gpu" in it. Below are examples of installation tests using this marker:

### Test: PyTorch can access the GPU (if available):
```
@pytest.mark.gpu
def test_torch_cuda_available():
    import torch
    if torch.version.cuda is None:
        pytest.skip("PyTorch not built with CUDA - GPU acceleration unavailable")
    
    if not torch.cuda.is_available():
        pytest.skip("CUDA not available to PyTorch - GPU acceleration unavailable")
    
    print(f"\nPyTorch CUDA version: {torch.version.cuda}")
    print(f"CUDA devices available: {torch.cuda.device_count()}")
```
This test simply prints the version of CUDA-PyTorch, and the number of CUDA devices.

### Test: CUDA runtime can handle computations:
```
@pytest.mark.gpu
def test_cuda_usable():
    """Verify CUDA runtime actually works"""
    try:
        x = torch.tensor([1.0], device="cuda")
        y = x * 2
        assert y.item() == 2.0
    except Exception as e:
        raise AssertionError(f"CUDA runtime failed: {e}")
```
This test performs a simple computation on the GPU using PyTorch, and is skipped if a CUDA build of PyTorch is not installed:
```
Run python -m pytest -s -vv tests
============================= test session starts ==============================
platform linux -- Python 3.10.19, pytest-9.0.2, pluggy-1.6.0 -- /opt/hostedtoolcache/Python/3.10.19/x64/bin/python
cachedir: .pytest_cache
PyQt5 5.15.11 -- Qt runtime 5.15.18 -- Qt compiled 5.15.14
rootdir: /home/runner/work/empanada-napari/empanada-napari/tests
configfile: pytest.ini
plugins: npe2-0.8.1, napari-0.6.6, dependency-0.6.1, qt-4.5.0, napari-plugin-engine-0.2.1, anyio-4.12.1
collecting ... collected 1 items

tests/test_installation.py::test_cuda_usable SKIPPED (No GPU available)
```
 
##Note For Containers:
If your plugin will be packaged within a container, below are examples of extra tests we recommend: 

### Test: The DISPLAY env variable is set and the Napari GUI works
```
def test_display_set():
    import os
    if os.getenv("GITHUB_ACTIONS") == "true":
        pytest.skip("Skipping in GitHub Actions")
    if not os.environ.get("DISPLAY"):
        pytest.fail("DISPLAY unset - napari GUI unavailable")
```
Often, users will run a container on a remote machine or HPC cluster. In order to run the Napari GUI on the host system, the `$DISPLAY` variable needs to be set, which is what this test verifies. 


### Test: The container has read/write access the main filesystem
```
def test_host_mounts_accessible():
    # Non-User Filesystems and mount points that existed in Linux docker container
    # Transferability to other runtimes and host systems not guaranteed
    EXCLUDE_FSTYPES = {"overlay", "proc", "tmpfs", "devpts", "sysfs",
                       "cgroup", "cgroup2", "mqueue", "devtmpfs"}
    EXCLUDE_MOUNTPOINTS = re.compile(r"^(/etc/|/proc|/sys|/dev)")
    with open("/proc/mounts") as f:
        for line in f:
            _, mountpoint, fstype, *_ = line.split()
            if fstype in EXCLUDE_FSTYPES:
                continue
            if EXCLUDE_MOUNTPOINTS.match(mountpoint):
                continue
            # Check read access (N.B. moot if container run as root)
            if not os.access(mountpoint, os.R_OK):
                warnings.warn(f"Mountpoint {mountpoint} ({fstype}) is not readable")
                continue
            # Check write access
            if not os.access(mountpoint, os.R_OK):
                warnings.warn(f"Mountpoint {mountpoint} ({fstype}) is not writeable")
                continue
            return  # plausible filesystem r+w mount 
    pytest.fail(
        "No host filesystems detected with read+write access"
        "You may need to bind mount data with -v /host/path:/container/path"
    )
```

When running Napari and the plugin in a container, the host's filesystem needs to be accessible so the plugin can read the user's data. This test checks the container's `/proc/mounts` list shows read/write access to the mounted filesystem.

## Unit Tests
Unit tests are written to make sure that each function returns an expected output, given a particular input. This should be the case regardless of the actual implementation of the function.

### Test: The _join_ranges() function should return the expected list of ranges
```
@pytest.mark.parametrize(['ranges', 'expected'],
    [([(0,10), (6, 10)], [[0, 10]]),
     ([(0,10), (11, 20)], [[0, 10], [11, 20]]),
     ([(0,10), (10, 20)], [[0, 20]])],
    ids=["overlapping_ranges", "non_overlapping_ranges",
         "border_ranges"])
def test__join_ranges(ranges, expected):
    """
    Test `_join_ranges` function.

    The function should:
    - read in a list of ranges of (start, end)
    - return an array containing a joined [start, end] range

    Expected:
    - A list of [start, end] ranges
    """
    new_ranges = _join_ranges(np.array(ranges))
    assert np.array_equal(new_ranges, expected)
```

This function takes a list containing a list of ranges, where the ranges overlap, it will return a joined range. If they don't, it should return the original ranges. 
The test looks at three scenarios: overlapping ranges (where the overlap is >=1), non-overlapping ranges, and border ranges (where the difference between the end of range 1 and beginning of range 2 is exactly 0). The output from the function is compared to our expected list of ranges, for each given scenario.

Below is an example of two different implementations of `_join_ranges()`:

### Implementation A:
```
def _join_ranges(ranges):
    joined = []
    running_range = None
    for range1,range2 in zip(ranges[:-1], ranges[1:]):
        if running_range is None:
            running_range = range1

        if running_range[1] >= range2[0]:
            running_range[1] = max(running_range[1], range2[1])
        else:
            joined.append(running_range)
            running_range = None

    if running_range is not None:
        joined.append(running_range)
    else:
        joined.append(range2)

    return joined
```

### Implementation B:
```
from functools import reduce

def _join_ranges(ranges=None):
    if ranges is None:
        return []

    def reducer(curr, nxt):
        last = curr[-1]
        if last[1] >= nxt[0]:
            curr[-1] = [last[0], max(last[1], nxt[1])]
        else:
            curr.append(nxt)
        return curr

    return reduce(reducer, ranges[1:], [ranges[0]])
```

We should write our unit tests so that either of these implementations will pass with the test data.


## Integration Tests
Integration tests are a step above unit tests, they check that different modules or components work together correctly (as opposed to individual functions).
With Napari plugins, it is good practice to write tests for individual widgets.   
In the example below, we are testing the `Slice Inference` widget by instantiating the `SliceInferenceWidget` object with the user's setup arguments, then running the model.

When this widget is used in the Napari GUI, the `viewer` and `image_layer` objects are accessed from Napari itself. If we are running our tests as part of a CI/CD pipeline, it is more efficient to use a mock version of Napari's objects and avoid triggering the GUI at all.  
Below, we use Napari's `ViewerModel` object to act as our mock viewer. We can then pass an image to this object, simulating an image layer:
```
from napari.components import ViewerModel

def test_slice_inference(self, image_2d, test_args, expected_shape):
    viewer = ViewerModel()
    image_layer = viewer.add_image(image_2d)
    inference_config = SliceInferenceWidget(viewer=viewer,
                                    image_layer=image_layer,
                                    **test_args)
    seg, _, _, _, _ = inference_config.config_and_run_inference(use_thread=False)
    assert isinstance(seg, np.ndarray)
    assert np.asarray(seg).shape == expected_shape
```
Finally, we assert that the output data is of the expected type and shape. This style of test can help find errors in how different components interact, especially issues that unit tests (which test pieces in isolation) might miss.

## Measuring test coverage
You can check test coverage locally using pytest-cov

```
pip install pytest-cov
```

Run your tests with:
```
pytest --cov=<your_package> --cov-report=html
```

Open the resulting report in your browser: 
```
open htmlcov/index.html
```

The report will show line-by-line what is being tested, and what is being missed. Ideally, we want everything covered by our tests. If you have lines that you know never need to be tested (like debugging code) you can omit specific lines from coverage with the comment ```# pragma: no cover```
