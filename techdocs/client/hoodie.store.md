# Hoodie.Store

*Source: hoodie/src/lib/store/scoped.js*

This class defines the API that `hoodie.store` (local store) and `hoodie.open` 
(remote store) implement to assure a coherent API. It also implements some
basic validations.

The returned API can be called as function returning a store scoped by the 
passed type, for example

<pre>
    var todoStore = hoodie.store('todo');
    
    todoStore.findAll().then(showAllTasks);
    todoStore.update('id123', {done: true});
</pre>

An important thing to note is that storing and accessing objects with hoodie always means accessing you personal objects. All stored data has a fixed association to the user who created them. So you won't be able to access other user's data by default. 

Never the less there are some community contributed solutions, available as [Hoodie Plugins](http://). Each one offers a different level of data privacy:


* [hoodie-plugin-shares](https://github.com/hoodiehq/hoodie-plugin-shares) share particular store objects, with particular rights to particular people. Also can invite people by mail.
* [hoodie-plugin-global-share](https://github.com/hoodiehq/hoodie-plugin-global-share) shares particular store objects to all people within the same application.
* [hoodie-plugin-punk](https://github.com/olizilla/hoodie-plugin-punk) share just everything to everyone within the same application.


**Attention!** Please note that most of these are community contributions and may have flaws or are just outdated. Always feel free to adopt an ophaned plugin of contribute your own.



## Methods

### store
`hoodie.store(_type_, _id_)`

It is most likely, that your application will have more than one type of store object. Even if you have just a single object `hoodie.store(type)` 
comes handy. Say you have to work with objects of the type `todo`, you usually
do something like the following:

<pre>
    hoodie.store.add('todo', { title: 'Getting Coffee' });
    hoodie.store.findAll('todo').done(function(allTodos) { /* ... */ });
</pre>

The `hoodie.store` method offers you a short handle here, so you can create
designated store context objects to work with:

<pre>
    var todoStore = hoodie.store('todo');

    todoStore.add({ title: 'Getting Coffee' });
    todoStore.findAll().done(function(allTodos) { /*...*/ });
</pre>

The benefit of this variant might not be clear at first glance. The primary benefit is, that you must set your object type only 
once. So in case you want to rename you object type *"todo"* to *"things-to-be-done"*
in a later phase of development, you have to change this in significantly fewer
areas on you application code. By this you also avoid typos by reducing the 
amount of occurrences, where you can make the mistake at.

Imagine having the type of *"todoo"* with a double o at the end. This would be a 
dramatic bug if it comes to storing a new todo object, because when reading all
"todo" (with single o this time) the new entries can't be found.

You can also create a very particular store, to work with access to just one specific stored object.

<pre>
    var singleStore = hoodie.store( 'todo', 'id123' );
</pre>

For the call like illustrated in the last example, only a minimal subset of functions will be available on the created store context. Every method those purpose is to target more than one stored object, will be left out (f.e. findAll). This is because we already specified a particular object form the store to work with.

### validate

`hoodie.store.validate(object, _options_)`

By default `hoodie.store.validate` only checks for a valid object `type` and object `id`. The `type` as well as the `id` may not contain any slash ("/"). 
This is due to the format, hoodie stores your object into the database.
Every stored database entry has an internal identifier. It is a combination of both, formatted as *"type/id"*. Therefore it is absolutely permitted to have a slashes in neither of both.

All other characters are allowed, though it might be the best, to stick with
alphabetical characters and numbers. But you are still free to choose.


If `hoodie.store.validate` returns nothing, the passed **object** is valid. 
Otherwise it returns an **[HoodieError]<#>**.


### save

`hoodie.store.save(type, _id_, properties)`

Creates or replaces an an eventually existing object in the store, that is of the same `type` and the same `id`.

When the `id` is of value *undefined*, the passed object will be created from scratch.

**IMPORTANT**: If you want to just to partially update an existing object, please see `hoodie.store.update(type, id, objectUpdate)`. The method `hoodie.store.save` will completely replace an existing object.

<pre>
	var carStore = hoodie.store('car');
	
	carStore.save(undefined, {color: 'red'});
	carStore.save(abc4567', {color: 'red'});
</pre>

### add

`hoodie.store.add(type, properties)`

Creates a new entry in your local store. It is the shorter version of a complete save. This means you can not pass the id of an existing property. In fact `hoodie.store.add` will force `hoodie.store.save` to create a new object with the passed object properties of the `properties` parameter.

<pre>
    hoodie.store
    	.add('todo', { title: 'Getting Coffee' })
    	.done(function(todo) { /* success handling */ });
    	.fail(function(todo) { /* error handling */ });
</pre>

### find

`hoodie.store.find(id)`

Searches the store for a stored object with a particular `id`. Returns a promise so success and failure can be handled. A failure occurs for example when no object

<pre>
	hoodie.store('todo')
    	.find('hrmvby9')
    	.done(function(todo) { /* success handling */ });
    	.fail(function(todo) { /* error handling */ });
</pre>


### findOrAdd

`hoodie.store.findOrAdd(type, id, properties)`

This is a convenient combination of hoodie.store.find and hoodie.store.add. Use can
use if you would like to work with a particular store object, which existence 
you are not sure about yet. Which cases would be worth using this? 
Well for example if you want to read a particular settings object, you want to 
work with in a later step.

<pre>
    // pre-conditions: You already read a user's account object.
    var configBlueprint = { language: 'en/en', appTheme: 'default' };
    var configId        = account.id + '_config';

    hoodie.store
        .findOrCreate('custom-config', configId, configBlueprint)
        .done(function(appConfig) { console.log('work with config', appConfig) });
</pre>

hoodie.store.findOrCreate takes three arguments here. All of them are required.

 * `type`       => The kind of document you want to search the store for.
 * `id`         => The unique id of the document to search the store for.
 * `properties` => The blueprint of the document to be created, in case nothing could be found.

The important thing to notice here is, that the `properties` parameter has no 
influence on the search itself. Unlike you may have used store searches 
with other frameworks, this will **not** use the `properties` parameter 
as further conditions to match a particular store entry. The only conditions the
store will be searched for are the document `type` and `id`.

Just to demonstrates the convenience of hoodie.store.findOrAdd, the below example
illustrates the more complex alternative way of find and add:

<pre>
	// IMPORTANT: BAD VARIATION
	
    // pre-conditions: You already read a user's account object.
    var defaultConfig = {language: 'en/en', appTheme: 'default'};
    var configId      = account.id + '_config';

    hoodie.store
        .find('custom-config', configId, configBlueprint)
        .done(function(appConfig) {
            console.log('work with config', appConfig);

            if(appConfig === undefined) {
                hoodie.store
                    .add('custom-config', bluePrint)
                    .done(function(newConfig) {
                        // work with the newConfig here
                    });
            }
        });
        
	// IMPORTANT: BAD VARIATION
</pre>


### findAll

`hoodie.store.findAll(type)`
`hoodie.store(type).findAll()`

With this you can retrieve all objects of a particular `type` from the store. Todos for instance. Given, that you already have existing todo objects stored, you can retrieve all of them like in the following example.

<pre>	

var todoStore = hoodie.store('todo');

todoStore
    .findAll()
    .then(function(allTodos) {
        allTodos.forEach(function(oneTodo) {
            console.log('found', oneTodo);
        });
    })
    .done(function() {
        console.log('successfully finished findAll');
    });
    
</pre>

What you really have to recognized here is that there is a mayor difference between the methods `then` and `done`. While `done` suggests that you can handle all the retrieved objects with it, actually it is `then` where `findAll` will deliver your data to. `done` on the other hand gets called when all other `then` calls have been passed. Yes you can utilize this mechanism to work with your data in several steps.

<pre>
var todoStore = hoodie.store('todo');

console.log(todoStore.findAll());

todoStore
    .findAll()
    .then(function(allTodos) {
        // get an array with all things 
        // you have on your todo list
        return allTodos.map(function(todo) {
           return todo.title;
        });
    })
    .then(function(titles) {
        // print out all the things you have todo
        titles.forEach(function(title) {
            console.log('You have to => ', title);
        });
    })
    .done(function() {
        // this gets called on the end
        console.log('successfully finished findAll');
    });
</pre>

There aren't any [callback closure functions](http://), like many other JavaScript libraries use to work with asynchronous flows. Hoodie uses so called **promises** to handle async flows. If you would like to now more about promises in hoodie, please see the [Hoodie promises Section](http://) for further details.


### update

`hoodie.store.update(type, id, properties)`
`hoodie.store.update(type, id, updateFunction)`
`hoodie.store(type).update(id, properties)`
`hoodie.store(type).update(id, updateFunction)`

In contrast to `.save`, the `.update` method does not replace the stored object,
but only changes the passed attributes of an existing object, if it exists. By this you are able to just update particular parts/attributes of your object. This is great for updating objects that are very large in bytes.

<pre>
// Example for store updates 
// using JavaScript plain objects as update parameter.
//
// A todo object could look like this:
//
// {	
//	id:'abc4567',
//	title: 'start learning hoodie', 
//	done: false,
//	dueDate: 1381536000
// }
	
var todoStore = hoodie.store('todo');

todoStore
    .findAll()
    .then(function(allTodos) {
        // just pick the first todo we can get
        var originTodo = allTodos.pop();

        console.log(originTodo.id, '=>', originTodo.dueDate)

        // update the picked todo and update it's dueDate to now
        todoStore
            .update(originTodo.id, { dueDate:(Date.now()) })
            .then(function(updatedTodo) {
                // beyond this point, please work with updatedTodo
                // instead of the originTodo, because originTodo 
                // is outdated.
                console.log(updatedTodo.id, '=>', updatedTodo.dueDate);
            });

        // update the picked todo and update it's dueDate to now
        todoStore
            .update('ID DOES NOT EXIST', { dueDate:(Date.now()) })
            .then(function(updatedTodo) {
                console.log('will never happen.');
            })
            .fail(function(error) {
                console.log('the update failed with', error);
            });
    });
</pre>

The example updates the todo object, which owns the `dueDate` of the first found todo object to `Date.now()`, which is the timestamp of right now. Every other attribute stays the same.

Further the example contains another example where we try to update an object, whose ID does not exist. This shall demostrate, how you can handle errors during updates. 

The `update` methods have a certain speciality. Beside that you can pass a plain JavaScript object with attributes updates, you can also pass a function, that manipulates the the object matched by the given `id`.

Cases when this advantage can be very useful are applying calculations or for conditioned updates. This will come mist handy when combined with `hoodie.store.updateAll`.

<pre>
// example for store updates 
// using functions as update parameter

var todoStore = hoodie.store('todo');

todoStore
    .findAll()
    .then(function(allTodos) {
        // just pick the first todo we can get
        var originTodo = allTodos.pop();

        todoStore.update(originTodo.id, function(oneTodo) {

            // Apply update only if conditions matches.
            if( Math.random() > 0.5) {
                // set dueDate to: right now
                oneTodo.dueDate = Date.now();
            }

            return oneTodo;
        }).then(function(updatedTodo) {
            console.log('update success', updatedTodo);
        }).fail(function(error) {
            console.log('failed update with', error);
        });

    });
</pre>


### updateAll

### remove

### removeAll

### decoratePromises

### trigger

### on

### unbind


## Code Example

<pre>
</pre>