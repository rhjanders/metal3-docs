# Live updates (servicing)

Live updates (servicing) enables baremetal-operator to conduct certain actions
on already provisioned BareMetalHosts. These actions currently include:

- configuring BIOS settings
- updating BIOS and/or BMC firmware

Live updates (servicing) is an opt-in feature. Operators may enable this
feature by creating a `HostUpdatePolicy` custom resource.

## HostUpdatePolicy custom resource definition

HostUpdatePolicy is the custom resource which controls applying live updates.
Each part of the functionality can be controlled separately by setting the
respective entry in the HostUpdatePolicy spec:

- `firmwareSettings` - controls BIOS setting changes
- `firmwareUpdates` - controls BIOS and BMC firmware updates

## Allowed values for firmwareSettings and firmwareUpdates fields

Each of the fields can be set to one of the two values:

- `onReboot` - which enables performing the requested change on next reboot, or
- `onPreparing` - which limits applying this type of change to Preparing
state (which only applies to nodes which are being provisioned)

## Example HostUpdatePolicy definition:

Here is an example of a HostUpdatePolicy CRD:

```yaml
apiVersion: metal3.io/v1alpha1
kind: HostUpdatePolicy
metadata:
  name: ostest-worker-0
  namespace: openshift-machine-api
spec:
  firmwareSettings: onReboot
  firmwareUpdates: onReboot
```

## How to perform Live updates on a BareMetalHost

- create a HostUpdatePolicy resource with the name matching the BMH to be
updated
- use the format above, ensure `firmwareSettings` and/or `firmwareUpdates` is
set to `onReboot`
- make changes to HostFirmwareSettings and/or HostFirmwareComponents as
required [Firmware Settings][firmware_settings.md]
[Firmware Updates][firmware_updates.md].

Example commands:

```
cat << EOF > hup.yaml
apiVersion: metal3.io/v1alpha1
kind: HostUpdatePolicy
metadata:
  name: ostest-worker-0
spec:
  firmwareSettings: onReboot
  firmwareUpdates: onReboot
EOF

oc apply -f hup.yaml

oc patch hostfirmwaresettings ostest-worker-0 --type merge -p 
'{"spec": {"settings": {"QuietBoot": "true"}}}'

oc patch hostfirmwarecomponents ostest-worker-0 --type merge -p 
'{"spec": {"updates": [{"component": "bios", 
                        "url": "http://10.6.48.30:8080/firmimgFIT.d9"}]}}'
oc annotate bmh ostest-worker-0 reboot.metal3.io=""
```
