# ServiceNow REST Integration Scaffold

Scaffold a REST API integration Script Include for a specific third-party system inside the NeedIt app.

Arguments: $ARGUMENTS
(Expected: `<SystemName> <base URL or API description>`)

## App Context
- Scope: `x_58872_needit` | App sys_id: `6ead8e780f603200cd674f8ce1050ed1`
- JS level: `helsinki_es5` — strict ES5, no arrow functions, no `let`/`const`, no template literals

## ServiceNow HTTP API
Use `RESTMessageV2` (not XMLHttpRequest, not fetch):
```javascript
var rm = new sn_ws.RESTMessageV2();
rm.setEndpoint(url);
rm.setHttpMethod('GET');                          // GET POST PUT PATCH DELETE
rm.setRequestHeader('Content-Type', 'application/json');
rm.setRequestHeader('Authorization', 'Bearer ' + token);
rm.setBasicAuth(username, password);              // for Basic auth
rm.setRequestBody(JSON.stringify(payload));       // for POST/PUT
rm.setTimeout(30000);

var response = rm.execute();
var statusCode = response.getStatusCode();
var body = response.getBody();
var parsed = JSON.parse(body);
```

## Auth patterns to use based on the system
- **Basic**: `rm.setBasicAuth(user, pass)`
- **API Key**: `rm.setRequestHeader('X-API-Key', key)` (or whatever header the API uses)
- **Bearer/OAuth**: `rm.setRequestHeader('Authorization', 'Bearer ' + token)`

## What to produce

Create `<SystemName>IntegrationUtil` with this structure:

```javascript
var SystemNameIntegrationUtil = Class.create();
SystemNameIntegrationUtil.prototype = {

    initialize: function(config) {
        // Required config fields: baseUrl, authType
        // Optional: timeout, maxRetries, dryRun
        this.baseUrl    = config.baseUrl;
        this.authType   = config.authType || 'none';
        this.timeout    = config.timeout  || 30000;
        this.maxRetries = config.maxRetries || 3;
        this.dryRun     = config.dryRun === true;
        this._credential = config.credential || '';  // token / apikey / password
        this._username   = config.username   || '';
    },

    // Public methods tailored to the system's actual API resources
    // e.g. getUsers(), getHosts(), getAlerts(), syncRecords()

    // Always return a result object, never throw:
    // { success: true/false, data: [...], statusCode: 200, error: null }

    _request: function(method, path, body, extraHeaders) {
        var url = this.baseUrl + path;
        var rm = new sn_ws.RESTMessageV2();
        rm.setEndpoint(url);
        rm.setHttpMethod(method);
        rm.setRequestHeader('Content-Type', 'application/json');
        rm.setRequestHeader('Accept', 'application/json');
        rm.setTimeout(this.timeout);
        this._applyAuth(rm);
        if (extraHeaders) {
            for (var h in extraHeaders) {
                rm.setRequestHeader(h, extraHeaders[h]);
            }
        }
        if (body) { rm.setRequestBody(JSON.stringify(body)); }
        try {
            var response = rm.execute();
            var code = response.getStatusCode();
            var responseBody = response.getBody();
            var parsed = null;
            try { parsed = JSON.parse(responseBody); } catch(e) {}
            return {
                success: code >= 200 && code < 300,
                statusCode: code,
                data: parsed,
                raw: responseBody,
                error: null
            };
        } catch(e) {
            gs.error('SystemNameIntegrationUtil._request error: ' + e.message);
            return { success: false, statusCode: 0, data: null, raw: '', error: e.message };
        }
    },

    _applyAuth: function(rm) {
        if (this.authType === 'basic') {
            rm.setBasicAuth(this._username, this._credential);
        } else if (this.authType === 'bearer') {
            rm.setRequestHeader('Authorization', 'Bearer ' + this._credential);
        } else if (this.authType === 'apikey') {
            rm.setRequestHeader('X-API-Key', this._credential);
        }
    },

    type: 'SystemNameIntegrationUtil'
};
```

Then add meaningful public methods based on the system's actual API (look up or infer from the description provided in $ARGUMENTS).

Generate both files:
- `update/sys_script_include_<system>_integration_util.xml` (XML wrapper)
- `scripts/<system>-integration-util.js` (standalone reference with stubs)

Use a fresh random 32-char hex sys_id.

After creating the files, show the user a **usage example**:
```javascript
var client = new x_58872_needit.SystemNameIntegrationUtil({
    baseUrl: 'https://api.example.com',
    authType: 'bearer',
    credential: 'YOUR_TOKEN'
});
var result = client.getUsers();
if (result.success) {
    gs.info('Got ' + result.data.length + ' users');
}
```

And remind them: import XML via **System Update Sets → Retrieved Update Sets → Import Update Set from XML**.
