Fetching State
==============

In the previous lesson, we learned how to commit a mutation to update our Vuex state. While our code works the way we have it, we’re not following best practices with Vuex. It is recommended by Core Vue Team Members to always wrap your mutations within actions.

So what is an action, exactly? An action allows us to program more nuanced behavior as it pertains to our app-wide state management. For example, what if we needed to run some conditional logic, or wait for an API call to return before we determine whether we should commit a mutation or not? We can encapsulate all of that behavior within an action.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1623779942680.jpg?alt=media&token=6c416ac1-6651-498c-ba40-b034b7b2fca3](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1623779942680.jpg?alt=media&token=6c416ac1-6651-498c-ba40-b034b7b2fca3)

Remember in our Intro to Vuex lesson, we looked at how an action could be written to commit a mutation that sets the `isLoading` state to `true`, then makes an API call, and when that call’s response returns, it commits a mutation to set the `isLoading` state to `false` before committing a mutation to set the `todos` state equal to the API call’s response. This is a great example of the value that actions provide, as a logical wrapping layer around our state changes.

Let’s look at a different example, from the non-Vue world, to make this abstract concept more concrete.

* * *

An Action Analogy
-----------------

Let’s say I ask my friend to pick up some bread from the store. There’s a big difference between asking for someone to go get bread vs them actually delivering bread to your breadbox. Similarly, there is a difference between an action that checks if there is “bread” on the server and only adds it the Vuex state if it was able to retrieve any.

There could be plenty of reasons why my friend wouldn’t be able to pick up the bread. Her car may break down on the way to the store, or the store might be out of bread. So actions are more like expressing an _intent_ or _desire_ for something to happen, for some change to be made to the state, depending upon some surrounding circumstances, and the mutation is the _fulfillment_ of that intention.

In this example, the mutation here would be something like `ADD_BREAD`, which adds the bread to the state, \*\*\*\*whereas the action is more like `fetchBread`, which requests “bread” from the server and only commits the `ADD_BREAD` \*\*\*\*mutation if it finds a loaf.

Hopefully this silly example helps you understand the difference between actions and mutations and how they work together to help maintain our application state. Let’s now head into the app we’ve been building and add some actions where necessary.

* * *

Our First Action
----------------

If we head back into our **EventCreate** component, we’ll see that we’re directly committing our `ADD_EVENT` mutation from the component itself.

📁 **views/EventCreate.vue**

    methods: {
        onSubmit() {
          const event = {
            ...this.event,
            id: uuidv4(),
            organizer: this.$store.state.user
          }
          EventService.postEvent(event)
          .then(() => {
            this.$store.commit('ADD_EVENT', event) // directly committing mutation
          })
          .catch(error => {
            console.log(error)
          })
        }
      }
    

We need to refactor things so that we instead use an action to trigger this behavior. Let’s cut out this code block that I’ve commented out below.

📁 **views/EventCreate.vue**

    methods: {
        onSubmit() {
          const event = {
            ...this.event,
            id: uuidv4(),
            organizer: this.$store.state.user
          }
          // cut out this code block
          // EventService.postEvent(event)
          // .then(() => {
          //   this.$store.commit('ADD_EVENT', event)
          // })
          // .catch(error => {
          //   console.log(error)
          // })
        }
      }
    

We’ll then head into our Vuex store file and add our first action, called `createEvent`, and paste that code into it.

📁 **store/index.js**

    actions: {
        createEvent({ commit }, event) {
          EventService.postEvent(event)
          .then(() => {
            commit('ADD_EVENT', event)
          })
          .catch(error => {
            console.log(error)
          })
        }
      },
    

There’s a few new things going on here…

First, notice that we’re passing in `{ commit }` to our action. This is part of what’s called the “context object” and simply gives us the ability to run commits on our store.

Second, notice how we’re now just saying `commit('ADD_EVENT', event)` and not `this.$store.commit('ADD_EVENT', event)` — that is because we are within the store and don’t need to access it as a global object like we needed to from within a component.

Another step we need to take now is to import our **EventService** into the store file (and delete that import statement from the **EventCreate** component).

📁 **store/index.js**

    import { createStore } from 'vuex'
    import EventService from '@/services/EventService.js'
    

This will ensure we’re able to run the API calls that live within that service file.

