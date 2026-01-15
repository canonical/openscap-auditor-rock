# OpenSCAP Auditor Rock

A lightweight OCI image (rock) built with Rockcraft that provides OpenSCAP security compliance auditing capabilities for container images.

**NOTE**: the [Ubuntu datastreams](https://github.com/ComplianceAsCode/content/releases) for STIG (Security Technical Implementation Guide) and CIS (Center for Internet Security) benchmarks are already included in the rocks, thus making it ready, off-the-shelf, to check compliance for other rocks and Ubuntu-based container images. Support for other container images can be added by either rebuilding the rock with the desired datastreams, or passing those at runtime into the container.

## Developer Guide

### Prerequisites

- [Rockcraft](https://canonical-rockcraft.readthedocs-hosted.com/en/latest/) installed
- LXD configured for building rocks
- Docker (for testing)

### Building the Rock

1. Clone this repository:
   ```bash
   git clone <repository-url>
   cd openscap-auditor-rock
   ```

2. Navigate to the rock's directory:
   ```bash
   cd openscap-auditor/1.4-25.10
   # If modifying the rock, edit the rockcraft.yaml file
   ```

3. Build the rock:
   ```bash
   rockcraft pack
   ```

4. Load the rock into Docker:
   ```bash
   sudo rockcraft.skopeo --insecure-policy copy \
     oci-archive:openscap-auditor_1.4_amd64.rock \
     docker-daemon:openscap-auditor:1.4
   ```


## User Guide

### Checking Compliance with STIG and CIS

The procedure is very similar for both, requiring you to
only have to define the "profile" you're checking compliance
against (i.e. `stig` or `cis_level2_server`). For example:

```bash
PROFILE=stig  # or cis_level2_server

# Pull the image you want to audit
TARGET_IMAGE=<YOURIMAGE>
docker pull $TARGET_IMAGE

# Set the VERSION_ID to be used to find the right datastream.
# For example, if the target image is based on 22.04,
# this should be 2204
VERSION_ID=2204

# Run STIG compliance scan
docker run --privileged --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ${PWD}:/report \
  ghcr.io/canonical/openscap-auditor-rock/openscap-auditor:1.4-25.10_edge exec \
  oscap-docker \
  image ${TARGET_IMAGE} \
  xccdf eval \
  --verbose ERROR \
  --profile ${PROFILE} \
  --report /report/report_${PROFILE}.html \
  /scap-security-guide-latest/ssg-ubuntu${VERSION_ID}-${PROFILE%%_*}-ds.xml

```

The scan will generate an HTML report in your current directory, named `report_stig.html`, or `report_cis_level2_server.html`,
depending on the `PROFILE` you chose.
