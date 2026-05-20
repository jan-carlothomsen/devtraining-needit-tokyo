# ServiceNow CMDB Operations

Write CMDB-related GlideRecord code for the NeedIt app.

Arguments: $ARGUMENTS
(Describe what you need: upsert a CI type, build relationships, query CIs, etc.)

## App Context
- Scope: `x_58872_needit` | JS level: `helsinki_es5`
- `var` only, prototype pattern, no arrow functions, no `let`/`const`

## Key CMDB Tables
| Purpose | Table |
|---|---|
| Servers / VMs | `cmdb_ci_server` |
| Applications | `cmdb_ci_appl` |
| Business Services | `cmdb_ci_service` |
| Computers / Endpoints | `cmdb_ci_computer` |
| Database instances | `cmdb_ci_db_instance` |
| Network devices | `cmdb_ci_netgear` |
| CI Relationships | `cmdb_rel_ci` |
| Relationship types | `cmdb_rel_type` |

## Common CI Fields
```
name, ip_address, os, os_version, manufacturer, model_id,
serial_number, cpu_count, ram, install_status, operational_status,
discovery_source, correlation_id, location, assignment_group
```

## Install Status Values
`1=Installed | 2=On Order | 3=In Maintenance | 6=In Stock | 7=Retired | 8=Stolen`

## GlideRecord Patterns

### Upsert (create or update)
```javascript
var gr = new GlideRecord('cmdb_ci_server');
gr.addQuery('name', ciName);
gr.query();
if (gr.next()) {
    gr.setValue('ip_address', ipValue);
    gr.setValue('os', osValue);
    gr.update();
} else {
    gr.initialize();
    gr.setValue('name', ciName);
    gr.setValue('ip_address', ipValue);
    gr.setValue('os', osValue);
    gr.setValue('discovery_source', 'x_58872_needit');
    gr.insert();
}
```

### Create a CI Relationship
```javascript
var relType = new GlideRecord('cmdb_rel_type');
relType.addQuery('name', 'Runs on::Runs');
relType.query();
relType.next();
var relTypeSysId = relType.sys_id.toString();

var rel = new GlideRecord('cmdb_rel_ci');
rel.initialize();
rel.setValue('parent', parentCiSysId);
rel.setValue('child', childCiSysId);
rel.setValue('type', relTypeSysId);
rel.insert();
```

### Common Relationship Type Names
```
Depends on::Used by | Hosted on::Hosts | Runs on::Runs
Connected by::Connects | Contains::Contained by | Members::Member of
```

### Retire a CI
```javascript
var gr = new GlideRecord('cmdb_ci_server');
if (gr.get(ciSysId)) {
    gr.setValue('install_status', 7);
    gr.setValue('operational_status', 6);
    gr.update();
}
```

## What to produce

Output **only the JavaScript** — no XML, no file wrappers.

If the result is a reusable utility, structure it as a Script Include class (user pastes it into a new Script Include). If it is a one-off operation, structure it as a plain script the user can run in **System Definition → Scripts - Background**.

Always:
- Use `discovery_source = 'x_58872_needit'` on any CI you create
- Avoid full table scans — use indexed fields (`name`, `ip_address`, `sys_id`, `correlation_id`)
- Return result objects `{ success, sysId, action, error }` from Script Include methods
