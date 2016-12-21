# ngx-http-test
[![Build Status](https://travis-ci.org/blacksonic/ngx-http-test.svg?branch=master)](https://travis-ci.org/blacksonic/ngx-http-test)
[![Dependency Status](https://david-dm.org/blacksonic/ngx-http-test.svg)](https://david-dm.org/blacksonic/ngx-http-test)
[![devDependency Status](https://david-dm.org/blacksonic/ngx-http-test/dev-status.svg)](https://david-dm.org/blacksonic/ngx-http-test?type=dev)
[![peerDependency Status](https://david-dm.org/blacksonic/ngx-http-test/peer-status.svg)](https://david-dm.org/blacksonic/ngx-http-test?type=peer)

Makes testing Http calls in Angular 2 as easy as were in AngularJS.

### Installation

```bash
npm install angular2-http-testing --save-dev
```

### Usage

```typescript
// First add it as a provider
beforeEach(() => {
  TestBed.configureTestingModule({
    providers: [
      ...
      FakeBackend.getProviders()
    ]
  });
});

// Get the instance of it
beforeEach(inject([..., FakeBackend], (..., fakeBackend: FakeBackend) => {
  backend = fakeBackend;
}));

// Use it in a test case
it('should call fake endpoint', (done) => {
  backend.expectGET('users/blacksonic').respond({ username: 'blacksonic' });
  
  subject.getProfile('blacksonic').subscribe((response) => {
    expect(response).toEqual({ username: 'blacksonic' });
    done();
  });
})
```

It is possible to specify every detail of the response.

```typescript
// can be object, it will be json stringified if not a string
let responseBody = { username: 'blacksonic' };
let responseStatus = 200;
let responseHeaders = { 'Content-Type': 'application/json' };

backend
  .expectGET('users/blacksonic')
  .respond(responseBody, responseStatus, responseHeaders);
```

It is not necessary to give the response, in that case the backend will respond with 200 empty response.

Also the expected url can be given as a regular expression instead of a string.

For requests outside of GET the request body can be also specified.

```typescript
backend.expectPost(
  'usernamepassword/login', // url
  { username: 'blacksonic', password: 'secret' }, // payload
  { 'Content-Type': 'application/json' } // headers
).respond(responseForm);
```

Convenience methods available for different kind of requests: 
```expectGet```, ```expectPost```, ```expectDelete```, ```expectPut```, ```expectPatch```, ```expectHead```, ```expectOptions```.

After the tests run it is possible to check for outstanding connections or expectations that are not addressed.

```typescript
afterEach(() => {
  backend.verifyNoPendingRequests();
  backend.verifyNoPendingExpectations();
});
```

By default expectations get verified instantly, but this can be switched off and do the verification by hand.

```typescript
it('should call fake endpoint', (done) => {
  backend.setAutoRespond(false);
  backend.expectGET('users/blacksonic').respond({ username: 'blacksonic' });
  
  subject.getProfile('blacksonic').subscribe((response) => {
    expect(response).toEqual({ username: 'blacksonic' });
    done();
  });
  
  backend.flush();
})
```

The ```flush``` method resolves all pending connections. It can also be resolved one by one with the ```flushNext``` method.
