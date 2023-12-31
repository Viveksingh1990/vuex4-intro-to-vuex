Global State
============

In this lesson, we’re going to implement some global state within our example application. You can access [the repo](https://github.com/Code-Pop/Intro-State-Management) for this course and `git checkout L2-start` to reach the starting code for this lesson.

Once you open up the starting code and have ran `npm install` to grab its dependencies, you’ll notice that this project looks just like the app we built in Real World Vue 3, with the addition of a new file in our **views** directory: **EventCreate.vue**. You’ll also notice that this new component has been added as a route within our **router/index.js** file. So what exactly is the purpose of this new component?

* * *

Getting Acquainted with EventCreate
-----------------------------------

**EventCreate** is essentially a form whose purpose is to create new events, as the component’s name suggests. If we were to run `npm run serve` within our project, we’ll see the **EventCreate** component showing up under the **Create Event** route in our app:

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1617118174355.jpg?alt=media&token=d132e9ce-0d25-458d-b0d6-c4e148e53471](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1617118174355.jpg?alt=media&token=d132e9ce-0d25-458d-b0d6-c4e148e53471)

To understand how this component is working, we’ll start off by looking at its `template`.

📁 **views/EventCreate.vue**

    <template>
    <h1>Create an event</h1>
    
    <div class="form-container">
    
      <form @submit.prevent="onSubmit">
        <label>Select a category: </label>
        <select v-model="event.category">
          <option
            v-for="option in categories"
            :value="option"
            :key="option"
            :selected="option === event.category"
          >{{ option }}</option>
        </select>
    
        <h3>Name & describe your event</h3>
    
        <label>Title</label>
        <input
          v-model="event.title"
          type="text"
          placeholder="Title"
        >
    
        <label>Description</label>
        <input
          v-model="event.description"
          type="text"
          placeholder="Description"
        />
    
        <h3>Where is your event?</h3>
    
        <label>Location</label>
        <input
          v-model="event.location"
          type="text"
          placeholder="Location"
        />
    
        <h3>When is your event?</h3>
        <label>Date</label>
        <input
          v-model="event.date"
          type="text"
          placeholder="Date"
        />
    
        <label>Time</label>
        <input
          v-model="event.time"
          type="text"
          placeholder="Time"
        />
    
        <button type="submit">Submit</button>
      </form>
    
    </div>
    </template>
    

As you can see, we’re using a series of `label` and `input` elements to construct the `form`, and each input is bound with `v-model` to our component’s `data`. By the way, if constructing Vue forms is brand new to you, you’ll want to check out the lesson on [forms and v-model](https://www.vuemastery.com/courses/intro-to-vue-3/forms-and-v-model-vue3) in our **Intro to Vue 3** course.

Instead of relying on the form’s default submission behavior, we’re running the `onSubmit` method when the `button` is clicked. For now, we’re simply logging the `event` data to the console.

📁 **views/EventCreate.vue**

    <script>
    export default {
      data () {
        return {
          categories: [
            'sustainability',
            'nature',
            'animal welfare',
            'housing',
            'education',
            'food',
            'community'
          ],
          event: {
            id: '',
            category: '',
            title: '',
            description: '',
            location: '',
            date: '',
            time: '',
            organizer: ''
          }
        }
      },
      methods: {
        onSubmit() {
          console.log("Event:", this.event)
        }
      }
    }
    </script>
    

To test this out, we can input some dummy data into our form and see what shows up in the browser’s console.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1617118174356.jpg?alt=media&token=2cdce0e5-eefd-47aa-90b4-14ca4aa4e6bd](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1617118174356.jpg?alt=media&token=2cdce0e5-eefd-47aa-90b4-14ca4aa4e6bd)

As we can see in the console, our `event` is being logged.

Now that we’re oriented to how this new component is working, we can move on to introducing the concept of global state into our app, which we’ll access from within this component specifically.

* * *

Adding Global State
-------------------

Did you notice that we have a couple empty fields in our `event` object that was logged to the console? The event was missing an `id` and an `organizer`. As for the `id`, we’ll solve that later in this lesson, since it requires using a simple library. For now, we’re going to focus on that `organizer` field.

For our app, we’ll assume that the user who created the event is the `organizer` of that event. With this in mind, we can add a `user` to our Vuex state, which we can access from our **EventCreate** component, and add that `user` as our event’s `organizer`.

So let’s head into our Vuex store and add that `user` state.

📁 **store/index.js**

    import { createStore } from 'vuex'
    
    export default createStore({
      state: {
        user: 'Adam Jahr'
      }
      ...
    })
    

For demo purposes, I’ve added my name here in string format as the `user`. You’re welcome to use whomever’s name you’d like.

Like we learned in the previous lesson, this `state` is globally accessible throughout our application. Back in our **EventCreate** component, we can access our `user` state by writing `this.$store.state.user`.

