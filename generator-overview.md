#What are Generators?

Sails is all about automating repetitive tasks to make your programming easier and **Generators** are no exception.  _Generators_ are command line utilities within Sails that automate the generation of files through templates within your Sails projects.  In fact, Sails core uses _generators_ to create Sails projects.  So when you type...

```sh
~/ $ sails new myProject
```

...sails is using _generators_ to build up the initial folder structure of a Sails app like this:

```javascript
myProject
       |_api
       |_assets
       |_config
       |_node_modules
       |_tasks
       |_views                
.gitignore
.sailsrc
app.js
Gruntfile.js
package.json
README.md
                
```

Other examples of _generators_ in Sails core (meaning they are built into Sails) include:

- sails-generate-adapter
- sails-generate-backend
- sails-generate-controller
- sails-generate-frontend
- sails-generate-model
- sails-generate-new
- sails-generate-views
- sails-generate-views-jade
- Although not a stand-alone module there's one other _generator_ accessed via `sails generate api`

To begin the process of generating a generator you can use `sails-generate-generator`. 

>**Note:** The idea of creating a _generator_ by invoking a _generator_ may seem like some kind of demented infinite loop but trust us it will not create a worm hole to an evil alternate universe. 

##Creating a Generator

First we need a Sails project.  If you haven't already created one go to your terminal and type:

```sh
~/ $ sails new myProject
```

`cd` into `myProject` or from any existing Sails project and create a _generator_ from the terminal named **awesome** by typing:

```sh
~/ $ sails generate generator awesome 
```

You'll know the generator was created if you see the message: `info: Created a new generator!`.

##Enabling the Generator

To enable the _generator_ you need to tell Sails about it via `\myProject\.sailsrc`. If you were using an existing generator you would link to an npm module in `.sailsrc` and then just install it with `npm install`.  Since you're developing a generator, you'll link to it directly.  To create the link go back to the terminal and `cd` into the `awesome` _generator_ folder and type:

```sh 
~/ $  pwd
```

The `pwd` command will return a fully resolved path to the _generator_ (e.g. `/Users/irl/sails_projects/myProject/awesome`).

Copy the path and then open `myProject/.sailsrc`.  Within the `modules` property add an `awesome` key and paste the path to the `awesome` _generator_ as the value. 

> **Note:** you can name the _generator_ anything you want, for now let's stick with `awesome`:

```javascript
{
  "generators": {
    "modules": {
    	"awesome": "/Users/irl/sails_projects/myProject/awesome"
    }
  }
}
```
>**Note:** Whatever name you give your _generator_ in the `.sailsrc` file will be the name you'll use from the terminal command-line to execute it.

Lastly, you'll need to do an `npm install` from the terminal in order to install the necessary modules that were added to the generator's `package.json` file.

##Using the Generator

Back at the terminal type: `sails generate awesome example`. Let's take a look at what was generated.

###What did the Generator create?

Open up your project in a text editor you'll notice that a folder called `hey_look_a_folder` was created and a file named `example` was also created: 


```javascript
/**
 * This is just an example file created at Wed Jun 04 2014 17:35:59 GMT-0500 (CDT).
 *
 * You can use underscore templates, see?
 */

module.exports = function () {
  // ...
};
``` 

The folder and file illustrate the power of the _generator_ not only to create elements but to use variables from the command-line to influence their content.

##Basic generator configuration

All of the configuration for the `awesome` _generator_ is contained in `\myProjects\awesome\Generator.js`.  The main parts of `Generator.js` is the `before()` function and the `targets` object.

###Configuring the before() function

Let's take a closer look at `Generator.js`:

<img src="http://i.imgur.com/1VOvDwe.jpg" />

Each _generator_ is passed the `scope` object which is useful when you want to otain the arguments that were entered when the _generator_ was executed as well as passing information to templates and other _generators_.

In our default `awesome` _generator_ a new key, `createdAt:` was created in our scope.  We'll take a look at this object within a template in a second.

```javascript
...
// Attach defaults
    _.defaults(scope, {
      createdAt: new Date()
    });
...
```
Next, the arguments used when executing the awesome _generator_ (e.g. `sails generate awesome <theargument>`) are available in an array from `scope.args`.  In our default `awesome` _generator_ a `filename` property was added to the scope and assigned the value of the first element of the `scope.args` array (e.g. example):

