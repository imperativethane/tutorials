# Creating Testing Framework

## NPM Packages

### Chai
Provides an advanced assert library

```javascript
const {assert} = require('chai);
```

Example use:

```javascript
assert.equal(response.status, 200);
```

### SuperTest 
Provides the functionality required to test APIs. Can make get/post requests.

```javascript
const request = require('supertest');
```
Example use:

```javascript
const response = await request(app).
get('/');
```

### JSDOM 
Allows you to create the parseTextFromHTML helper function which will take your response and all you to find the text that is to be rendered within the HTML.

```javascript
const {jsdom} = require(‘jsdom’);
```

Helper function: 

```javascript
const parseTextFromHTML = (htmlAsString, selector) => {
  const selectedElement = jsdom(htmlAsString).querySelector(selector);
  if (selectedElement !== null) {
    return selectedElement.textContent;
  } else {
    throw new Error(`No element with selector ${selector} found in HTML string`);
  }
};
```

## Feature Testing/Browser Testing

## Server Testing

### Test status codes - httpstatuses.com

```javascript
describe('root page', () => {
describe('GET request', () => {
it('returns a 200 status', async () => {
const response = await request(app).
get('/');
assert.equal(response.status, 200);
});
});
});
```

### Response content
Many servers return dynamic HTML content based on the user, the URL accessed, header values, and more. We use TDD to ensure the server responds correctly for each case. 

Need to create tests that:

1. Exercise the ‘happy path’ – the expected use cases of the application.
1. Exercise the ‘sad path’ – unexpected or invalid use of the application.




