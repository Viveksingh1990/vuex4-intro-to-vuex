Error Handling
==============

Currently, our app is missing a couple pretty important features. Specifically, whenever a user creates a new event from the **EventCreate** view, we want to take them _to_ that new event. In other words, we want to use Vue Router to navigate the user to the **EventDetails** view, which displays the details for the event they just created. So in this lesson, we’ll first accomplish that before moving on to a secondary routing behavior, where we route our user to a new **ErrorDisplay** view if an error occurs while they’re using our app.

* * *

Routing the user to EventDetails
--------------------------------

Let’s head into the **EventCreate** component to orient ourselves to how things are currently functioning.

📁 **views/EventCreate.vue**

      
    methods: {
      onSubmit() {
        const event = {
          ...this.event,
          id: uuidv4(),
          organizer: this.$store.state.user
        }
        this.$store.dispatch('createEvent', event)
      }
    }
    

As a refresher, when we click the submit button, the `onSumit` method fires, which dispatches the `createEvent` action, passing it the `event` we want to create. If that event-creation was successful, we want to route our user to the **EventDetails** for that event. That will look like this:

📁 **views/EventCreate.vue**

    methods: {
      onSubmit() {
        const event = {
          ...this.event,
          id: uuidv4(),
          organizer: this.$store.state.user
        }
        this.$store.dispatch('createEvent', event)
          .then( () => {
            this.$router.push({
              name: 'EventDetails',
              params: { id: event.id }
            })
          })
      }
    }
    

As you can see, we’re using Vue Router (`this.$router`) to `push` our user to the route by the name of `EventDetails`, and we’re using the `params` property to set the route’s `id` equal to the `event.id`.

If router params and dynamic segments are new to you, we cover this in our [Dynamic Routing](https://www.vuemastery.com/courses/real-world-vue3/dynamic-routing) lesson of Real World Vue 3. But as a refresher: when we route to **EventDetails**, we’re setting the `id` parameter of that route so that **EventDetails** has access to that `id` as a prop. Then, as soon as we arrive at **EventDetails**, it dispatches the `fetchEvent` action, using that `id` prop to get the event by its `id`, and then **EventDetails** makes use of its computed property (which is mapped to that event in the Vuex state) to print out the `event` details.

📁 **views/EventDetails.vue**

    <template>
      <div v-if="event">
        <h1>{{ event.title }}</h1>
        <p>{{ event.time }} on {{ event.date }} @ {{ event.location }}</p>
        <p>{{ event.description }}</p>
      </div>
    </template>
    
    <script>
    export default {
      props: ['id'],
      created() {
        this.$store.dispatch('fetchEvent', this.id)
      },
      computed: {
        event() {
          return this.$store.state.event
        }
      }
    }
    </script>
    

We can test this out in the browser by filling out the form and hitting submit. We should see that we’re routing to the **EventDetails** view for that new event.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1626117643421.jpg?alt=media&token=5be97e3f-2910-433d-af16-5cd1ba61be31](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1626117643421.jpg?alt=media&token=5be97e3f-2910-433d-af16-5cd1ba61be31)

Excellent. We got it to work! Now our app is behaving in a more sensical manner, routing the user to the item they’ve created versus leaving them waiting on the previous view.

* * *

Handling Errors
---------------

When we take a look through our application code, we see a number of places where we’re catching errors. Currently, they’re all within our Vuex actions:

📁 **store/index.js**

      actions: {
        createEvent({ commit }, event) {
          EventService.postEvent(event)
            .then(() => {
              commit('ADD_EVENT', event)
            })
            .catch(error => {
              console.log(error) // <--- here
            })
        },
        fetchEvents({ commit }) {
          EventService.getEvents()
            .then(response => {
              commit('SET_EVENTS', response.data)
            })
            .catch(error => {
              console.log(error) // <--- here
            })
        },
        fetchEvent({ commit, state }, id) {  
          const existingEvent = state.events.find(event => event.id === id)
          if (existingEvent) {
              commit('SET_EVENT', existingEvent)
          } else {
            EventService.getEvent(id)
              .then(response => {
                commit('SET_EVENT', response.data)
              })
              .catch(error => {
                console.log(error) // <--- and here
              })
          }
        }
      }
    

As a placeholder solution, I’ve just been console.logging these errors. But there are a couple problems with that.

First, your average user isn’t going to have their browser’s developer console open while using your app, so they’ll never see these errors. We need a way to let the user know when an error occurs.

Second, when handling errors, we don’t want to do so at a global level within Vuex. The component that ultimately triggered the error should be responsible for handling that error.

So the solution we’re going to implement involves these steps:

1.  Create a new component to display errors (**ErrorDisplay**)
2.  Add **ErrorDisplay** as a route
3.  Enable our Vuex actions to hand over the error to the component that triggered it
4.  Route the user to the **ErrorDisplay** view when an error is caught

Let’s get started…

* * *

### The ErrorDisplay Component

The component we’ll use to display our errors will be quite simple. It just needs a template that prints out the error, which the component receives as a prop.

📁 **views/ErrorDisplay.vue**

    <template>
      <h4>Oops! There was an error:</h4>
      <p>{{ error }}</p>
    </template>
    
    <script>
    export default {
      props: ['error']
    }
    </script>
    

