# ServiceNow XML Update Set Wrapper

Wrap existing JavaScript code into a valid ServiceNow Script Include XML update set record, ready to import.

Arguments: $ARGUMENTS
(Provide the class name, or point to an existing `.js` file. If neither, ask for the JavaScript to wrap.)

## App Context
- Scope: `x_58872_needit` | App sys_id: `6ead8e780f603200cd674f8ce1050ed1`

## What to do

1. Read the JavaScript (from file or from the argument / conversation context)
2. Strip any local-testing stubs (the `if (typeof gs === 'undefined')` blocks) — those must NOT go into the XML
3. Validate ES5 compliance (no `let`/`const`, no arrow functions, no template literals — flag any violations)
4. Determine the class name from `ClassName.prototype = {` or from the argument
5. Generate a unique 32-char lowercase hex sys_id
6. Produce the XML file at `update/sys_script_include_<snake_case_name>.xml`

## XML Template
```xml
<?xml version="1.0" encoding="UTF-8"?>
<record_update table="sys_script_include">
    <sys_script_include action="INSERT_OR_UPDATE">
        <access>public</access>
        <active>true</active>
        <api_name>x_58872_needit.ClassName</api_name>
        <client_callable>false</client_callable>
        <description>Infer a one-line description from the code</description>
        <name>ClassName</name>
        <script><![CDATA[
JAVASCRIPT_GOES_HERE
        ]]></script>
        <sys_class_name>sys_script_include</sys_class_name>
        <sys_id>GENERATED_32_CHAR_HEX_SYS_ID</sys_id>
        <sys_package display_value="NeedIt" source="x_58872_needit">6ead8e780f603200cd674f8ce1050ed1</sys_package>
        <sys_scope display_value="NeedIt">6ead8e780f603200cd674f8ce1050ed1</sys_scope>
        <sys_update_name>sys_script_include_GENERATED_32_CHAR_HEX_SYS_ID</sys_update_name>
    </sys_script_include>
</record_update>
```

## Rules
- `sys_id` and `sys_update_name` must use the **same** generated hex ID
- JavaScript inside `<![CDATA[...]]>` — do NOT XML-escape the JS content
- Do NOT include tracked-only fields: `sys_created_by`, `sys_created_on`, `sys_updated_by`, `sys_updated_on`, `sys_mod_count`
- If the class name contains `Ajax` or the code calls `AbstractAjaxProcessor`, set `<client_callable>true</client_callable>` instead

## After creating the XML

Remind the user:
1. Validate with: `xmllint --noout update/sys_script_include_<name>.xml`
2. Import via: **ServiceNow → System Update Sets → Retrieved Update Sets → Import Update Set from XML**
3. Preview the update set → Commit

If any ES5 violations were found in step 3, list them and offer to fix them before writing the XML.