We could technically add that state directly to the `event` data object itself, like so:

📁 **views/EventCreate.vue**

      data () {
        return {
          ...
          event: {
            id: '',
            category: '',
            title: '',
            description: '',
            location: '',
            date: '',
            time: '',
            organizer: this.$store.state.user
          }
        }
      },
    

However, it’s recommended to keep your `data` separate from your Vuex state to avoid reactivity issues. For example, if the value of your state’s `user` changed, that change won’t be reflected within the data property where you initially called it, since the data is only created once when the component is constructed.

So to avoid accidentally submitting stale state attached to the `event`, we can instead set the organizer for our event when we submit the form. That way, we are only ever accessing the `user` state when we’re ready to create the event.

📁 **views/EventCreate.vue**

    methods: {
      onSubmit() {
        this.event.organizer = this.$store.state.user
        console.log("Event:", this.event)
      }
    }
    

We can test this out by creating a dummy event in the browser. We should see that the event that is logged to the console now has an organizer, based on whatever value lives in our `user` state.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.opt.1617118183269.jpg?alt=media&token=66a063d2-6f72-4110-bca4-c9fbc9323e99](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.opt.1617118183269.jpg?alt=media&token=66a063d2-6f72-4110-bca4-c9fbc9323e99)

Great. We’re now successfully accessing our global Vuex state from within our **EventCreate** component, and adding it to the event we’re creating.

* * *

Adding the id
-------------

Now we’re ready to solve the missing `id` field. There are different ways to add a unique ID to objects within your Vue apps, and for this course we’re going to be using the popular [uuid library](https://www.npmjs.com/package/uuid), which should already be installed as a dependency within your project (if not, go ahead and install it now). To make use of it, we’ll need to import it into our **EventCreate** component.

📁 **views/EventCreate.vue**

    <script>
    import { v4 as uuidv4 } from 'uuid'
    export default {
      ...
    }t
    </script>
    

Now when and where do we want to generate the new `id` and add it to our `event`? We’ll do so in `onSubmit`, similar to how we added the `organizer`.

📁 **views/EventCreate.vue**

    methods: {
      onSubmit() {
    		this.event.id = uuidv4()
        this.event.organizer = this.$store.state.user
        console.log("Event:", this.event)
      }
    }
    

This way, as we submit the `event`, we’re giving it the final property values that it needs before sending it off (or console.logging for now).

Let’s test this out in the browser to make sure everything is working as expected.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.opt.1617118183270.jpg?alt=media&token=5efa0451-254e-4f37-a9f1-83d92055fb0f](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.opt.1617118183270.jpg?alt=media&token=5efa0451-254e-4f37-a9f1-83d92055fb0f)

Success! We now have a complete `event` with all of the necessary fields filled in. Of course, we don’t just want to be logging these events to the console, we want to actually submit them to our database. In the next lesson, we’ll learn how to use JSON Server to _post_ these events to our mock database

* * *

A note about JSON Server
------------------------

If you took the **Real World Vue 3** course, you’ll remember that we used [My JSON Server](https://my-json-server.typicode.com/) as our mock database, where we pulled our events from. Specifically, we used the **web version** of this helpful library because we needed the ability to deploy our app and still have access to this mock database, which the web version allowed us to do.

However, like I mentioned in that course, that web version has its limits. One of those limitations is in regards to _posting_ new items to the mock database: “Changes are faked and aren’t persisted”.

In other words: While using the web version of My JSON Server, whenever we post an event to our mock database, that event won’t actually be added to our **db.json** file. So for this course, we’re using the [locally running version](https://www.npmjs.com/package/json-server) of JSON Server instead. This version does not have those same limitations, and our new events will indeed be persisted within the **db.json** file, once we _post_ them to it.

If you don’t already have this library installed globally on your device, you can do so by running `npm install -g json-server`

You may have even noticed that when you ran `npm run serve` earlier, the console output these lines:

    \{^_^}/ hi!
    
      Loading db.json
      Done
    
    Resources
      http://localhost:3000/events
    
      Home
      http://localhost:3000
    

That’s because I’ve added onto the `serve` command within our project’s package.json file:

**📃 package.json**

    "scripts": {
        "serve": "vue-cli-service serve & json-server --watch db.json",
        ...
      },
    

Now, the `serve` command not only spins up our app with a local development server, but it also runs json-server, telling it to “watch” our **db.json** file (our mock database) for changes.

* * *

Coming up next…
---------------

In the next lesson, we’ll learn how to post events to our mock database as well as add these new events to our Vuex state.

### Lesson Resources

##### Source Code:

*   [Starting Code](https://github.com/Code-Pop/Vuex_Fundamentals/tree/L2-start)
    
*   [Ending Code](https://github.com/Code-Pop/Vuex_Fundamentals/tree/L2-end)
