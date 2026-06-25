# Change history for releases

## Changes for v2 (initially released November 2024)

* Default homedir to /home from /var/lib due to https://access.redhat.com/solutions/7133297
* Drop undocumented "extra" features in code derived from ansible-workshops (i.e. `build_idm`, `build_sat`).
* Drop rpm-based installation scheme (hosts/ subdirectory).
* Drop "legacy" Makefile targets (they were for the non-containerized install)
* Drop support for AAP versions <= 2.4. This includes dropping support for mechanisms other than containerized install.
* Support for AAP 2.5
* Introduce direct support for running under OpenShift Validated Patterns
* Depend on infra.aap_utilities to handle some installation related work
* Add mechanism to disable eda_rulebook_activations prior to configuring them to prevent crash of aap configuration
  playbook while trying to manage running rulebook activations

## Update to v2 (April 2026)

* Note that AGOFv2 Also support AAP 2.6
* Add a parameter to the openshift pre-init playbook to allow skipping the local vault load.

## Update to v2 (June 2026)

* Restore AAP 2.5 install and config-as-code path alongside 2.6 and 2.7.

## Update to v2 (May 2026)

* Default AGOFv2 to AAP 2.6
* Improve numerous error messages and provide a pre-check target.
* Fix an issue where change to infra.aap_utilties.aap_setup_download could not find an installer to download, causing
installation to fail for containerized installer.
