# ServiceNow REST Integration Scaffold

Scaffold a REST API integration Script Include for a specific third-party system.

Arguments: $ARGUMENTS
(Expected: `<SystemName> <base URL or API description>`)

## App Context
- Scope: `x_58872_needit` | JS level: `helsinki_es5` — strict ES5 only
- No `let`/`const`, no arrow functions, no template literals, no `class`

## ServiceNow HTTP API — use RESTMessageV2
```javascript
var rm = new sn_ws.RESTMessageV2();
rm.setEndpoint(url);
rm.setHttpMethod('GET');                           // GET POST PUT PATCH DELETE
rm.setRequestHeader('Content-Type', 'application/json');
rm.setRequestHeader('Accept', 'application/json');
rm.setBasicAuth(username, password);               // Basic auth
rm.setRequestHeader('Authorization', 'Bearer ' + token); // Bearer auth
rm.setRequestHeader('X-API-Key', key);             // API Key auth
rm.setRequestBody(JSON.stringify(payload));         // POST/PUT body
rm.setTimeout(30000);

var response = rm.execute();
var statusCode = response.getStatusCode();
var body = response.getBody();
```

## Base Structure to follow
```javascript
var SystemNameIntegrationUtil = Class.create();
SystemNameIntegrationUtil.prototype = {

    initialize: function(config) {
        this.baseUrl     = config.baseUrl;
        this.authType    = config.authType || 'none'; // 'basic'|'bearer'|'apikey'|'none'
        this.timeout     = config.timeout  || 30000;
        this.dryRun      = config.dryRun === true;
        this._credential = config.credential || '';
        this._username   = config.username   || '';
    },

    // Add public methods for the system's actual resources here

    _request: function(method, path, body) {
        var rm = new sn_ws.RESTMessageV2();
        rm.setEndpoint(this.baseUrl + path);
        rm.setHttpMethod(method);
        rm.setRequestHeader('Content-Type', 'application/json');
        rm.setRequestHeader('Accept', 'application/json');
        rm.setTimeout(this.timeout);
        this._applyAuth(rm);
        if (body) { rm.setRequestBody(JSON.stringify(body)); }
        try {
            var response = rm.execute();
            var code = response.getStatusCode();
            var responseBody = response.getBody();
            var parsed = null;
            try { parsed = JSON.parse(responseBody); } catch(e) {}
            return { success: code >= 200 && code < 300, statusCode: code, data: parsed, error: null };
        } catch(e) {
            gs.error('SystemNameIntegrationUtil error: ' + e.message);
            return { success: false, statusCode: 0, data: null, error: e.message };
        }
    },

    _applyAuth: function(rm) {
        if (this.authType === 'basic')  { rm.setBasicAuth(this._username, this._credential); }
        if (this.authType === 'bearer') { rm.setRequestHeader('Authorization', 'Bearer ' + this._credential); }
        if (this.authType === 'apikey') { rm.setRequestHeader('X-API-Key', this._credential); }
    },

    type: 'SystemNameIntegrationUtil'
};
```

## What to produce

Output **only the JavaScript** — no XML, no file wrappers.

The user pastes this into a new Script Include in ServiceNow:
- **Name**: `SystemNameIntegrationUtil`
- **API Name**: `x_58872_needit.SystemNameIntegrationUtil`
- **Script**: the JavaScript

Build meaningful public methods based on the actual API described in the argument.

After the code, show a usage example the user can run in a Background Script to test it.
