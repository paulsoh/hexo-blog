---
title: Organizing fetches in redux actions
date: 2017-10-20 10:34:14
tags: react, es6, javascript, refactoring, fetch
---

### This article assumes that you are:
 * Using redux and redux thunk
 * Using fetch API for making API requests
 * Using jwt tokens stored in localStorage for authorization with the external api server


If you've been working with external CRUD-like apis then your **actionCreators** would probably look something like this:

```javascript
// Action for POSTing Article
const postArticleToServer = (article) => {
  return (dispatch) => {
    dispatch({
      type: 'POST_ARTICLE_REQUEST',
    })

    fetch('http://abc.com/article', {
      header: {
        Authorization: `Bearer ${window
          .localStorage.token}`,
      },
      body: JSON.stringify(payload),
      method: 'POST',
    })
      .then(resp => resp.json())
      .then(result => {
        dispatch({
          type: 'POST_ARTICLE_SUCCESS',
          payload: result,
        })
      })
      .catch(error => {
        dispatch({
          type: 'POST_ARTICLE_FAILED',
          payload: result,
        })
      })
  }
}

// Action for GETing Article
const getArticlesFromServer = () => {
  return (dispatch) => {
    dispatch({
      type: 'GET_ARTICLES_REQUEST',
    })

    fetch('http://abc.com/article', {
      // Fetch uses 'GET' as default
      header: {
        Authorization: `Bearer ${window
          .localStorage.token}`,
      },
    })
      .then(resp => resp.json())
      .then(result => {
        dispatch({
          type: 'GET_ARTICLES_SUCCESS',
          payload: result,
        })
      })
      .catch(error => {
        dispatch({
          type: 'GET_ARTICLES_FAILED',
          payload: result,
        })
      })
  }
}

// Action for PUTing Article
const getArticlesFromServer = (id, payload) => {
  return (dispatch) => {
    dispatch({
      type: 'UPDATE_ARTICLE_REQUEST',
    })

    fetch('http://abc.com/article/' + id, {
      header: {
        Authorization: `Bearer ${window
          .localStorage.token}`,
      },
      body: JSON.stringify(payload),
      method: 'PUT',
    })
      .then(resp => resp.json())
      .then(result => {
        dispatch({
          type: 'UPDATE_ARTICLES_SUCCESS',
          payload: result,
        })
      })
      .catch(error => {
        dispatch({
          type: 'UPDATE_ARTICLES_FAILED',
          payload: result,
        })
      })
  }
}
```

To allow the client to perform simple CRUD tasks for a single resource, we can see that already a lot of code is being repetitive. Especially the `fetch`-ing part. Imagine making actions for several endpoints (such as comments, feeds, contacts ...) 

In order to tidy up these fetch calls, I'll make a ApiClient class that has the following signature:

```javascript
ApiClient(baseUrl, { methods: ['GET', 'SHOW', 'POST', 'PUT', 'DELETE'], isAuthRequired: true })
```

where `methods` specify which methods are allowed by the endpoint.

Then, we could use it like this inside our actionCreators


```javascript
const articleApi = new ApiClient(
  'http://abc.com/article',
  {
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    isAuthRequired: true,
  }
)

// Action for POSTing Article
const postArticleToServer = (article) => {
  return (dispatch) => {
    dispatch({
      type: 'POST_ARTICLE_REQUEST',
    })

    articleApi.post(payload)
      .then(result => {
        dispatch({
          type: 'POST_ARTICLE_SUCCESS',
          payload: result,
        })
      })
      .catch(error => {
        dispatch({
          type: 'POST_ARTICLE_FAILED',
          payload: result,
        })
      })
  }
}

// Action for GETing Article
const getArticlesFromServer = () => {
  return (dispatch) => {
    dispatch({
      type: 'GET_ARTICLES_REQUEST',
    })

    articleApi.post(payload)
      .then(result => {
        dispatch({
          type: 'GET_ARTICLES_SUCCESS',
          payload: result,
        })
      })
      .catch(error => {
        dispatch({
          type: 'GET_ARTICLES_FAILED',
          payload: result,
        })
      })
  }
}

// Action for PUTing Article
const getArticlesFromServer = (id, payload) => {
  return (dispatch) => {
    dispatch({
      type: 'UPDATE_ARTICLE_REQUEST',
    })

    articleApi.put(id, payload)
      .then(result => {
        dispatch({
          type: 'UPDATE_ARTICLES_SUCCESS',
          payload: result,
        })
      })
      .catch(error => {
        dispatch({
          type: 'UPDATE_ARTICLES_FAILED',
          payload: result,
        })
      })
  }
}
```

The internals of the ApiClient class might look something like this

```javascript
class ApiClient {
  constructor(baseUrl, options) {
    this.baseUrl = baseUrl;
    this.options = options;
    this.header = {};

    if (options.isAuthRequired) {
      this.header = {
        Authorization: `Bearer ${localStorage.token}`
      }
    }
  }

  get = () => {
    // If 'GET' is not specified in `methods` 
    if (!this.options.methods.includes('GET')) {
      raise Error('GET is not supported')
    }

    const endpoint = this.baseUrl;
    return fetch(endpoint, {
      headers: this.header,
    }).then((resp) => resp.json())
  }

  show = (id) => {
    // If 'SHOW' is not specified in `methods` 
    if (!this.options.methods.includes('SHOW')) {
      raise Error('SHOW is not supported')
    }

    const endpoint = this.baseUrl + id;
    return fetch(endpoint, {
      headers: this.header,
    }).then((resp) => resp.json())
  }

  post = (payload) => {
    // If 'POST' is not specified in `methods` 
    if (!this.options.methods.includes('POST')) {
      raise Error('POST is not supported')
    }

    const endpoint = this.baseUrl;
    return fetch(endpoint, {
      headers: this.header,
      method: 'POST',
      body: JSON.stringify(payload)
    }).then((resp) => resp.json())
  }

  put = (id, payload) => {
    // If 'PUT' is not specified in `methods` 
    if (!this.options.methods.includes('PUT')) {
      raise Error('PUT is not supported')
    }

    const endpoint = this.baseUrl + id;
    return fetch(endpoint, {
      headers: this.header,
      method: 'PUT',
      body: JSON.stringify(payload)
    }).then((resp) => resp.json())
  }
  ...
}
```
