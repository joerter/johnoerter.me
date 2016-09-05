---
title: Build a Universal JavaScript app
date: 8:14 06/18/2016
taxonomy:
    category: blog
    tag: [javascript]
---

In this article, we will walk through how to write a universal (or isomorphic) JavaScript app. But first, let's talk through what a universal JavaScript application is, and why it is such an exciting concept.

## What is a Universal JavaScript app?

To put it simply, a universal JavaScript app is an application that can render itself on the client and the server. It combines the features of traditional server side MVC frameworks (Rails, ASP.NET MVC, Spring MVC) where markup is generated on the server and sent to the client with the features of SPA frameworks (Angular, Ember, Backbone, etc.) where the server is only responsible for the data and the client generates markup.

## Universal or Isomorphic?

There has been some debate in the JavaScript community over the terms "universal" and "isomorphic" to describe apps that can run on the client and server. I personally prefer the term "universal" simply because it's a more familiar word and makes the concept easier to understand. If you're interested in this discussion, you can read the below articles:

[Isomorphic JavaScript: The Future of Web Apps](http://nerds.airbnb.com/isomorphic-javascript-future-web-apps/) by Spike Brehm popularizes the term "isomorphic".

[Universal JavaScript](https://medium.com/@mjackson/universal-javascript-4761051b7ae9#.o6gplaif7) by Michael Jackson puts forth the term "universal" as a better alternative.

[Is "Isomorphic JavaScript" a good term?](http://www.2ality.com/2015/08/isomorphic-javascript.html) by Dr. Axel Rauschmayer says that maybe certain applications should be called isomorphic and others should be called universal.

## What are the advantages?

### Context Switching

Switching between one language on the server and JavaScript on the client can harm your productivity. JavaScript is a unique language that, for better or worse, behaves in a very different way from most server side languages. Writing universal JavaScript apps allows you to simplify your workflow, and immerse yourself in JavaScript. If you're writing a web application today, chances are you're writing a lot of JavaScript anyways. Why not dive in? Node continues to improve with better performance and more features thanks to V8 and it's well run community, and npm is a fantastic package manager with thousands of quality packages available. There is tremendous brain power being devoted to JavaScript right now. Take advantage!

On top of that, maintainability of a universal app is better because it allows more code reuse. How many times have you implemented the same validation logic in your server and front end code?  Or rewritten utility functions? With some careful architecture and decoupling, you can write and test code once that will work on the server and client.

### Performance

SPAs are great because they allow the user to navigate applications without waiting for full pages to be sent down from the server. The cost, however, is longer wait times for the application to be initialized on the first load because the browser needs to receive all the assets needed to run the full app up front. What if there are rarely visited areas in your app? Why should every client have to wait for the logic and assets needed for those areas? This was the problem [Netflix solved using universal JavaScript](http://techblog.netflix.com/2015/08/making-netflixcom-faster.html).

MVC apps have the inverse problem. Each page only has the markup, assets, and JavaScript needed for that page, but the trade off is round trips to the server for every page.

###  SEO

Another disadvantage of SPAs is their weakness on SEO. Although [web crawlers are getting better at understanding JavaScript](https://webmasters.googleblog.com/2014/05/understanding-web-pages-better.html), a site generated on the server will always be superior. With universal JavaScript, any public facing page on your site can be easily requested and indexed by search engines.

## Building an Example Universal JavaScript App

Now that we've gained some background on universal JavaScript apps, let's walk through building a very simple blog website as an example. Here are the tools we'll use:

[Express](http://expressjs.com/)  
[React](https://facebook.github.io/react/)  
[React Router](https://github.com/reactjs/react-router)  
[Babel](https://babeljs.io/)  
[Webpack](https://webpack.github.io/)  

I've chosen these tools because of their popularity, and ease of accomplishing our task. I won't be covering how to use Redux or other Flux implementations because, while useful in a production application, are not necessary for demoing how to create a universal app.

To keep things simple, we will forgo a database and just store our data in a flat file. We'll also keep the Webpack shenanigans to a minimum and only do what is necessary to transpile and bundle our code.

You can grab the code for this walkthrough at [https://github.com/joerter/universal-blog](https://github.com/joerter/universal-blog), and follow along. There are branches for each step along the way. Be sure to run `npm install` for each step.

Let's get started!

### Step 1: Serving Post Data
`git checkout serving-post-data && npm install`

We're going to start off slow, and simply set up the data we want to serve. Our posts are stored in the `posts.js` file, and we just have a simple Express server in `server.js` that takes requests at `/api/post/{id}`. Snippets of these files are below.

    // posts.js
    module.exports = [
      ...
      {
        id: 2,
        title: 'Expert Node',
        slug: 'expert-node',
        content: 'Street art 8-bit photo booth, aesthetic kickstarter organic raw denim hoodie non kale chips pour-over occaecat. Banjo non ea, enim assumenda forage excepteur typewriter dolore ullamco. Pickled meggings dreamcatcher ugh, church-key brooklyn portland freegan normcore meditation tacos aute chicharrones skateboard polaroid. Delectus affogato assumenda heirloom sed, do squid aute voluptate sartorial. Roof party drinking vinegar franzen mixtape meditation asymmetrical. Yuccie flexitarian est accusamus, yr 3 wolf moon aliqua mumblecore waistcoat freegan shabby chic. Irure 90\'s commodo, letterpress nostrud echo park cray assumenda stumptown lumbersexual magna microdosing slow-carb dreamcatcher bicycle rights. Scenester sartorial duis, pop-up etsy sed man bun art party bicycle rights delectus fixie enim. Master cleanse esse exercitation, twee pariatur venmo eu sed ethical. Plaid freegan chambray, man braid aesthetic swag exercitation godard schlitz. Esse placeat VHS knausgaard fashion axe cred. In cray selvage, waistcoat 8-bit excepteur duis schlitz. Before they sold out bicycle rights fixie excepteur, drinking vinegar normcore laboris 90\'s cliche aliqua 8-bit hoodie post-ironic. Seitan tattooed thundercats, kinfolk consectetur etsy veniam tofu enim pour-over narwhal hammock plaid.'
      },
    ...
    ]

    // server.js
    ...

    app.get('/api/post/:id?', (req, res) => {
      const id = req.params.id
      if (!id) {
        res.send(posts)
      } else {
        const post = posts.find(p => p.id == id);
        if (post)
          res.send(post)
        else
          res.status(404).send('Not Found')
      }
    })

    ...

You can start the server by running `node server.js`, and then request all posts by going to `localhost:3000/api/post`or a single post by id such as `localhost:3000/api/post/0`. Great! Let's move on.

### Step 2: Add React
`git checkout add-react && npm install`

Now that we have the data exposed via a simple web service, let's use React to render a list of posts on the page. Before we get there, however, we need to setup webpack to transpile and bundle our code. Below is our simple `webpack.config.js` to do this:

    // webpack.config.js
    var webpack = require('webpack')

    module.exports = {
      entry: './index.js',

      output: {
        path: 'public',
        filename: 'bundle.js'
      },

      module: {
        loaders: [
          { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader?presets[]=es2015&presets[]=react' }
        ]
      }
    }

All we're doing is bundling our code with `index.js` as an entry point, and writing the bundle to a public folder that will be served by Express. Speaking of `index.js` here it is:

    // index.js
    import React from 'react'
    import { render } from 'react-dom'
    import App from './components/app'

    render (
      <App />, document.getElementById('app')
    )

And finally, we have `App.js`

    // components/App.js
    import React from 'react'

    const allPostsUrl = '/api/post'

    class App extends React.Component {
      constructor(props) {
        super(props)
        this.state = {
          posts: []
        }
      }

      componentDidMount() {
        const request = new XMLHttpRequest()
        request.open('GET', allPostsUrl, true)
        request.setRequestHeader('Content-type', 'application/json');

        request.onload = () => {
          if (request.status === 200) {
            this.setState({
              posts: JSON.parse(request.response)
            });
          }
        }

        request.send();
      }

      render() {
        const posts = this.state.posts.map((post) => {
          return <li key={post.id}>{post.title}</li>
        })

        return (
          <div>
            <h3>Posts</h3>
            <ul>
              {posts}
            </ul>
          </div>
        )
      }
    }

    export default App

Once the App component is mounted, it sends a request for the posts, and renders them as a list. To see this step in action, build the webpack bundle first with `npm run build:client`. Then, you can run `node server.js` just like before. `http://localhost:3000` will now display a list of our posts.

### Step 3: Client side routing with React Router
`git checkout client-side-routing && npm install`

Now that we're pulling and displaying posts, let's add some navigation to individual pages for each post. To do this, we will turn our list of posts from step 2 into links that are always present on the page. Each post will live at `http://localhost:3000/:postId/:postSlug`. We can use React Router and a `routes.js` file to setup this structure:

    // components/routes.js
    import React from 'react'
    import { Route } from 'react-router'
    import App from './App'
    import Post from './Post'

    module.exports = (
      <Route path="/" component={App}>
        <Route path="/:postId/:postName" component={Post} />
      </Route>
    )

We've changed the render method in App.js to render links to posts instead of just `<li>` tags:

    // components/App.js
    import React from 'react'
    import { Link } from 'react-router'

    const allPostsUrl = '/api/post'

    class App extends React.Component {
      constructor(props) {
        super(props)
        this.state = {
          posts: []
        }
      }

      ...

      render() {
        const posts = this.state.posts.map((post) => {
          const linkTo = `/${post.id}/${post.slug}`;

          return (
            <li key={post.id}>
              <Link to={linkTo}>{post.title}</Link>
            </li>
          )
        })

        return (
          <div>
            <h3>Posts</h3>
            <ul>
              {posts}
            </ul>

            {this.props.children}
          </div>
        )
      }
    }

    export default App

And, we'll add a Post.js component to render each post's content:

    // components/Post.js
    import React from 'react'

    class Post extends React.Component {
      constructor(props) {
        super(props)

        this.state = {
          title: '',
          content: ''
        }
      }

      fetchPost(id) {
        const request = new XMLHttpRequest()
        request.open('GET', '/api/post/' + id, true)
        request.setRequestHeader('Content-type', 'application/json');

        request.onload = () => {
          if (request.status === 200) {
            const response = JSON.parse(request.response)
            this.setState({
              title: response.title,
              content: response.content
            });
          }
        }

        request.send();

      }

      componentDidMount() {
        this.fetchPost(this.props.params.postId)
      }

      componentWillReceiveProps(nextProps) {
        this.fetchPost(nextProps.params.postId)
      }

      render() {
        return (
          <div>
            <h3>{this.state.title}</h3>
            <p>{this.state.content}</p>
          </div>
        )
      }
    }

    export default Post

The `componentDidMount()` and `componentWillReceiveProps()` methods are important because they let us know when we should fetch a post from the server. `componentDidMount()` will handle the first time the Post.js component is rendered, and then `componentWillReceiveProps()` will take over as React Router handles rerendering the component with different props.

Run `npm build:client && node server.js` again to build and run the app. You will now be able to go to `http://localhost:3000` and navigate around to the different posts. However, if you try to refresh on a single post page, you will get something like `Cannot GET /3/debugging-node-apps`. That's because our Express server doesn't know how to handle that kind of route. React Router is handling it completely on the front end. Onward to server rendering!

### Step 4: Server rendering
`git checkout server-rendering && npm install`

Okay, now we're finally getting to the good stuff. In this step, we'll use React Router to help our server take application requests and render the appropriate markup. To do that, we need to also build a server bundle like we build a client bundle, so that the server can understand JSX. Therefore, we've added the below webpack.server.config.js:

    // webpack.server.config.js
    var fs = require('fs')
    var path = require('path')

    module.exports = {

      entry: path.resolve(__dirname, 'server.js'),

      output: {
        filename: 'server.bundle.js'
      },

      target: 'node',

      // keep node_module paths out of the bundle
      externals: fs.readdirSync(path.resolve(__dirname, 'node_modules')).concat([
        'react-dom/server', 'react/addons',
      ]).reduce(function (ext, mod) {
        ext[mod] = 'commonjs ' + mod
        return ext
      }, {}),

      node: {
        __filename: true,
        __dirname: true
      },

      module: {
        loaders: [
          { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader?presets[]=es2015&presets[]=react' }
        ]
      }
    }

We've also added the following code to server.js:

    // server.js    
    import React from 'react'
    import { renderToString } from 'react-dom/server'
    import { match, RouterContext } from 'react-router'
    import routes from './components/routes'

    const app = express()

    ...

    app.get('*', (req, res) => {
      match({ routes: routes, location: req.url }, (err, redirect, props) => {
        if (err) {
          res.status(500).send(err.message)
        } else if (redirect) {
          res.redirect(redirect.pathname + redirect.search)
        } else if (props) {
          const appHtml = renderToString(<RouterContext {...props} />)
          res.send(renderPage(appHtml))
        } else {
          res.status(404).send('Not Found')
        }
      })
    })

    function renderPage(appHtml) {
      return `
      <!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="UTF-8">
        <title>Universal Blog</title>
      </head>
      <body>
        <div id="app">${appHtml}</div>
        <script src="/bundle.js"></script>
      </body>
      </html>
      `
    }

    ...

Using React Router's `match` function, the server can find the appropriate requested route, `renderToString`, and send the markup down the wire. Run `npm start` to build the client and server bundles and start the app. Fantastic right? We're not done yet, though. Even though the markup is being generated on the server, we're still fetching all the data client side. Go ahead and click through the posts with your dev tools open, and you'll see the requests. It would be far better to load the data while we're rendering the markup instead of having to request it separately on the client.

Since server rendering and universal apps are still bleeding edge, there aren't really any established best practices for data loading. If you're using some kind of Flux implementation, there may be some specific guidance. But for this use case, we will simply grab all of the posts and feed them through our app. In order to this, we first need to do some refactoring on our current architecture.

### Step 5: Data Flow Refactor
`git checkout data-flow-refactor && npm install`

It's a little weird how each post page has to make a request to the server for its content, even though the App component already has all the posts in its state. A better solution would be to have App simply pass the appropriate content down to the Post component.

    // components/routes.js
    import React from 'react'
    import { Route } from 'react-router'
    import App from './App'
    import Post from './Post'

    module.exports = (
      <Route path="/" component={App}>
        <Route path="/:postId/:postName" />
      </Route>
    )

In our routes.js, we've made the Post route a componentless route. It's still a child of the App route, but now has to completely rely on the App component for rendering. Below are the changes to App.js:

    // components/App.js
    ...

      render() {
        const posts = this.state.posts.map((post) => {
          const linkTo = `/${post.id}/${post.slug}`;

          return (
            <li key={post.id}>
              <Link to={linkTo}>{post.title}</Link>
            </li>
          )
        })

        const { postId, postName } = this.props.params;
        let postTitle, postContent
        if (postId && postName) {
          const post = this.state.posts.find(p => p.id == postId)
          postTitle = post.title
          postContent = post.content
        }

        return (
          <div>
            <h3>Posts</h3>
            <ul>
              {posts}
            </ul>

            {postTitle && postContent ? (
              <Post title={postTitle} content={postContent} />
            ) : (
              <h1>Welcome to the Universal Blog!</h1>
            )}
          </div>
        )
      }
    }

    export default App

If we are on a post page, then `props.params.postId` and `props.params.postName` will both be defined and we can use them to grab the desired post and pass the data on to the Post component to be rendered. If those properties are not defined, then we're on the homepage and can simply render a greeting. Now, our Post.js component can be a simple stateless functional component that simply renders it's properties.

    // components/Post.js
    import React from 'react'

    const Post = ({title, content}) => ( <div> <h3>{title}</h3> <p>{content}</p> </div>)

    export default Post

With that refactoring complete, we're ready to implement data loading.

### Step 6: Data Loading
`git checkout data-loading && npm install`

For this final step, we just need to make two small changes in server.js and App.js:

    // server.js

    ...
    app.get('*', (req, res) => {
      match({ routes: routes, location: req.url }, (err, redirect, props) => {
        if (err) {
          res.status(500).send(err.message)
        } else if (redirect) {
          res.redirect(redirect.pathname + redirect.search)
        } else if (props) {
          const routerContextWithData = (
            <RouterContext
              {...props}
              createElement={(Component, props) => {
                return <Component posts={posts} {...props} />
              }}
            />
          )
          const appHtml = renderToString(routerContextWithData)
          res.send(renderPage(appHtml))
        } else {
          res.status(404).send('Not Found')
        }
      })
    })

    ...

    // components/App.js
    import React from 'react'
    import Post from './Post'
    import { Link, IndexLink } from 'react-router'

    const allPostsUrl = '/api/post'

    class App extends React.Component {
      constructor(props) {
        super(props)
        this.state = {
          posts: props.posts || []
        }
      }

    ...

In server.js, we're changing how the `RouterContext` creates elements by overwriting its `createElement` function and passing in our data as additional props. These props will get passed to any component that is matched by the route, which in this case will be our App component. Then, when the App component is initialized it sets its posts state property to what it got from props or an empty array.

That's it! Run `npm start` one last time, and cruise through your app. You can even disable JavaScript, and the app will automatically degrade to requesting whole pages.

Thanks for reading!
