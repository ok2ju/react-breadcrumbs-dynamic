# react-breadcrumbs-dynamic

[![Npm package](https://img.shields.io/npm/v/react-breadcrumbs-dynamic.svg?style=flat)](https://npmjs.com/package/react-breadcrumbs-dynamic)
[![Npm downloads](https://img.shields.io/npm/dm/react-breadcrumbs-dynamic.svg?style=flat)](https://npmjs.com/package/react-breadcrumbs-dynamic)
[![Travis build status](http://img.shields.io/travis/oklas/react-breadcrumbs-dynamic.svg?style=flat)](https://travis-ci.org/oklas/react-breadcrumbs-dynamic)
[![Test Coverage](https://img.shields.io/codecov/c/github/oklas/react-breadcrumbs-dynamic.svg)](https://codecov.io/gh/oklas/react-breadcrumbs-dynamic)
[![Code Climate](https://codeclimate.com/github/oklas/react-breadcrumbs-dynamic/badges/gpa.svg)](https://codeclimate.com/github/oklas/react-breadcrumbs-dynamic)
[![Dependency Status](https://david-dm.org/oklas/react-breadcrumbs-dynamic.svg)](https://david-dm.org/oklas/react-breadcrumbs-dynamic)

***

[`🏠`](https://oklas.github.io/react-breadcrumbs-dynamic) > [`React dynamic breadcrumbs`](https://oklas.github.io/react-breadcrumbs-dynamic) > [`extremely flexible`](https://oklas.github.io/react-breadcrumbs-dynamic) > [`and`](https://oklas.github.io/react-breadcrumbs-dynamic) > [`easy to use`](https://oklas.github.io/react-breadcrumbs-dynamic)


This is completely router-independent react breadcrumbs solution which means
that you can use it with any version of React Router (2 or 3 or 4) or any other
routing library for React or without routing at all. All what you need is just
to specify components for breadcrumbs items and its props. However props and
components should be specified separately. Props should be specified in
intermediator component `BreadcrumbsItem` anywhere in your hierarchy of
components and routes. Breadcrumbs will be built and (currently) sorted by the
length of the URL. An application may contain several breadcrumbs with different
components and design.

Visit live **[DEMO](//oklas.github.io/react-breadcrumbs-dynamic)** (source code of demo in [example](example) folder)


# Installation

``` sh
npm install --save react-breadcrumbs-dynamic
```

# Base configuration

Add a `<BreadcrumbsProvider/>` component to the root of your React component
tree like you do it for `react-redux` or `react-router`.
In the react tree the `BreadcrumbsProvider` component must be parent of all
components of this library with any deeps of nesting. See also
*Advanced Usage and Performance* section.

``` javascript
import {BreadcrumbsProvider} from 'react-breadcrumbs-dynamic'

const theApp = (
  <BreadcrumbsProvider>
    <App />
  </BreadcrumbsProvider>
)

ReactDOM.render(theApp, document.getElementById('root'))
```


# Instance configuration

The breadcrumbs instance is implemented in the `Breadcrumbs` component. The
`Breadcrumbs` component needs to be configured, however all params have default
value. In this example the `react-router` v4 routing specification is used.
Please note that `item` and `finalItem` require react component (class) instead
of react element. However `separator` requires react element.

``` javascript
import {Breadcrumbs} from 'react-breadcrumbs-dynamic'

const Page = (props) => (
  return (
    <div className="App">
      <Header>
        <Breadcrumbs
          separator={<b> / </b>}
          item={NavLink}
          finalItem={'b'}
          finalProps={{
            style: {color: 'red'}
          }}
        />
      </Header>
      {props.children}
      <Footer>
        <Breadcrumbs/>
      </Footer>
    </div>
  )
}
```


# Add item to breadcrumbs

Each routed component in your react tree is generally associated with route
and with correspondent breadcrumbs. Each component may add its breadcrumbs
item. The `BreadcrumbsItem` component mandatory requires the `to` prop which
contains bearing key with URL for breadcrumbs working. So if you use simple
`<a>` tag for breadcrumb url - you need to use the `duplicateProps` and/or
`renameProps`, or need to specify both `to` and `href`.
See also *Advanced Usage and Performance* section.

Simple configure of the breadcrumbs items:

``` javascript

import {BreadcrumbsItem} from 'react-breadcrumbs-dynamic'

const App = (props) => (
  <div>
    <BreadcrumbsItem to='/'>Main Page</BreadcrumbsItem>
    {props.children}
    <Route exact path="/user" component={User} />
  </div>
)
  

const User = (props) => (
  <div>
    <BreadcrumbsItem to='/user'>Home</BreadcrumbsItem>
    <h2>Home</h2>
    {props.children}
    <Route exact path="/user/contacts" component={Contacts} />
  </div>
)

const Contacts = (props) => (
  <div>
    <BreadcrumbsItem to='/user/contacts'>Contacts</BreadcrumbsItem>
    <h2>Contacts</h2>
    ...
  </div>
)
```


# Result

The result of above code will represent breadcrumbs like this:

``` javascript
  <NavLink to='/'>Main Page</NavLink>
  <b> / </b>
  <NavLink to='/user'>Home</NavLink>
  <b> / </b>
  <b to='/user/contacts'>Contacts</b>
```

___

# Advanced Usage and Performance

This library makes everything possible to make the things simple and effective. 
However, the library cannot know about your application state nor
can undertake deep inspections of your data to detect changes in efficiency
considerations.


## Avoid to pass arrays/objects/jsx in props of `BreadcrumbsItem`

When jsx code like this: `<tag>...</tag>` or `<Component t='...'/>`
is evaluated  the new react element instance is created even if the same props
was passed. Same thing is when code contains `{}` or `[]` - the new object or
array is created in-place. All this cases represents new instance which is
considered as value change event even if this arrays/objects/jsx have same
values in depth.

The Library does not analyze data into depth to detect changes. So when you
pass new react element or newly created array or object to `BreadcrumbsItem`
as props it receives same props with new value. It gives proper result but
applying params will be defered to additional tree rerendering step and
executed after all again.

Full complete list of types allowed as `BreadcrumbsItem` props for better
performance:

* string
* number
* undefined
* boolean
* symbol

So to avoid additional processing you should not specify arrays or objects or react
elements (ie jsx) as props of `BreadcrumbsItem` in `render()` method:

``` javascript
render() {
  return <BreadcrumbsItem to='/' title={<b>Main Page</b>} x={[1,2,3]}/> // bad
}

render() {
  return <BreadcrumbsItem to='/'><b>Main Page</b></BreadcrumbsItem> // bad
}

render() {
  return <BreadcrumbsItem to='/'>Main Page</BreadcrumbsItem> // good
}
```


## Custom update filter

Another way is to use breadcrumbs item update filter. This is achieved by
specifying the function in `shouldBreadcrumbsUpdate` param of
`BreadcrumbsProvider` like this:

``` javascript
const theApp = (
  <BreadcrumbsProvider
    shouldBreadcrumbsUpdate = {
      (prevProps, props) => {
        // custom breadcrumbs item update filter condition
        return prevProps.to != props.to
      }
    }
  />
    <App />
  </BreadcrumbsProvider>
)
```

The item update filter must guarantee that the difference in the params
are really due to different source data. For example condition
`{k:'v'} != {k:'v'}` shows that this is the different data, although in
fact source data is the same and has not been changed.

**Warning:** if update filter determines the data changing at the time when
the original data has been unchanged this forms an infinite loop.

On the other hand, if some data is not recognized as changed, the breadcrumbs
will not be updated and will not look like expected.


## Alternate usage

An alternative, more flexible and at the same time more complicated way is to
update breadcrumbs only when related to breadcrubms item data is changed.
The library provides interface for that. The higher order
component creation function `withBreadcrumbsItem` will integrate `breadcrumbs`
object with `item()` and `items()` functions into props of your `Component`.

**Warning**: Never call `breadcrumbs.item()` or `breadcrumbs.items()` from
`render()` or `componentWillUpdate()` methods or from `constructor()` of your
component. This means breadcrumbs item must be not depend from state of
your current "with breadcrubms" component.

This way allows to specify arrays and objects and react elements (i.e. jsx) in
props but functions `breadcrumbs.item()` or `breadcrumbs.items()` must be
called from the `if` statement, where is the update condition which 
checks for changes the application related to breadcrubms item data.

**Warning:** if update condition determines the data changing at the time when
the original data has been unchanged this forms an infinite loop.


``` javascript
import { withBreadcrumbsItem, Dummy as Item } from 'react-breadcrumbs-dynamic'

@withBreadcrumbsItem
class CustomComponent extends React.Component {
  static propTypes = {
    breadcrumbs: PropTypes.object,
  }

  componentWillMount() {
    this.configureBreadcrumbs(this.props)
  }

  componentWillReceiveProps(nextProps) {
    // mandatory required update filter condition
    if( this.props.slug !== nextProps.slug ) {
      this.configureBreadcrumbs(nextProps)
    }
  }

  configureBreadcrumbs = (props) => {
    props.breadcrumbs.items(
      <div>
        <Item to='/' some={[1,2,3]}><b>Home</b></Item>
        <Item to=`/${props.slug}`><b>{props.slug}</b></Item>
      </div>
    )
  }

  render() {
    return (
      <div />
    )
  }
}
```

# The components and functions

## `Breadcrumbs` component props

* `separator` - separator between breadcrumbs items (default: undefined)
* `item` - component of breadcrumbs items (default: 'a')
* `finalItem` - component of final breadcrumbs item (default: value of `item` prop)
* `finalProps` - final item props which override specified in `BreadcrumbsItem` (default: {})
* `container` - wrapper component (default is `span`)
* `containerProps` - props for `container` components if defined (default: {})
* `renameProps` - rename props passed from item intermedator to item
* `duplicateProps` - duplicate same as `renameProps` but without remove original.


## `BreadcrumbsItem` component props

The `BreadcrumbsItem` component may have any prop and may have children. Each prop
for `BreadcrumbsItem` will be passed to correspondent breadcrumb component specified
in `item` or `finalItem` prop of `Breadcrumbs`. Only one prop is mandatory.

* `to` - mandatory required bearing key with URL for breadcrumbs working
* `...` - any more number of properties.


## `BreadcrumbsProvider` component props

The `BreadcrumbsProvider` props:

* `shouldBreadcrumbsUpdate` - custom breadcrumbs item update filter which is
the function with two args, where first arg is `prevProps` and second arg is
`props` - previous and current breadcrumbs item props respectively.


## `withBreadcrumbsItem()` function

This function creates higher order component. It acquire one argument with your
custom react component and return appropriate component which will have
`breadcrumbs` in its props with methods `item()` and `items()`.


## `this.props.breadcrumbs.item()` and `this.props.breadcrumbs.items()`

Methods to configure breadcrumbs item of your current react component.
These methods will be added to props by HOC of `withBreadcrumbsItem` function.
The function `item()` accepts one react component with props and the functions
`items()` accepts react component with children which may contain any number of
components to create correspondent number of breadcrumbs item. The breadcrumbs
item for these functions may be any react component and only props is
significant. The `Dummy` and the `Item` components is exported by library
for this case. Each function accepts null to reset breadcrumbs items to none if
current component is no more needed to represent any breadcrumbs items.


## LICENSE

#### [MIT](./LICENSE.md)
