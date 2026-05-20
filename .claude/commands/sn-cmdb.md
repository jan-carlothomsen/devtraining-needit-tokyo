# ServiceNow CMDB Operations

Write CMDB-related GlideRecord code for the NeedIt app (ES5, scope x_58872_needit).

Arguments: $ARGUMENTS
(Describe what you need: upsert a CI type, build relationships, query CIs, etc.)

## App Context
- Scope: `x_58872_needit` | JS level: `helsinki_es5`
- ES5 only: `var`, prototype pattern, no arrow functions, no `let`/`const`

## Key CMDB Tables
| Purpose | Table |
|---|---|
| Generic CI base | `cmdb_ci` |
| Servers / VMs | `cmdb_ci_server` |
| Applications | `cmdb_ci_appl` |
| Business Services | `cmdb_ci_service` |
| Computers / Endpoints | `cmdb_ci_computer` |
| Network devices | `cmdb_ci_netgear` |
| Database instances | `cmdb_ci_db_instance` |
| CI Relationships | `cmdb_rel_ci` |
| Relationship types | `cmdb_rel_type` |
| Source tracking | `cmdb_source_record` |

## Common CI Fields
```
name, ip_address, os, os_version, manufacturer, model_id,
serial_number, cpu_count, ram, install_status, operational_status,
discovery_source, correlation_id, location, assignment_group
```

## Install Status Values
```
1 = Installed  |  2 = On Order  |  3 = In Maintenance
6 = In Stock   |  7 = Retired   |  8 = Stolen
```

## GlideRecord Patterns

### Upsert (create or update)
```javascript
var gr = new GlideRecord('cmdb_ci_server');
gr.addQuery('name', ciName);
// add more identifier fields if needed:
// gr.addQuery('ip_address', ipAddress);
gr.query();
if (gr.next()) {
    // update existing
    gr.setValue('os', osValue);
    gr.setValue('ip_address', ipValue);
    gr.update();
    gs.info('Updated CI: ' + gr.sys_id);
} else {
    // create new
    gr.initialize();
    gr.setValue('name', ciName);
    gr.setValue('ip_address', ipValue);
    gr.setValue('os', osValue);
    gr.setValue('discovery_source', 'x_58872_needit');
    var newSysId = gr.insert();
    gs.info('Created CI: ' + newSysId);
}
```

### Create a CI Relationship
```javascript
// First: get the relationship type sys_id
var relType = new GlideRecord('cmdb_rel_type');
relType.addQuery('name', 'Runs on::Runs');  // forward::reverse name
relType.query();
relType.next();
var relTypeSysId = relType.sys_id.toString();

// Then: create the relationship
var rel = new GlideRecord('cmdb_rel_ci');
rel.initialize();
rel.setValue('parent', parentCiSysId);   // the CI that "runs on"
rel.setValue('child', childCiSysId);     // the CI being "run on" (e.g. server)
rel.setValue('type', relTypeSysId);
// prevent duplicates:
var existingRel = new GlideRecord('cmdb_rel_ci');
existingRel.addQuery('parent', parentCiSysId);
existingRel.addQuery('child', childCiSysId);
existingRel.addQuery('type', relTypeSysId);
existingRel.query();
if (!existingRel.next()) {
    rel.insert();
}
```

### Common Relationship Type Names (cmdb_rel_type.name)
```
Depends on::Used by
Hosted on::Hosts
Runs on::Runs
Connected by::Connects
Contains::Contained by
Members::Member of
Virtualized by::Virtualizes
```

### Bulk Query with Encoded Query
```javascript
var gr = new GlideRecord('cmdb_ci_server');
gr.addEncodedQuery('install_status=1^operational_status=1^discovery_source=x_58872_needit');
gr.setLimit(100);
gr.query();
while (gr.next()) {
    gs.info(gr.name + ' - ' + gr.ip_address);
}
```

### Deactivate / Retire a CI
```javascript
var gr = new GlideRecord('cmdb_ci_server');
if (gr.get(ciSysId)) {
    gr.setValue('install_status', 7);       // Retired
    gr.setValue('operational_status', 6);   // Non-Operational
    gr.update();
}
```

## What to produce

Based on the argument, write the appropriate GlideRecord code as a Script Include method or a standalone server-side script. Always:
- Use `var`, prototype pattern if it's a Script Include
- Return a result object `{ success, sysId, action, error }` from upsert methods
- Guard against GlideRecord insert/update failures with try/catch where appropriate
- Add `discovery_source = 'x_58872_needit'` on any CI you create
- Avoid full table scans — always use indexed query fields (`name`, `ip_address`, `sys_id`, `correlation_id`)

If the result is a complete Script Include, produce both the XML and JS files (see `/sn-script-include` pattern). If it is a standalone snippet, produce just the JavaScript and explain where to run it (Background Scripts, Business Rule, Scheduled Job, etc.).