```javascript
...
scope.filename = scope.args[0];
...
```

Finally, another property (e.g. scope.whatIsThis) was added to the scope object.

```javascript
...
scope.whatIsThis = 'an example file created at '+scope.createdAt;
...
```

###Configuring the targets object

Now, let's take a look at the `targets` object in `myProject\awesome\Generator.js` to better understand how the folder (e.g. hey_look_a_folder) and file (e.g. example) were generated. 

<img src="http://i.imgur.com/D2YAqBl.jpg?1" />

The `template` and `folder` helpers look a lot like routes.  These helpers perform the actions that their names indicate.

####The _template_ helper

Not surprisingly the _template_ helper creates files based upon a template.  Remember, that the scope object is accessible to the templates.

```javascript
...
'./:filename': { template: 'example.template.js' },
...
```

The left-hand side specifies the path and filename where as the right dictates which template the _generator_ will use to create the file.  Notice we're using the `filename` from the `scope.filename` assignment that was based upon the the first element of `scope.args` in the `before()` function.  The templates can be found in `myProject\awesome\templates`.  In the awesome _generator_ we're using `example.template.js`:

```javascript
/**
 * This is just <%= whatIsThis %>.
 *
 * You can use underscore templates, see?
 */

module.exports = function () {
  // ...
};
```

>**Note:** the scope property `whatIsThis` which as you may recall uses the createdAt: property created in the `before` function.

####The _folder_ helper

The _folder_ helper generates folders.  

```javascript
...
'./hey_look_a_folder': { folder: {} }
...
```
The left-hand side specifies the path and name of the folder.  The right-hand side specifies any optional parameters. For example, by default, if a folder already exists at that location an error will be displayed:
`Something else already exists at ::<path of folder>`.  If you want the _generator_ to overwrite an existing folder you have two options.  You can alter the _folder_ helper to overwrite the existing folder by specifying `force: true` in the options parameters:

```javascript
...
'./hey_look_a_folder': { folder: { force: true} }
...
```


You can also use the `--force` parameter from the command-line when executing the _generator_ which will configure all helpers to overwrite:

```sh
~/ $ sails generate awesome test --force
```

##Using a generator within a generator
To leverage the work of other programmers, _generators_ were designed to be used by other _generators_.  This is where the scope object being passed down to all _generators_ becomes really powerful. 

For example, Sails core has a _generator_ called `sails-generate-model`.  Since it's built into Sails core, there's no installation necessary.  To add it to our awesome _generator_ example is simple.  Within the `myProject\awesome\Generator.js` include it by inserting `./': ['model'],`

```javascript
...
targets: {

    // './:filename': { template: 'example.template.js' },

    './': ['model'],

    // './hey_look_a_folder': { folder: {} }

  },
...
```
>**Note:** By using `./` as the path, any models will be placed in the `\api\models` folder from whatever folder the generator was executed.

That's it!  Now let's create a model from within the awesome _generator_.  From the terminal type:

```sh
~/ $ sails generate awesome user name:string email:email
```

If you take a look in `myProject\api\models` you'll see a new file named `User.js` has been created that contains the model attributes specified earlier.

```javascript
/**
* User
*
* @description :: TODO: You might write a short summary of how this model works and what it represents here.
* @docs        :: http://sailsjs.org/#!documentation/models
*/

module.exports = {

  attributes: {

    name : { type: 'string' },

    email : { type: 'email' }
  }
};
```

##Bonus: Publishing your generator to npmjs.org

To publish the awesome _generator_ to npmjs.org go into the `myProject\awesome\package.json` file and change the name, author and any other meta information (e.g. licensing).

From within the `myProject\awesome` folder at the terminal type:
```sh
~/ $ npm publish
```
>**Note:** I'm assuming you already have an account on npm.  If not, go to (npmjs.org)[https://www.npmjs.org/] and create one.

To unpublish the module, type: 

```sh
~/ $  npm unpublish` --force
```
Change the `myProject\.sailsrc` to:

```javascript
{
  "generators": {
    "modules": {
      "awesome": "whatever you named the module in package.json"
    }
  }
}
```

From the awesome _generator_ folder within the terminal type:

```sh
~/ $ npm install
```

And you're all set!

