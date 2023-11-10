Updating State
==============

If we take a look at our example application, we are successfully pulling our global Vuex `user` state into our local **EventCreate** component. But we’re currently just logging our new event to the console. In this lesson, we want to actually post that event to our mock database, and to add the event to our Vuex state. We can achieve this with another feature within Vuex: **mutations.**

* * *

Posting our Event
-----------------

Before we deal with any of the Vuex behavior, let’s implement the steps to post the event to our mock database with JSON Server. This means we need to add a new post request to our **EventService.js** file.

📁 **services/EventService.js**

    import axios from 'axios'
    
    const apiClient = axios.create({
      baseURL: 'http://localhost:3000',
      withCredentials: false,
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json'
      }
    })
    
    export default {
      getEvents() {
        return apiClient.get('/events')
      },
      getEvent(id) {
        return apiClient.get('/events/' + id)
      },
      postEvent(event) { // new post request
        return apiClient.post('/events', event)
      }
    }
    

As you can see, the `postEvent` method takes in the `event` and _posts_ it to our `/events` endpoint.

**Sidenote:** Because we’re using the locally running version of JSON Server, our `baseURL` is: `'http://localhost:3000'` — this is different from the baseURL we used in Real World Vue 3 when using the web version of JSON Server.

Now that our `postEvent` method is ready, we can use it in our **EventCreate** component. We’ll do so within the `onSubmit` method, after importing **EventService** into this component.

📁 **views/EventCreate.vue**

    <script>
    import { v4 as uuidv4 } from 'uuid'
    import EventService from '@/services/EventService.js'
    
    export default {
      ...
      methods: {
        onSubmit() {
          this.event.id = uuidv4()
          this.event.organizer = this.$store.state.user
          EventService.postEvent(this.event)
            .then(() => {
              // add event to Vuex state
            })
            .catch(error => {
              console.log(error)
            })
        }
      },
    ...
    }
    </script>
    

Here, we’re simply calling the `postEvent` method, which _posts_ our event (`this.event`) to our mock database. I’ve also included the skeleton for additional steps; we’ll want to `then` add the event to our Vuex state, and do some error handling if we `catch` any errors.

We’ll get to that Vuex step in a moment (error handling is in a later lesson), but first let’s test this out in the browser by creating a new dummy event, hitting submit, then heading over to the Events tab. If we did everything correctly, we should see our new event in that list.

* * *

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1620672533846.jpg?alt=media&token=51bb61f6-ef59-436c-be1a-7c85274b7266](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1620672533846.jpg?alt=media&token=51bb61f6-ef59-436c-be1a-7c85274b7266)

* * *

So far so good! We’re successfully posting our new event to our mock database. Remember from Real World Vue 3, when our **EventList** component is `created`, it’s fetching all of our events from our mock database. That’s why we’re seeing it displayed in the Events tab.

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
    

Now that we’ve got our new events posting to the database, we can move on to fleshing out that skeleton and adding the event to the Vuex state. But first, I want to show you an alternative syntax for adding new properties to our `event` object.

Instead of doing:

📁 **views/EventCreate.vue**

    onSubmit() {
      this.event.id = uuidv4()
      this.event.organizer = this.$store.state.user
      EventService.postEvent(this.event)
        // code omitted 
    }
    

We can instead use the spread operator, like this:

📁 **views/EventCreate.vue**

    onSubmit() {
      const event = {
        ...this.event,
        id: uuidv4(),
        organizer: this.$store.state.user
      }
      EventService.postEvent(event)
        // code omitted
    }
    

By using the spread operator `...`, we’re able to take the original `event` object and “spread” out its properties onto a new object along with the additional properties we need (`id` and `organizer`). If you prefer to use the syntax we were using before, that’s up to you. This is just a more modern syntactical approach.

* * *

Updating our State
------------------

In order to add our event to the Vuex state, we’ll need to write our first mutation. As I mentioned in the first lesson of this course, mutations are what we use within Vuex to update or _mutate_ the state. Let’s get started writing a mutation that can add an event to our state.

First we’ll give our state an `events` array, which we can add new events to.

📁 **store/index.js**

    export default createStore({
      state: {
        user: 'Adam Jahr',
        events: [] // new events array
      },
      mutations: {
        ADD_EVENT(state, event) { // our first mutation
          state.events.push(event)
        }
      }
      ...
    )}
    

As you can see, our `ADD_EVENT` mutation takes in two arguments: the Vuex `state` itself, and the `event` that we want to `push` onto our new `events` array within that state. That’s as simple as it is. We’ve just written the code that we can call to add events to our state.

Now we need to call or _commit_ this mutation from within our **EventCreate** \*\*component. We’ll do so within the `onSubmit` method.

📁 **views/EventCreate.vue**

    onSubmit() {
      const event = {
        ...this.event,
        id: uuidv4(),
        organizer: this.$store.state.user
      }
      EventService.postEvent(event)
      .then(() => {
        this.$store.commit('ADD_EVENT', event)
      })
      .catch(error => {
        console.log(error)
      })
    }
    

This new code within the `.then()` is fairly literal. We’re accessing our global Vuex store (`this.$store`) and telling it to `commit` the mutation that we’ve specified in the first argument (`ADD_EVENT`), and we’re passing in the event that we want to add.

As for terminology: This second argument is called the _payload_. And `commit` is just Vuex syntax that means we are calling our mutation, which _commits_ us to a new state within our app. And if you’re wondering why I put the mutation name in all caps, that is a convention that is common for mutations. It’s entirely optional. The benefit that I personally like about the all caps is that it makes it very visually obvious when a state change is set to occur. Almost like our code is yelling `I_WILL_CHANGE_STATE`!

* * *

### Is it working?

In order to demonstrate that we’re indeed committing the mutation and adding a new event to our state, I’m going to add a `div` to the bottom of the **EventCreate** template, below the submit button, to display our Vuex `events` state.

📁 **views/EventCreate.vue**

    <div>{{ $store.state.events }}</div>
    

Again, this is just to demonstrate that our code is actually working. At the time I’m producing this course, the Vue 3 Developer Tools aren’t yet released. Once they’re ready, they should contain a Vuex tab that will show you a timestamped record of every mutation that was committed, allowing you to do “time-travel debugging” to see what the state was at the times of those mutations.

But for now, we’re going to hack a solution to prove that our mutation is in fact working. When I create a new dummy event and hit submit, voila:

* * *

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1620672533847.jpg?alt=media&token=e49aaa57-f1cc-4405-9dc0-7621b8323440](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1620672533847.jpg?alt=media&token=e49aaa57-f1cc-4405-9dc0-7621b8323440)

At the bottom of the page, we can see our Vuex `events` state is being displayed, complete with the new event that we just _committed_.

We can go ahead and delete that `<div>{{ $store.state.events }}</div>` line from our template since it was just for demo purposes.

* * *

Coming up next
--------------

We’re making great progress. We’ve now implemented two pieces of Vuex state: `user` and `events` and we’ve learned how to use a mutation to update our state, all while posting events to our mock database. While our code works the way we have it, we’re not yet following best practices with Vuex.

In the next lesson, we’ll learn about Vuex actions and how they can wrap our mutations with some conditional and/or asynchronous logic for Vuex code that can scale. See you there!

### Lesson Resources

##### Source Code:

*   [Starting Code](https://github.com/Code-Pop/Vuex_Fundamentals/tree/L3-start)
    
*   [Ending Code](https://github.com/Code-Pop/Vuex_Fundamentals/tree/L3-end)
