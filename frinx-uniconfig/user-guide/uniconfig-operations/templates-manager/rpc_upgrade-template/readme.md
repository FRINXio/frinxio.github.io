# Upgrading template to latest yang repository

Template can be upgraded to latest YANG repository using 'upgrade-template' RPC.
This procedure consists of:

1. **Read template** - Reading of template configuration from
   'templates' topology in Configuration datastore.
2. **Version-drop** - Conversion of template into target schema context
   that is created by specified yang-repository. Because of this feature,
   it is possible to change template between different versions of devices
   with different revisions of YANG schemas but with similar structure.
   Version-drop is also aware of 'ignoredDataOnWriteByExtensions'
   RESTCONF filtering mechanism.
3. **Removal of previous template / writing new template** - If
   'upgraded-template-name' is not specified in RPC input,
   previous template will be deleted and replaced by new one.
   If it is specified, previous template will not be deleted.

Description of input RPC fields:

- **template-name**: Name of the existing input template. This field is mandatory.
- **upgraded-template-name**: Name of upgraded/new template. This field is optional.
- **yang-repository**: Name of YANG repository against
  which version-dropping is used. This field is optional.
  If no yang-repository is specified, latest yang repository will be used.

Description of output RPC fields:

- Successful response: returns http status code 200 without fields.

- Failed response: returns http status code 400-500 and error with
  fields shown below:
- **error-type** : Type of the error.
- **error-tag** : Tag of the error, also determining http status code.
- **error-message** : Description of the error that occurred during
  application of template.
- **error-info** : Additional information related to error. For example,
  node identification, topology identification.

  No fields are used, only HTTP response codes [200 - OK, 404 - Fail]

## RPC Examples

### Successful Example

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/template-manager:upgrade-template' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "template-name": "interface_template",
        "upgraded-template-name": "new_interface_template",
        "yang-repository": "schema1"
    }
}'
```

### Auto-upgrading of templates

This feature is used to automatically upgrade all stored templates using the old
YANG repository to the latest YANG repository with help from the version-drop procedure.
For the auto-upgrading process to work, the latest YANG repository must already be configured.
The upgrade process must be explicitly enabled in the configuration file and occurs
when UniConfig is started.

There is also an option to back up templates before the upgrade with the standard rotation procedure.
The names of backed-up templates follow the pattern '{template-name}_backup_{index}',
where '{template-name}' represents the name of the original template and'{index}'
represents the backup index. The most recent backup index is always '0' and older ones
are rotated by incrementing the corresponding index. If a backed-up template reaches
the configured limit (maximum number of backups), it is permanently removed from the database.

Overview of available settings ('application.properties'):

```properties UniConfig templates configuration (config/application.properties)
# Template settings
templates.enabled=false
templates.latest-schema-reference-module-name=system
templates.enabled-templates-upgrading=false
templates.backup-templates-limit=7
```

- **enabledTemplatesUpgrading** - Enables the auto-upgrading process at UniConfig startup.
  If disabled, the other setting is ignored.
- **backupTemplatesLimit** - Maximum number of stored backup templates.
  If exceeded, older templates are removed during the rotation procedure.
  If set to 0, templates are not backed up at all.