* * *

Dispatching Actions
-------------------

Now that our action is ready to run, how do we actually call it? In Vuex terminology, whenever we run an action, we “dispatch” it. So let’s now head back into **EventCreate** and dispatch our new action, passing in the `event` as the payload just like we did when we committed the mutation earlier.

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
    

We can test this out by heading into the browser and creating a new dummy event, with a title like “Learning Luncheon”. Once we hit submit, we should be able to route over to the **EventList** and see our new event showing up.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1623779942681.jpg?alt=media&token=e9db2776-0667-4d6d-a8e2-c91709e44cde](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1623779942681.jpg?alt=media&token=e9db2776-0667-4d6d-a8e2-c91709e44cde)

Great! We got it to work. Now, we’re dispatching our `createEvent` action, which handles the _posting_ of our `event` to the mock db and adds that `event` to our Vuex state by committing the `ADD_EVENT` mutation.

If at this point you’re thinking: \*Sure, but that was a lot of extra code for the exact same behavior…\*🤔

You’re right! But it’s a best practice for a reason because it builds in scalability to your app. While you might not need any conditional logic and/or asynchronous code to wrap your mutation today, when you discover that you need it next month, you can simply add it into your action and not have to refactor a bunch of code throughout your app.

* * *

Fetching Events with Actions
----------------------------

Now that we’ve written our first action, we can continue building our action-writing muscles with a couple more use cases in our example app. Specifically, I’m referring to our **EventList** and **EventDetails** components.

📁 **views/EventList.vue**

    created() {
        EventService.getEvents()
          .then(response => {
            this.events = response.data
          })
          .catch(error => {
            console.log(error)
          })
      }
    

📁 **views/EventDetails.vue**

    created() {
        EventService.getEvent(this.id)
          .then(response => {
            this.event = response.data
          })
          .catch(error => {
            console.log(error)
          })
      }
    

Notice how these components are getting the event(s) from our mock db when the components are `created`. Since our `events` need to be available as app-wide state, we’ll want this behavior to be handled within Vuex actions instead. Remember, we already have `events` in our state.

📁 **store/index.js**

    state: {
        user: 'Adam Jahr',
        events: []
      },
    

Now we just need to be adding all of the events that we fetch from the db to that state. So let’s start by creating an action that fetches all of the events from the mock db. This will look very similar to the code we originally had in our **EventList**’s `created` hook.

📁 **store/index.js**

    actions: {
        // .
        fetchEvents({ commit }) {
          EventService.getEvents()
            .then(response => {
              commit('SET_EVENTS', response.data)
            })
            .catch(error => {
              console.log(error)
            })
        }
      },
    

Within this new `fetchEvents` action, we’re now able to run the `getEvents` call from within Vuex. Notice how once this action’s API call receives a response, it commits a new mutation: `SET_EVENTS`, passing in that response itself (the events).

Here’s that new mutation:

📁📁 **store/index.js**

    mutations: {
        ADD_EVENT(state, event) {
          state.events.push(event)
        },
        SET_EVENTS(state, events) {
          state.events = events
        }
      },
    

The `SET_EVENTS` mutation simply sets the `events` in our state equal to the `events` that it’s handed from the API call’s response.

Now that our `fetchEvents` action is written, we can dispatch it from the **EventList** component.

* * *

### Dispatching fetchEvents

When the **EventList** component is `created`, it will dispatch the `fetchEvents` action, which loads our Vuex state with the db’s `events`.

📁 **views/EventList.vue**

    created() {
      this.$store.dispatch('fetchEvents')
    }
    

We’re not done yet though, because this component’s template needs those `events` in order to create an `EventCard` for each of them.

📁 **views/EventList.vue**

    <EventCard v-for="event in events" :key="event.id" :event="event" />
    

Because those `events` are now housed in the Vuex store, we can’t set our local component `data` equal to them like we were doing before.

📁 **views/EventList.vue**

    export default {
      // ...
      data() {
        return {
          events: null
        }
      },
      created() {
        // previous code
        EventService.getEvents()
        .then(response => {
          this.events = response.data
        })
        .catch(error => {
          console.log(error)
        })
      }
    }
    </script>
    