Now that our component is ready, we can make it a view-level component and add it as a route.

* * *

### Adding ErrorDisplay as a Route

We’ll import that new **ErrorDisplay.vue** file into the router’s **index.js** file and add a route object for it.

📁 **router/index.js**

    // ...
    import ErrorDisplay from '@/views/ErrorDisplay.vue'
    
    const routes = [
      // ...
      {
        path: '/error/:error',
        name: 'ErrorDisplay',
        props: true,
        component: ErrorDisplay
      }
    ]
    

Take note of how the route’s `path` has `:error` on it. This is a dynamic segment, which we’re giving **ErrorDisplay** access to as a prop with `props: true`. Again, if dynamic routing is new to you, check out our [Real World Vue 3 lesson](https://www.vuemastery.com/courses/real-world-vue3/dynamic-routing) on that topic.

* * *

### Accessing the error from the action

We need a way for the component that ultimately triggered the error to gain access to that error. This means a couple things. First, we need to `return` the result of our Vuex actions’ API calls so that the component that dispatched them can receive that result. Second, we need the action that _caught_ the error to `throw` it to the component that caused it, so it can `catch` it, too.

Let’s head back into the Vuex store and make those changes.

📁 **store/index.js**

      actions: {
        createEvent({ commit }, event) {
          EventService.postEvent(event)
            .then(() => {
              commit('ADD_EVENT', event)
            })
            .catch(error => {
              throw(error) // <--- throw error
            })
        },
        fetchEvents({ commit }) {
          return EventService.getEvents() // <--- return result
            .then(response => {
              commit('SET_EVENTS', response.data)
            })
            .catch(error => {
              throw(error) // <--- throw error
            })
        },
        fetchEvent({ commit, state }, id) {  
          const existingEvent = state.events.find(event => event.id === id)
          if (existingEvent) {
              commit('SET_EVENT', existingEvent)
          } else {
            return EventService.getEvent(id) // <--- return result
              .then(response => {
                commit('SET_EVENT', response.data)
              })
              .catch(error => {
                throw(error) // <--- throw error
              })
          }
        }
      }
    

With these adjustments in place, we’re ready to `catch` the error locally in the component. And when an error is caught, we’ll route to the **ErrorDisplay** view, and display that error.

* * *

### Routing to ErrorDisplay

First up, we’ll head into EventCreate and add that routing behavior there.

📁 **views/EventCreate.vue**

    onSubmit() {
      // ...
      this.$store.dispatch('createEvent', event)
        .then( () => {
          this.$router.push({
            name: 'EventDetails',
            params: { id: this.event.id }
          })
        })
        .catch( error => {
          this.$router.push({
            name: 'ErrorDisplay',
            params: { error: error }
          })
        })
    }
    

We now have some simple routing logic. If the event is created successfully, we’ll route the user to view that event’s details. If there was an error, we’ll route them to view that error. Again, here we are giving the `ErrorDisplay` route access to the `error` as a param, which will be fed in as a prop.

📁 **views/ErrorDisplay.vue**

    <template>
      <h4>Oops! There was an error:</h4>
      <p>{{ error }}</p>
    </template>
    
    <script>
    export default {
      props: ['error']
    }
    </script>
    

Next, we’ll repeat this process within the two other components that might generate an error: **EventList** and **EventDetails**. Both of these components dispatch actions that fetchEvent(s) from our mock db, so in each we need to add a `catch` wherein we route to the **ErrorDisplay** component to display the error.

📁 **views/EventList.vue**

    created() {
      this.$store.dispatch('fetchEvents')
        .catch(error => {
          this.$router.push({
            name: 'ErrorDisplay',
            params: { error: error }
          })
        })
    },
    

📁 **views/EventDetails.vue**

    created() {
      this.$store.dispatch('fetchEvent', this.id)
        .catch(error => {
          this.$router.push({
            name: 'ErrorDisplay',
            params: { error: error }
          })
        })
    },
    

Now that our error handling is all set up, we can check to make sure this is working by forcing a network error to happen. We’ll just go into the EventService file and mess up the `baseURL`, giving it the wrong string:

📁 **services/EventService.js**

    baseURL: 'http://localhost:3008', // <--- supposed to be 'http://localhost:3000'
    

Now when we create an event, or head to the **EventList** and **EventDetails** views, we’ll see this:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1626117643422.jpg?alt=media&token=3b6fe778-d80a-4f98-922a-ba7450b728f5](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1626117643422.jpg?alt=media&token=3b6fe778-d80a-4f98-922a-ba7450b728f5)

Great. We’ve now successfully implemented a simple error handling solution in our application.

* * *

What’s next?
------------

With that, we’ve come to the end of the lesson and we’ve concluded the code for this course. There are a number of Vuex features we did not cover, so in the next and final lesson I’ll give a brief overview of what those are and point you in the right direction to continue your learning journey with state management with Vuex.

![](/images/folder-link-blue.svg)

### Lesson Resources

##### Source Code:

*   [Starting Code](https://github.com/Code-Pop/Vuex_Fundamentals/tree/L5-start)
    
*   [Ending Code](https://github.com/Code-Pop/Vuex_Fundamentals/tree/L5-end)
    
*   [Dynamic Routing](https://www.vuemastery.com/courses/real-world-vue3/dynamic-routing)
