# Creating Testing Framework

## NPM Packages

### Mocha 

Popular testing framework that operates on Node.js. Provides compatibility with both front-end and back-end asynchronous testing.

### Chai
Provides an advanced assertion library.

```javascript
const {assert} = require('chai);
```

Example use:

```javascript
assert.equal(response.status, 200);
```

### PhantomJS
Headless browser scriptable with a JavaScript API that allows us to write tests that mimic user interaction.
Means we can avoid rendering the application in the browser.

### WebdriverI/O

WebdriverIO provides methods that allow us to programmatically interact with the user-facing elements of our app in the headless browser that PhantomJS runs.

Needs to be saved as a devDependency in package.json

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

Feature tests exercise behaviour by simulating a user navigating the application in a web browser.

#### Example One

```javascript
describe('User visits root', () => {

  describe('without existing messages', () => {
    it('starts blank', () => {
      browser.url('/');

      assert.equal(browser.getText('#messages'), '');
    });
  });
});
```

In this example we are using:
* Chai assert library
* WebdriverI/O
    1. The .getText method gets the text content from the selected DOM element
    2. browser is a global variable provided by WebdriverI/O

#### Example Two

```javascript
const {assert} = require('chai');

describe('User visits root', () => {

  describe('posting a message', () => {
    it('saves the message with the author information', () => {
      
      const message ='feature tests often hit every level of the TDD Testing Pyramid';
      const author = 'username';

      browser.url('/');
      browser.setValue('input[id=author]', author);
      browser.setValue('textarea[id=message]', message);
      browser.click('input[type=submit]');

      assert.include(browser.getText('#messages'), author);
      assert.include(browser.getText('#messages'), message);
    });
  });
});
```

Once you get to a test that won't pass then it's time to move to the server testing layer!

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

#### Happy Path

##### Example One

```javascript
    it('contains the correct title', async () => {
      const response = await request(app).
      get('/');
      assert.equal(parseTextFromHTML(response.text, '#page-title'), 'Messaging App');
    });
```
##### Example Two
```javascript
describe('POST', () => {
    describe('when the Message is valid', () => {
      it('redirects to the index', async () => {
        const author = 'Inquisitive User';
        const message = 'Why Test?';

        const response = await request(app)
          .post('/messages')
          .type('form')
          .send({author, message});

        assert.equal(response.status, 302);
        assert.equal(response.headers.location, '/');
      });
    });
});
```

#### Sad Path 

We need to make sure our server properly handles invalid passwords, form field errors, etc.

While there may only be one happy path there is multiple sad paths and it makes more sense to test these at a server level.

##### Example
```javascript
describe('POST', () => {
    describe('when the author is blank', () => {
        it('renders an error message', async () => {
        const message = 'Server Testing';

        const response = await request(app)
            .post('/messages')
            .send({message});
        assert.equal(response.status, 400);
        assert.equal(JSON.parse(response.text).message, 'Every message requires an author');
        });
    });
});
```



