# testr.js

Unit testing require.js modules, with both stubbed and script-loaded dependencies.
Compatible with all test frameworks - Jasmine, QUnit, JsTestDriver, Buster JS, etc.
Distributed under the MIT license.

### Usage

Create a new instance of a module under test using the testr method:

```javascript
testr('path/to/module', stubs);
testr('path/to/module', useExternal);
testr('path/to/module', stubs, useExternal);
```

**path/to/module**: the requirejs path name of the module to be unit tested.

**stubs**: *(optional)* a collection of stubs to use in place of dependencies. Each key is the requirejs path name of the module to be stubbed; each value is the module object to use as the stub.

**useExternal**: *(optional)* a boolean to indicate if you wish to load in stubs from an external file. See the *Setup* section for details on where the external stub files should be placed.

### Setup

Include the requirejs script before testr.js, and be sure to have a valid `data-main` attribute that points to your application's top-level require call. Once all source code has been loaded, testr.js will automatically attempt to load all spec and external stub files. These will use an identical path, with a `baseUrl` of `spec` or `stub`. For example:

> **Source**: /src/path/to/module.js
> **Spec**: /src/path/to/module.spec.js
> **Stub**: /src/path/to/module.stub.js

*Note: If the spec or stub file does not exist, this will result in a 404 error. Feel free to fork this project and suppress these, if possible.*

### Example

The module under test is described below.

```javascript
// requirejs path: 'today'
// default string.format.style: 'long'

define(['string', 'util/date'], function(string, date) {
	return {
		getDateString: function() {
			return string.format('Today is %d', date.today);
		},
		setFormat: function(form) {
			string.format.style = form;
		},
		getFormat: function() {
			return string.format.style;
		}
	}
});
```

Testing the *today* module, while stubbing the *date* module and using the actual *string* implementation.

```javascript
var date, today, output, passed;

// stubbing
date = {};
date.today = new Date(2012, 3, 30);

// module instancing
today = testr('today', {'util/date': date});

// testing
output = today.getDateString();
passed = (output === 'Today is Monday, 30th April, 2012');
console.log('User-friendly date: ' + (passed ? 'PASS' : 'FAIL'));
```

#### Using Jasmine BDD

```javascript
describe('Today print', function() {

	var date = {}, today;

	beforeEach(function() {
		date.today = new Date(2012, 3, 30);
		today = testr('today', {'util/date': date});	
	});

	it('is user-friendly', function() {
		expect(today.getDateString()).toBe('Today is Monday, 30th April, 2012');
	});

	it('updates the print format', function() {
		date.today = new Date(2012, 2, 30);
		today.setFormat('short');
		today.polluted = true;
		expect(today.getDateString()).toBe('Today is Fri, 30 Mar 12');
		expect(today.getFormat()).toBe('short');
	});

	it('is not polluted', function() {
		expect(today.polluted).toBeUndefined();
		expect(today.getDateString()).toBe('Today is Monday, 30th April, 2012');
		expect(today.getFormat()).toBe('long');
	});

});
```

### Projects using testr.js

#### [asq](https://github.com/mattfysh/asq)

Wrap a 'one-at-a-time' queue around an asynchronous function.

#### [after](https://github.com/mattfysh/after)

Another promises implementation.