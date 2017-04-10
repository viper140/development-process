# Vue Project Guide

## 1. Scaffold a Project 

### 1.1. Initialize a Vuejs Project
Run `yarn global add vue-cli@latest` to install the latest vue-client.  

Run `vue init webpack my-project` to create a new project named `my-project`. When asked, use recommended settings and answer "Y" to vue-router, ESLint, unit test and e2e test.  

### 2.2. Customize Configuration and Install Packages
To make the root folder less crowded, move the `index.html` to the `src/` folder.
Change the line `template: 'index.html'` to `template: 'src/index.html'` in both `build/webpack.dev.conf.js` and `build/webpack.prod.conf.js`.

Edit `package.json` file to update vue and vue-router to the latest verion. 
Run `yarn install` to  install all packages used in the project. 
Run `yarn run dev` to check that the site is up and running correctly. 

Finally, copy [resources/gitignore](./resources/gitignore) to the `.gitignore` file in the project folder. 

## 2. Architecture Overview
The architecture is about the relationship among components. 

There are three communication mechanism between two components: props for a parent to sync data with a child, slot for a parent to insert conent to child,  event for a child to notify a parent, and finally a global store is to sync data globally. 

We should use functional stateless components as much as possible. Therefore the primary interaction between a parent and child is via props and events. 

A component uses states when 
* it communicates with backend or 
* it is used/controlled by multiple components. 

## 3. Entity Editing Pattern
A common use case in a business application is to retrieve an entity from a backend server and edit it using an HTML form. The suggested method is using the `computed` option of a component to 1) retrieve the entity using a store action; 2) make a copy of the entity use either `return Object.assign({}, entity)` or if only a few attributes are neeed, `return { {attr1, attr2 } = { ...entity }}`. 

Then bind the `computed` copy to a form with all input fields using `v-model` directive. 

Finally, when a user submits/saves the changes, raise an event and send the `computed` copy to the backend server.  

## 4. Workflow
The workflow describes the steps needed to create new component in the frontedn. 

### Steip 0: Create Data Model
Create a data model for every api call request in `infrastructure/api_client/`. Use the data model when a component calls an action and transform the request to an actual HTTP call. 

Though a store action is used to bridge the call from a component to an HTTP call, there is no need to use the data model in store action definition for two reasons: 1. Vuex proxies all action calls and the data types are lost. 2. There is no data process inside the action.   

### Step 1: Create HTTP request
Create all remote requests in a file inside the `infrastructure/api_client/`. 

### Step 2: Create Store State
* All store putter/getter using as naming convention of "setMyState" and "getMyState". For async actions, using the most meaningful business name for the action. Use `module/operation-name` as a pattern to create method names in their corresponding getter, setter or action file.   
* create the initial states, actions, getters and mutations and put them into `index.ts`.
* add the module to `infrastructure/store/index.ts`.
* It may be a good idea to check the existing state data before send remote request in a fetch function. However, careful design and implementation are required to make sure that the cached data is valid. 

The store mutation and action methods only take one payload argument. 

### Step 3: Create a component
* Use [`vue-class-component`](https://github.com/vuejs/vue-class-component) syntax to create a component.
* Call the fetch data in `created` lifecycle callback to fetch data and put it in the store. 
* Initially the component props have no value, therefore it is necessary to use `v-if` to guard the template content to avoid reading fields of an undefined object (either as a prop or a computed value). 

### Step 4: Pass data to a presentation component
* define the data as a prop in a presentation component.
* in the container component, check if the data is available in presentation component template using `v-if` before pass it to the child presentation component. 

### Step 5: Raise event in a presentation component
* raise an event with a name that starts with an event source name and terminates with a verb ending with -ing or -ed (Closing/Closed). 
* an event should have have a `Event` postfix in its name. For example, `defaultChangedEvent`. 

### Step 6: Handle event in a container component 
* a paraent uses `@eventName="theActionHandler"` to catch the event.
* an event handler should have a "Handler" postfix and is often defined as a component method.
* if it is a remote call, the event handler call the mapped action to call remote server.

### Step 7: Process result 
* get the result or handle error
* update local state based on the call result. 

## 5. Error Handling
TBD. 