We instead need to grab the `events` from where they live within the Vuex store. We can do this via a computed property. (If computed properties are new to you, checkout out [this lesson](https://www.vuemastery.com/courses/intro-to-vue-3/computed-properties-vue3) in our Intro to Vue 3 course.)

📁📁 **views/EventList.vue**

    export default {
      // ...
      created() {
        this.$store.dispatch('fetchEvents')
      },
      computed: {
        events() {
          return this.$store.state.events
        }
      }
    }
    </script>
    

Now, our template will be able to access our `events` state through this new local `events` computed property, which returns them: `this.$store.state.events`

If you’re wondering why we’re using a computed property for this, that is because computed properties allow us to retain the reactivity of the Vuex state. In other words, if and when the global `events` in our Vuex state gets updated, our local `events` computed property will recalculate (because its dependency, the state, has changed). This insures our local `events` will always be synced up with our global `events` state.

We’re now going to repeat a similar process to refactor the code in **EventDetails** to use Vuex.

* * *

The fetchEvent Action
---------------------

To get started, we’re essentially going to copy the code that is currently in **EventDetails**’ `created` hook.

📁 **views/EventDetails.vue**

    created() {
        EventService.getEvent(this.id)
          .then(response => {
            this.event = response.data
          })
          .catch(error => {
            console.log(error)
          })
      }
    

And paste it into a new `fetchEvent` action we’ll add to our Vuex store.

📁 **store/index.js**

    fetchEvent({ commit }, id) {  
      EventService.getEvent(id)
        .then(response => {
          commit('SET_EVENT', response.data)
        })
        .catch(error => {
          console.log(error)
        })
    }
    

The difference here is now we can say `getEvent(id)` instead of `this.id`. Also, notice how when the `response` is received, this action is committing a new `SET_EVENT` mutation, which sets the `event` state within our app equal to the `event` we just retrieved by `id`.

📁 **store/index.js**

    state: {
      // ...
      event: {}
    },
    mutations: {
      // ...
      SET_EVENT(state, event) {
        state.event = event
      }
    },
    

This ensures we have a currently active `event` in our app, which we can then access from within the component that needs it: **EventDetails**. Let’s head into that component now and dispatch the `fetchEvent` action, and pull in this new `event` state through a computed property.

📁 **views/EventDetails.vue**

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
    

To sum up what we just accomplished: we’re now dispatching the `fetchEvent` action, and passing in the `id` prop as the payload. This gives the API call within `fetchEvent` what it needs to retrieve the event that we’re displaying the details of in this component. We’re then pulling in that event we retrieved through our computed property, which is synced up to the `event` in our Vuex state.

There we have it. We’ve refactored our app to now make all of its API calls via Vuex actions, and synced up our component’s to the Vuex state it depends on via computed properties.

* * *

A Performance Enhancement
-------------------------

Before we go, there is a performance boost we can implement within our code. What happens if we’re on the **EventList** view, and we click on an event? Right now, we’re routed to **EventDetails**, which dispatches `fetchEvent`, triggering an API call to get that event by its `id` so we can display its details.

But what if we already have that event stored in our Vuex state? We can save ourselves (and the user) an API call if we already have the event we need!

Let’s add that performance enhancement to the `fetchEvent` action.

📁 **store/index.js**

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
          console.log(error)
        })
      }
    }
    

Now, we’ll run a check to see if we can find the `event` we’re about to make an API call for already existing within our `events` state. If it in fact exists, we’ll simply `SET_EVENT` and be done with it. If the event does not exist, then of course we’ll need to go `getEvent` from our mock db, and then `SET_EVENT`.

* * *

Next steps
----------

We covered a lot of ground in this lesson. Actions are arguably the most complex feature within Vuex, since they are the _actors_ that handle a lot of the heavy lifting. So what will we cover next?

Currently, when we fill out our **EventCreate** form and hit submit, we aren’t taken anywhere. We should be routed to the new event we just created. In the next lesson, we’ll get that working, along with implementing a simple yet elegant solution for handling the errors that might happen within our Vuex actions. See you there!

### Lesson Resources

##### Source Code:

*   [Starting Code](https://github.com/Code-Pop/Vuex_Fundamentals/tree/L4-start)
    
*   [Ending Code](https://github.com/Code-Pop/Vuex_Fundamentals/tree/L4-end)
