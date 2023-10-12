# Upgrading template to latest yang repository

Templates can be upgraded to the latest YANG repository using the `upgrade-template` RPC.

This procedure consists of the following steps:

1. **Read template** - Reads a template configuration from the
   *templates* topology in the *Configuration* datastore.
2. **Version-drop** - Converts a template into a target schema context
   created by the specified yang-repository. Because of this feature,
   it is possible to change templates between different versions of devices
   with different revisions of YANG schemas but with similar structure.
   Version-drop is also aware of the `ignoredDataOnWriteByExtensions`
   RESTCONF filtering mechanism.
3. **Removal of previous template / writing new template** - If
   `upgraded-template-name` is not specified in the RPC input,
   the previous template is deleted and replaced by a new one.
   If specified, the previous template will not be deleted.

RPC input fields:

- **template-name**: Name of the existing input template. This field is mandatory.
- **upgraded-template-name**: Name of upgraded/new template. This field is optional.
- **yang-repository**: Name of YANG repository against
  which version-dropping is used. This field is optional.
  If no yang-repository is specified, the latest yang repository is used.

RPC output fields:

- Successful response: Returns HTTP status code 200 with no fields.

- Failed response: Returns HTTP status code 400-500 and an error with
  the following fields:
   - **error-type** : Type of the error.
   - **error-tag** : Tag of the error, also determining HTTP status code.
   - **error-message** : Description of the error that occurred during
  application of template.
   - **error-info** : Additional information related to the error. For example:
  node identification, topology identification.

  No fields are used, only HTTP response codes [200 - OK, 404 - Fail]

## RPC examples

### Successful example

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

### Auto-upgrading templates

This feature is used to automatically upgrade all stored templates that use the old
YANG repository to the latest YANG repository with help from the version-drop procedure.
For the auto-upgrade process to succeed, the latest YANG repository must already be configured.
The upgrade process must be explicitly enabled in the configuration file and occurs
when UniConfig is started.

Additionally, there is an option to back up templates before the upgrade with the standard rotation procedure.
The names of backed-up templates follow the pattern '{template-name}_backup_{index}',
where '{template-name}' represents the name of the original template and'{index}'
represents the backup index. The most recent backup index is always '0' and older ones
are rotated by incrementing the corresponding index. If a backed-up template reaches
the configured limit (maximum number of backups), it is permanently removed from the database.

Available settings ('application.properties'):

```properties UniConfig templates configuration (config/application.properties)
# Template settings
templates.enabled=false
templates.latest-schema-reference-module-name=system
templates.enabled-templates-upgrading=false
templates.backup-templates-limit=7
```

- **enabledTemplatesUpgrading** - Enable the auto-upgrading process at UniConfig startup.
  If disabled, the other setting is ignored.
- **backupTemplatesLimit** - Maximum number of stored backup templates.
  If exceeded, older templates are removed during the rotation procedure.
  If set to 0, templates are not backed up at all.
