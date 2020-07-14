## Mini Movie App

We're going to build a tiny React single page app that has a text input, a button and when a user inputs a movie, they will get a set of movies that match from OMDB.

### Setup

In student_examples
- `mkdir movies`
- `cd movies`
- `touch app.js index.html`
- `atom .`

**Index.html**
- html boilerplate
- link React scripts and Babel
```
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title>Movie App</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/16.3.2/umd/react.production.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/16.3.2/umd/react-dom.production.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.26.0/babel.min.js"></script>
    <script type="text/babel" src="app.js"></script>
  </head>
  <body>
    <div class="container">
      <!-- Let's build our Movie App here -->
    </div>
  </body>
</html>
```

**app.js**

- set up a `ReactDOM.render()` that will render an `App` component that will, for now, just say 'hello world'

```js
class App extends React.Component {
  render () {
    return (
      <div>Hello World</div>
    )
  }
}

ReactDOM.render(
  <App />,
  document.querySelector('.container')
)
```


### Setting Up Our Form

We're going to be making requests to OMDB using `fetch`. We'll be viewing those results in Chrome's Console. Once we build out the functionality, we'll start incorporating our data into our web page.

Go to [request an api key from OMDB](http://www.omdbapi.com/apikey.aspx)

Fill out the form, you should get an email asking you to validate your key within a few minutes. Hold on to this key, we'll be using it soon enough.


An http request requires:
- headers (fetch handles most of this for us)
- a url to send the request
- a method (get, post, put, delete...)
- a way to send data if it is a post or put request
- a way to wait for the response before doing something with the incoming data (We'll use promises, but there is a newer way with async/await that you can research on your own)
- a way to do something with the incoming data
- a way to handle errors

Let's build our first fetch request.

`fetch` is a function. It takes two arguments where the first one is the url and the second one **MUST** be an object ([Doc](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)). We'll code this out together after we put together our query string

The object requires a minimum of two key value pairs
- `method` - a string: `GET`, `POST`, `PUT`, `DELETE`
- `url` - a string of where to send the request

- if the request is sending data (ie input from input fields), a third key value pair is required
- `data`: an object of key value pairs
Since we are just sending GET requests, we won't need `data` for now - We'll build out our `fetch` request after we set up our form.

**app.js**

Let's build our form and form functionality. We've done this before and it seems like a lot of boilerplate functionality. Because react has reusable components, you could just build this functionality once per project. But you may find that you want different behaviors for different inputs.

We're going to break our request into multiple parts and then concatenate it. This will help us see each piece and what it does, and then possibly, help us have more flexibility as we build more complex/varied requests

- `baseURl` : `'http://www.omdbapi.com/?'` the start of our request beginning with `http://`, after the last `/` we have a question mark, that will start our query parameters
- `apikey`: `'apikey='` + `'your api key here'` `apikey` is the specific key OMDB is looking for, then we'll add our own api key (no spaces).
- `query`: `'&t='` the ampersand `&` lets us know that there is another query parameter coming up. `t=` is the next key, it matches the type of search we want to do. There are a few searches in the OMDB documentation. Perhaps later on, we'd want to use a different one?.
- `movieTitle`: `''` - the movie title we'd like to search for. We'll be able to enter what we want using our input and form
-   `searchURL:` `''` - here we'll end up concatenating all the pieces to make a working url.

```js
constructor (props) {
  super(props)
  this.state = {
    baseURL: 'http://www.omdbapi.com/?',
    apikey: 'apikey=' + '98e3fb1f',
    query: '&t=',
    movieTitle: '',
    searchURL: ''
  }
}
```

Next we'll have to add our form(and we'll add an anchor tag at the bottom, if we've successfully built our URL, we should be able to click on it and see our JSON from OMBD):

```js
render () {
    return (
      <React.Fragment>
        <form onSubmit={this.handleSubmit}>
          <label htmlFor='movieTitle'>Title</label>
          <input
            id='movieTitle'
            type='text'
            value={this.state.movieTitle}
            onChange={this.handleChange}
          />
          <input
            type='submit'
            value='Find Movie Info'
          />
        </form>
        <a href={this.state.searchURL}>{this.state.searchURL}</a>
      </React.Fragment>
    )
  }
```

Finally, we have to add our `handleChange` and `handleSubmit` functionality. And let's not forget to bind those in our constructor

```js
constructor (props) {
  super(props)
  this.state = {
    baseURL: 'http://www.omdbapi.com/?',
    apikey: 'apikey=' + '98e3fb1f',
    query: '&t=',
    movieTitle: '',
    searchURL: ''
  }
  this.handleSubmit = this.handleSubmit.bind(this)
  this.handleChange = this.handleChange.bind(this)
}
handleChange (event) {
  this.setState({ [event.target.id]: event.target.value })
}
handleSubmit (event) {
  event.preventDefault()
  this.setState({
    searchURL: this.state.baseURL + this.state.apikey + this.state.query +  this.state.movieTitle
  })
}
```





our searchURL should look like (the api key is fake, it should be the one you got via email).



```js
http://www.omdbapi.com/?apikey=9999999&t=Eraserhead
```

When you click on your anchor tag, you should be taken to the JSON view:

![OMDB response](https://i.imgur.com/ojU0Qrp.png)


Note: including your API key in the `app.js` and then pushing it up to github makes your API key findable. OMDB keys are not that valuable, so it shouldn't be a worry.

However, there are services that cost thousands of dollars a month. People write bots to look for exposed API keys to steal them. We don't have time to cover hiding API keys today. But keep it in mind for your projects.

### Using Fetch

Rather than render the `a` tag, we want to be able display the data we want on on our web page. Let's start out by getting our JSON from OMDB to console log. When we are doing a get request, all we need to do is put a string as our first argument.

We have a one little GOTCHA - we have to make sure that state has been set, before we make our request. So we'll add our fetch request as a second argument in our `setState` function inside our `handleSubmit`

```js
handleSubmit (event) {
  event.preventDefault()
  this.setState({
    searchURL: this.state.baseURL + this.state.apikey + this.state.query + this.state.movieTitle
  }, () => {
    // fetch request will go here
  })
}
```


Let's isolate and look at each part of our fetch request as we build it inside our callback of `setState`

**fetch function**
```js
fetch()
```

**fetch function with url string as argument**
```js
  fetch(this.state.searchURL)
```

Remember, JavaScript just runs through the code top to bottom. It doesn't wait for one thing to finish before moving on to the next thing.

One way we've been handling waiting is by using callbacks (like we just did with `setState`). Callbacks can get unwieldy. Promises can be a nicer way to write our code. Often you'll be working with promises that were already created for you from a library or framework (like `fetch`)

A promise is a way of writing code that says 'hey, wait for me, and I _promise_ I'll send you a response soon. _Then_, you can do what you need to do'

A promise has three states
- Pending
- Fulfilled (completed successfully)
- Rejected (it failed)

After calling the initial function (in our case `fetch()`), all we need to do is chain another function `.then()`

`.then()` takes two callback functions.

The first one is `onFulfilled` this one is executed if the promise is fulfilled. This function can do whatever we want, for now, we'll just console.log the response.

The second one is `onRejected` this one is executed instead of the first one if the promise has failed in some way.


**`Then` we need to convert our response to JSON**
```js
  fetch(this.state.searchURL).then(response => return response.json())
```

**Then we will console log our response. We can break our `then()` onto multiple lines to make it easier to read(())**

```js
fetch(this.state.searchURL)
  .then(response => {
    return response.json()
  }).then(json => console.log(json),
      err => console.log(err))
```

Our error handling can be very simple, we can just console log the error.

**The entire `handleSubmit` function**
```js
handleSubmit (event) {
  event.preventDefault()
  this.setState({
    searchURL: this.state.baseURL + this.state.apikey + this.state.query + this.state.movieTitle
  }, () => {
    console.log(this.state.searchURL)
    fetch(this.state.searchURL)
      .then(response => {
        return response.json()
      }).then(json => console.log(json),
        err => console.log(err))
  })
}
```

Expected Appearance:
![success OMDB Eraserhead console response](https://i.imgur.com/ADtTqUz.png)

## Rendering our response in the Browser


Let's make a movie component that will render a view of our movie

```js
class MovieInfo extends React.Component {
  render () {
    return  (
      <div>
        <h1>Title: {this.props.movie.Title}</h1>
        <h2>Year: {this.props.movie.Year}</h2>
        <img src={this.props.movie.Poster} alt={this.props.movie.Title}/>
        <h3>Genre: {this.props.movie.Genre}</h3>
        <h4>Plot: {this.props.movie.Plot}</h4>
      </div>
    )
  }
}
```

**Gotcha!** - the data that came back to us from OMDB set the keys to have a capital letter- we need to make sure we match our data


Now, instead of console.logging our data, let's set the state and clear the form

```js
handleSubmit (event) {
  event.preventDefault()
  this.setState({
    searchURL: this.state.baseURL + this.state.apikey + this.state.query + this.state.movieTitle
  }, () => {
    fetch(this.state.searchURL)
      .then(response => {
        return response.json()
      }).then(json => this.setState({
        movie: json,
        movieTitle: ''
      }),
      err => console.log(err))
  })
}
```

Finally, let's swap out our `a` tag with our component and pass in our movie data as props we'll render nothing if there is no data.

```js
render () {
  return (
    <React.Fragment>
      <form onSubmit={this.handleSubmit}>
        <label htmlFor='movieTitle'>Title</label>
        <input
          id='movieTitle'
          type='text'
          placeholder='Movie Title'
          onChange={this.handleChange}
        />
        <input
          type='submit'
          value='Find Movie Info'
        />
      </form>
      {(this.state.movie)
        ? <MovieInfo movie={this.state.movie} />
        : ''
      }
    </React.Fragment>
  )
}
```


**Entire code**

```js
class MovieInfo extends React.Component {
  render () {
    return  (
      <div>
        <h1>Title: {this.props.movie.Title}</h1>
        <h2>Year: {this.props.movie.Year}</h2>
        <img src={this.props.movie.Poster} alt={this.props.movie.Title}/>
        <h3>Genre: {this.props.movie.Genre}</h3>
        <h4>Plot: {this.props.movie.Plot}</h4>
      </div>
    )
  }
}

class OMDBQueryForm extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      baseURL: 'http://www.omdbapi.com/?',
      apikey: 'apikey=' + '98e3fb1f',
      query: '&t=',
      movieTitle: '',
      searchURL: '',
      movie: ''
    }
    this.handleSubmit = this.handleSubmit.bind(this)
    this.handleChange = this.handleChange.bind(this)
  }
  handleChange (event) {
    this.setState({ [event.target.id]: event.target.value })
  }
  handleSubmit (event) {
    event.preventDefault()
    this.setState({
      searchURL: this.state.baseURL + this.state.apikey + this.state.query + this.state.movieTitle
    }, () => {
      fetch(this.state.searchURL)
        .then(response => {
          return response.json()
        }).then(json => this.setState({
          movie: json,
          movieTitle: ''
        }),
        err => console.log(err))
    })
  }
  render () {
    return (
      <React.Fragment>
        <form onSubmit={this.handleSubmit}>
          <label htmlFor='movieTitle'>Title</label>
          <input
            id='movieTitle'
            type='text'
            value={this.state.movieTitle}
            onChange={this.handleChange}
          />
          <input
            type='submit'
            value='Find Movie Info'
          />
        </form>
        {(this.state.movie)
          ? <MovieInfo movie={this.state.movie}/>
          : ''
        }
      </React.Fragment>
    )
  }
}

class App extends React.Component {
  render () {
    return (
      <OMDBQueryForm />
    )
  }
}

ReactDOM.render(
  <App />,
  document.querySelector('.container')
)

```
