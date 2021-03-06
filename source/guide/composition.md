title: Composing ViewModels
type: guide
order: 10
---

## Registering a Component

Vue.js allows you to treat registered ViewModel constructors as reusable components. To register a component, use the global `Vue.component()` method:

``` js
var MyComponent = Vue.extend({
    template: 'A custom component!'
})
Vue.component('my-component', MyComponent)
```

To make things easier, you can also directly pass in the option object instead of an actual constructor:

``` js
Vue.component('my-component', {
    template: 'A custom component!'
})
```

Then you can use it in a parent ViewModel's template:

``` html
<div v-component="my-component"></div>
```

If you prefer, all registered components can also be used in the form of a custom element tag:

``` html
<my-component></my-component>
```

## Encapsulating Private Assets

Sometimes a component needs to use assets such as directives, filters and its own child components, but might want to keep these assets encapsulated so the component itself can be reused elsewhere. You can do that using the private assets instantiation options. Private assets will only be accessible by the instances of the owner component and its child components.

``` js
// All 5 types of assets
var MyComponent = Vue.extend({
    directives: {
        // id : definition pairs same with the global methods
        'private-directive': function () {
            // ...
        }
    },
    filters: {
        // ...
    },
    components: {
        // ...
    },
    transitions: {
        // ...
    },
    partials: {
        // ...
    }
})
```

## Data Inheritance

### Inheriting Objects with `v-with`

A child component can inherit Objects in its parent's data by combining `v-component` with `v-with`:

``` html
<div id="demo-1">
    <p v-component="user-profile" v-with="user"></p>
</div>
```

``` js
// registering the component first
Vue.component('user-profile', {
    template: '{&#123;name&#125;}<br>{&#123;email&#125;}'
})
// the `user` object will be passed to the child
// component as its $data
var parent = new Vue({
    el: '#demo-1',
    data: {
        user: {
            name: 'Foo Bar',
            email: 'foo@bar.com'
        }
    }
})
```

**Result:**

<div id="demo-1" class="demo"><p v-component="user-profile" v-with="user"></p></div>
<script>
    Vue.component('user-profile', {
        template: '{&#123;name&#125;}<br>{&#123;email&#125;}'
    })
    var parent = new Vue({
        el: '#demo-1',
        data: {
            user: {
                name: 'Foo Bar',
                email: 'foo@bar.com'
            }
        }
    })
</script>

### Inheriting Array Elements with `v-repeat`

For an Array of Objects, you can combine `v-component` with `v-repeat`. In this case, for each Object in the Array, a child ViewModel will be created using that Object as data, and the specified component as the constructor.

``` html
<ul id="demo-2">
    <!-- reusing the user-profile component we registered before -->
    <li v-repeat="users" v-component="user-profile"></li>
</ul>
```

``` js
var parent2 = new Vue({
    el: '#demo-2',
    data: {
        users: [
            {
                name: 'Chuck Norris',
                email: 'chuck@norris.com'
            },
            {
                name: 'Bruce Lee',
                email: 'bruce@lee.com'
            }
        ]
    }
})
```

**Result:**

<ul id="demo-2" class="demo"><li v-repeat="users" v-component="user-profile"></li></ul>
<script>
var parent2 = new Vue({
    el: '#demo-2',
    data: {
        users: [
            {
                name: 'Chuck Norris',
                email: 'chuck@norris.com'
            },
            {
                name: 'Bruce Lee',
                email: 'bruce@lee.com'
            }
        ]
    }
})
</script>

## Accessing Child Components

Sometimes you might need to access nested child components in JavaScript. To enable that you have to assign a child component an ID using `v-component-id`. For example:

``` html
<div id="parent">
    <div v-component="user-profile" v-component-id="profile"></div>
</div>
```

``` js
var parent = new Vue({ el: '#parent' })
// access child component
var child = parent.$.profile
```

When `v-component-id` is used together with `v-repeat`, the value you get will be an Array containing the child components mirroring the data Array.

## Event Communication Between Nested Components

Although you can directly access a ViewModels children and parent, it is more convenient to use the built-in event system for cross-component communication. It also makes your code less coupled and easier to maintain. Once a parent-child relationship is established, you can dispatch and trigger events using each ViewModel's [event instance methods](/api/instance-methods.html#Cross-ViewModel_Events).

``` js
var Child = Vue.extend({
    created: function () {
        this.$dispatch('child-created', this)
    }
})

var parent = new Vue({
    template: '<div v-component="child"></div>',
    components: {
        child: Child
    },
    created: function () {
        this.$on('child-created', function (child) {
            console.log('new child created: ')
            console.log(child)
        })
    }
})
```

<script>
var Child = Vue.extend({
    created: function () {
        this.$dispatch('child-created', this)
    }
})

var parent = new Vue({
    template: '<div v-component="child"></div>',
    components: {
        child: Child
    },
    created: function () {
        this.$on('child-created', function (child) {
            console.log('new child created: ')
            console.log(child)
        })
    }
})
</script>

Next: [Building Larger Apps](/guide/application.html).
