# React Container Components

In this lesson, we will learn about React "container components." By the end of
this lesson you will be able to:

1. Explain the difference between container components and presentational components
2. Explain how to create a container component.
3. Explain the benefits of using a container component.

## What Are Container Components

We know now that much of our app ought to consist of presentational components,
which deal exclusively with the way the UI looks and which preferably carry no
state. By the same token, our applications will almost always need some
components that take responsibility for the _the way things work._

![Dolphins](https://media.giphy.com/media/9HXOac9CZshOM/giphy.gif)

So what is a container component? The main thing to keep in mind here is that
container components and presentational components go together. In fact, you can
think of them as part of the same design pattern. Where presentational
components don't manage state, container components do. Where presentation
components are usually subordinate "children" in a component hierarchy,
container components are in almost every case the "parents" of presentational
components.

Here's a concise definition of the container component pattern:

- Container components are primarily concerned with _how things work_

- They rarely have any HTML markup of their own, aside from a wrapping `<div>`;

- They are often stateful

- They are responsible for providing data and behavior to their children (usually
  presentational components).

## An Example

To make things a bit clearer, let's look at an example. Let's take a very basic
use case of a `BookList` widget that we need to write for an application that
allows book lovers to communicate about their favorite reads. The component
specification is simple enough: the component should just render the current
user's book list. Here's one way this component could be written:

```jsx
class BookList extends Component {
  constructor(props) {
    super(props);

    this.state = {
      books: [],
    };
  }

  componentDidMount() {
    fetch(
      "https://learn-co-curriculum.github.io/books-json-example-api/books.json"
    )
      .then((response) => response.json())
      .then((bookData) => this.setState({ books: bookData.books }));
  }

  renderBooks = () => {
    return this.state.books.map((book) => {
      return (
        <div className="book">
          <img src={book.img_url} />
          <h3>{book.title}</h3>
        </div>
      );
    });
  };

  render() {
    return <div className="book-list">{this.renderBooks()}</div>;
  }
}
```

In some respects, this is a perfectly acceptable component. It meets our basic
specification. However, there are some disadvantages to this implementation.
Consider for a moment the number of assumptions this component makes about how
and what data is fetched and how that data displayed:

- It assumes that the call to the api returns a JSON object containing a list of
  book objects with the properties `img_url` and `title`.

- It assumes that the book list will always be rendered with the same markup
  returned by the render function.

In short, this component has tightly coupled together a whole set of assumptions
about the data layer with another set of assumptions about the presentation
layer. This is less than ideal.

Why? Well, what if the data returned by our API call changed at some point in
the future? Suddenly we'd have errors. Or, what if we found that we needed book
lists elsewhere in our app? Could we reuse this code? No! Because the whole
thing has too many assumptions built-in; it's just too opinionated about how it
_should_ be used. What a shame!

![Mr. Bean is outraged](http://gifrific.com/wp-content/uploads/2014/02/Angry-Mr-Bean-Shakes-Head.gif)

## Separating Concerns Using a Container Component

Luckily, with React, we have an alternative. The key lies in our ability to
_compose_ our UI out of multiple React components. Using this feature, we can
break the above single component into two components. First, we'd isolate the
UI layer into a presentational component; then we'd wrap that presentational
component in a container component that handles the state and other business
logic.

Here's how this might look. Take a gander at the following code snippets. First,
we have the presentational component, `Book`:

```jsx
// src/Book.js
import React from "react";

const Book = ({ title, img_url }) => (
  <div className="book">
    <img src={img_url} alt={title} />
    <h3>{title}</h3>
  </div>
);

export default Book;
```

The component above has one thing to do - given props of `title` and `img_url`, define how
these props should be displayed. This component is so simple, it's easy to write it compactly
as a functional component.

Since we're dealing with an app that displays book*s*, one level up from
`Books`, we can write a second component, `BookList`. This component takes in a
`books` prop and maps over it, rendering a `Book` component for each element in
the `books` prop.

```jsx
// src/BookList.js
import React from "react";
import Book from "./Book";

const BookList = ({ books }) => (
  <div className="book-list">
    {books.map((book) => (
      <Book title={book.title} img_url={book.img_url} />
    ))}
  </div>
);

export default BookList;
```

Here again, although there is a bit more going on in the `BookList` component,
there isn't really anything complicated going on - this component could also be
considered a presentational component, as all it really does is to define how to
display the `Book` components and to pass data to those components.

That leaves the final component, where the main logic is:

```jsx
// src/BookListContainer.js
import React from "react";
import BookList from "./BookList";

class BookListContainer extends React.Component {
  constructor() {
    super();

    this.state = {
      books: [],
    };
  }

  componentDidMount() {
    fetch(
      "https://learn-co-curriculum.github.io/books-json-example-api/books.json"
    )
      .then((response) => response.json())
      .then((bookData) => this.setState({ books: bookData.books }));
  }

  render() {
    return <BookList books={this.state.books} />;
  }
}

export default BookListContainer;
```

This component maintains the state and handles a request for data from a remote
API. In terms of rendering, all it does it render `BookList` and pass a piece of
state down.

So what have we done? We've broken up the original component into a few pieces.
All the state is contained in our container component `BookListContainer`. The
logic is the same, but it is has been _uncoupled_ from the presentation layer,
which is now contained in the `BookList` and `Book` components. Those
components, which are stateless, are now incredibly stable as well as concise.

Because we've uncoupled (or to be more precise, loosely coupled) the data and
presentation layers, we can now easily reuse the `BookList` component. We could,
for example, write another container component &mdash; let's say
`FavoritedBookListContainer` &mdash; and then import and wrap `BookList` in
order to build a different piece of our UI with the same code.

If, later, our design team decided to redesign the way book lists appear in our
app, they would only need to update `BookList`. This is React at its best.
Saving time and minimizing codebase.

The container component pattern in React therefore helps us write better code by
enhancing the separation of concerns, i.e. the decoupling of the data and
presentation layers, and by enabling code-reuse.

## Addendum on Container Components

React is an ever evolving framework. Using container and presentational components
is just one way in which we can write and structure React applications. New additions
to the language, such as [React hooks], solve some of the challenges that were helped
by creating separate components to handle logic and presentation. The originator
of the container design pattern, Dan Abramov (part of the React dev team), has added a note to his
original post on container components regarding this.

While container components have been de-emphasized, they are still a useful tool
while you get your feet wet with React. Structuring apps with container and
presentational components helps us to understand the way data is shared in an
application. It allows us to think in a more object-oriented way about React
components and can keep our applications organized.

It is now entirely possible to write React applications in a single functional
component, and we encourage you to explore the newest features of React after
completing these lessons.

[react hooks]: https://reactjs.org/docs/hooks-intro.html

## Resources

- [Container Components](https://medium.com/@learnreact/container-components-c0e67432e005#.2kd1wuyp4)
- [CSS Tricks: Container Components](https://css-tricks.com/learning-react-container-components/)
