# React Request

A library for making HTTP requests with React. Inspired by React Apollo.

### Motivation

[React Apollo](https://github.com/apollographql/react-apollo) exports
[a higher-order component](https://www.apollographql.com/docs/react/basics/setup.html#graphql)
that makes network requests for you. Although Apollo is designed to work
specifically with [GraphQL](http://graphql.org), many of the features of React
Apollo make sense outside of that context.

This library abstracts those features into a generic HTTP component.

### Features

✓ Uses the native [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) API  
✓ Smart [deduping of requests](./docs/guides/request-deduplication.md)  
✓ Customizable [response caching](./docs/guides/response-caching.md)  
✓ Expressive API for parallel requests  
✓ Small footprint (~2kb gzipped)

### Installation

This package is not currently distributed on npm, but it will be shortly.

### Documentation

* [Getting Started](#getting-started)
* [API](#api)
  * [\<Fetch/\>](#fetch-)
  * [\<FetchComposer/\>](#fetchcomposer-)
* [Guides ⇗](./docs/guides/INDEX.md)
  * [Why JSX? ⇗](./docs/guides/why-jsx.md)
  * [Using the `lazy` Prop ⇗](./docs/guides/using-the-lazy-prop.md)
  * [Aborting ⇗](./docs/guides/aborting.md)
  * [Differences with `fetch()` ⇗](./docs/guides/differences-with-fetch.md)
  * [Differences with React Apollo](./docs/guides/differences-with-react-apollo.md)
  * [Request Keys ⇗](./docs/guides/request-keys.md)
  * [Response Caching ⇗](./docs/guides/response-caching.md)
  * [Request Deduplication ⇗](./docs/guides/request-deduplication.md)
  * [Best Practices ⇗](./docs/guides/best-practices.md)
  * [Integration with Technologies ⇗](./docs/guides/integration-with-technologies.md)
* [Examples ⇗](./docs/examples.md)
* [FAQ ⇗](./docs/FAQ.md)
* [Acknowledgements](#acknowledgements)

### Getting Started

Here's a quick look at what using React Request is like:

```js
import React, { Component } from 'react';
import { Fetch } from 'react-request';

class App extends Component {
  render() {
    return (
      <Fetch
        url="https://jsonplaceholder.typicode.com/posts/1"
        render={({ fetching, error, data }) => {
          if (fetching) {
            return <div>Loading data...</div>;
          }

          if (error) {
            return <div>There was a network error.</div>;
          }

          return (
            <div>
              <div>Post ID: {data.id}</div>
              <div>Post Title: {data.title}</div>
            </div>
          );
        }}
      />
    );
  }
}
```

Need to make multiple requests? We got you.

```js
import React, { Component } from 'react';
import { FetchComposer } from 'react-request';

class App extends Component {
  render() {
    return (
      <FetchComposer
        requests={[
          <Fetch url="https://jsonplaceholder.typicode.com/posts/1" />,
          <Fetch
            url="https://jsonplaceholder.typicode.com/posts/1"
            method="DELETE"
          />
        ]}
        render={([readPost, deletePost]) => {
          return (
            <div>
              {readPost.fetching && 'Loading post 1'}
              {!readPost.fetching && 'Post 1 is not being fetched'}
              <button onClick={() => deletePost.fetch()}>Delete Post 1</button>
            </div>
          );
        }}
      />
    );
  }
}
```

These examples just scratch the surface of what you can do with React Request.
Check out the API reference below for more.

### API

This library has two exports:

* `Fetch`: A component for making a single HTTP request
* `FetchComposer`: A component for making parallel HTTP requests

#### `<Fetch />`

A component for making a single HTTP request. It accepts every value of `init` and `input`
from the
[`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch)
API as a prop, in addition to a few other things.

The props that come from the `fetch()` method are:

* `url`
* `method`: defaults to `"GET"`
* `body`
* `credentials`
* `headers`
* `mode`
* `cache`
* `redirect`
* `referrer`: defaults to `"about:client"`
* `referrerPolicy`: defaults to `""`
* `integrity`: defaults to `""`
* `keepalive`
* `signal`

To learn more about the valid options for these props, refer to the
[`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch)
documentation.

Here's an example demonstrating some of the most commonly-used props:

```jsx
<Fetch
  url="/posts/2"
  method="patch"
  credentials="same-origin"
  headers={{
    'csrf-token': myCsrfToken
  }}
  body={JSON.stringify({ title: 'New post' })}
  render={({ fetch }) => {
    <button onClick={() => fetch()}>Update Post</button>;
  }}
/>
```

In addition to the `fetch()` props, there are a number of other useful props.

##### `render`

The [render prop](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) of this component.
It is called with one argument, `result`, an object with the following keys:

* `fetching`: A Boolean representing whether or not a request is currently in flight for this component
* `error`: A Boolean representing if a network error occurred. Note that HTTP "error" status codes do not
  cause `error` to be `true`; only failed or aborted network requests do. For more, see the
  ["Using Fetch" MDN guide](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch#Checking_that_the_fetch_was_successful).
* `response`: An instance of [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response). The
  [`body`](https://developer.mozilla.org/en-US/docs/Web/API/Body) will already be read, and made
  available to you as `response.data`.
* `data`: An alias of `response.data`.
* `fetch`: A function that makes the HTTP request. See notes below.
* `url`: The URL that was passed into `<Fetch />`.
* `requestName`: The name of the request (see `requestName` below)

There are three common use cases for the `fetch` prop:

* For GET requests, it can allow users to refresh the data
* Anytime there is a network error, you can use this function to retry the request
* When `lazy` is `true`, you can use this to actually make the request, typically as
  a result of user input

`fetch` accepts one argument: `options`. Any of the `fetch()` options described above are valid
`options`. This allows you to customize the request from within the component.

##### `lazy`

Whether or not the request will be called when the component mounts. The default value
is based on the request method that you use.

| Method                   | Default value |
| ------------------------ | ------------- |
| GET, HEAD, OPTIONS       | `false`       |
| POST, PUT, PATCH, DELETE | `true`        |

```jsx
<Fetch
  url="/books"
  lazy
  render={({ fetch }) => {
    <button onClick={() => fetch()}>Fetch books</button>;
  }}
/>
```

##### `onResponse`

A function that is called every time a response is received. Receives two arguments: `error` and `response`.
This could be used for analytics or syncing response data with a store such as [Redux](https://github.com/reactjs/redux/).

```jsx
<Fetch
  url="/posts/2"
  onResponse={(error, response) => {
    if (error) {
      console.log('Ruh roh', error);
    } else {
      console.log('Got a response!', response);
    }
  }}
  render={() => {
    <div>Hello</div>;
  }}
/>
```

##### `transformData`

A function that is called with the data returned from the response. You can use this
hook to transform the data before it is passed into `render`.

```jsx
<Fetch
  url="/posts/2"
  transformData={data => {
    return data.post;
  }
  render={({ fetching, error, response, data }) => {
    <div>
      {fetching && ('Loading...')}
      {error && ('There was an error.')}
      {!fetching && !error && response.status === 200 && (
        <div>
          <h1>{data.title}</h1>
          <div>{data.content}</div>
        </div>
      )}
    </div>
  }}
/>
```

##### `responseType`

The content type of the response body. Defaults to `json`. Valid values are the methods
on [Body](https://developer.mozilla.org/en-US/docs/Web/API/Body).

```jsx
// If you have an endpoint that just returns raw text, you can convert it into
// an object using `responseType` and `transformData`.
<Fetch
  url="/countries/2"
  responseType="text"
  transformData={countryName => {
    return {
      countryName
    };
  }}
  render={({ data }) => {
    <div>{data.countryName}</div>;
  }}
/>
```

##### `requestName`

A name to give this request, which can help with debugging purposes. The request name is
analogous to a function name in JavaScript. Although we could use anonymous functions
everywhere, we tend to give them names to help humans read and debug the code.

```jsx
<Fetch url={`/posts/${postId}`} requestName="readPost" />
```

> Note: This feature is analogous to the [operation name](http://graphql.org/learn/queries/#operation-name) in GraphQL.

##### `fetchPolicy`

This determines how the request interacts with the cache. Valid options are:

* `"cache-first"`
* `"cache-and-network"`
* `"network-only"`
* `"cache-only"`

For documentation on this prop, refer to the [response caching guide](./docs/guides/response-caching.md).

> This prop behaves identically to the Apollo prop
> [with the same name](https://www.apollographql.com/docs/react/basics/queries.html#graphql-config-options-fetchPolicy).

---

#### `<FetchComposer />`

A component that simplifies making parallel requests.

##### `requests`

An array of `Fetch` components. Use any of the above props, but leave out `render`.

> Note: if you pass a `render` prop, it will be ignored.

##### `render`

A function that is called with the array of responses from `requests`.

```js
import React, { Component } from 'react';
import { FetchComposer } from 'react-request';

class App extends Component {
  render() {
    const { bookId, authorId } = this.props;

    return (
      <FetchComposer
        requests={[
          <Fetch url={`/books/${bookId}`} />,
          <Fetch url={`/authors/${authorId}`} />,
          <Fetch url={`/books/${bookId}`} method="DELETE" />
        ]}
        render={([readBook, readAuthor, deleteBook]) => {
          return (
            <div>
              {readBook.fetching && 'Fetching book'}
              {readAuthor.fetching && 'Fetching author'}
              {!readBook.fetching && 'Book not being fetched'}
              {!readAuthor.fetching && 'Author not being fetched'}
              <button onClick={() => deleteBook.fetch()}>
                Delete this book
              </button>
            </div>
          );
        }}
      />
    );
  }
}
```

### Acknowledgements

This library was inspired by [Apollo](https://www.apollographql.com). The
library [Holen](https://github.com/tkh44/holen) was referenced during the
creation of this library.
