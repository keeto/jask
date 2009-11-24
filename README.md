Jask: Javascript Tasks
======================

Jask is a simple task runner written in Javascript, and can be used with [v8cgi](http://code.google.com/p/v8cgi/) or [node.js](http://nodejs.org/). Somewhat inspired by Ruby's Rake.


Installing Jask
---------------

In order to run Jask, you'll need:

- A working [v8cgi](http://code.google.com/p/v8cgi/) or [nodejs](http://nodejs.org/) installation.
- The v8cgi [GetOpt Module](http://code.google.com/p/v8cgi/source/browse/trunk/lib/getopt.js) (for command line parsing). This is included in the v8cgi package. If you're using node.js, you'll have to download a copy of the module and add it to your modules directory (usually `/usr/local/lib/node/libraries/`).

To install Jask:

	$ git clone git://github.com/keeto/jask.git jask
	$ cd jask
	$ node jask		# node.js
	$ v8cgi jask	# v8cgi
	
If you want to use Jask without specifying the engine, you can edit the shebang line on the Jask source:

	#! /usr/bin/env v8cgi
	#! /usr/bin/env node

Then copy it to somewhere accessible:

	$ chmod +x jask
	$ cp jask ~/sbin
	$ jask
	JASK: Javascript Tasks

	Usage:
	  $ jask [-t=taskfile] [namespace:]task args
	  $ jask -l

	Options:
	  -l, --list					List available tasks
	  -t=file, --taskfile=file		Set taskfile.
	  -q, --quiet					Don't show any log messages.
	  -v, --verbose					Show verbose logs.
	  -h, --help					Show this help text.


Using Jask
----------

When run, Jask looks for a `taskfile.js` file in the current directory. If you want to set the taskfile to read, use `-t=filename` or `--taskfile=filename` (omit the '.js' when specifying the taskfile). To run a specific task, you put in the task's namespace, then the taskname then arguments, if any:
	
	$ jask [-t=taskfile] namespace:task arguments

Optionally, if you only have one namespace in your task file, you can omit the namespace and just pass the taskname:
	
	$ jask [-t=taskfile] task arguments
	
To get a list of all available task, use `-l` or `--list`.

	$ jask -l


Taskfiles
---------

Taskfiles are simple Javascript files (with the '.js' extension) that contain an `export` declaration:

	exports.myTasks = {
		
		task1: function(name){
			console.log("Hello, " + name + '!\n');
		},
		
		task2: function(name, address){
			this.tast1(name);
			console.log('You live in ' + address + '.\n');
			this._hidden();
		},
		
		_hidden: function(){
			console.log('I\'m from a hidden function!\n');
		}
		
	};

The taskfile above will create a new namespace named `myTask` with two tasks, `task1` and `task2`. The function `_hidden`, as well as any other function that starts with an underscore, is a private function. It is absolutely necessary to add the export declaration together with the namespace, or Jask will not run.

Each task is a simple javascript function. Because task namespaces are objects, you can use `this` to refer to the current namespace. You also have access to the global namespace (via `Engine.Global`). And because taskfiles are plain javascript files, you can include any modules you'll need on your taskfile using the `include` and `require` declarations (see the [v8cgi](http://code.google.com/p/v8cgi/wiki/API) or [nodejs](http://nodejs.org/api.html) docs for more details).

To run a task, simply pass the namespace and task name to Jask, together with your arguments, separated by spaces:

	$ jask myTasks:task2 Mark Manila
	  Hello, Mark!
	  You live in Manila.


Including Other Taskfiles
-------------------------

To simplify the process of including other taskfiles, you can include an `Include` property on your taskfile, which is an array of filenames for other taskfiles:

	exports.myTasks = {
		
		Include: ['myOtherTasks', 'moretasks/tasks.js'],
	
		task1: function(){
			console.log("Yay!");
		}
	
	};

Take note that Jask will look for files relative to the current directory where it was ran.


Task Dependencies
-----------------

You can specify simple dependencies using your task names:

	exports.myTasks = {
	
		getWater: function(){
			console.log("Got water!\n");
		},
	
		'drink < getWater': function(){
			console.log("Drank!\n")
		}
	
	};

With this dependency declaration, `getWater` will be called first before `drink`. The name of the task should will always be on the left, and the name of the dependency on the right. You can specify whether to run the task dependancy before or after the task.

	'drink < getWater' // getWater -> drink.
	'drink > getWater' // drink -> getWater (?).

Be sure you place spaces before `<` and `>`, otherwise, your task will not be parsed. You can also specify the namespace of the task:

	'drink < wells:getWater'

The arguments passed on your main task are also passed to your dependant task.


Documenting Your Task
---------------------

You can document your tasks by adding a `Desc` property on your taskfile:

	exports.myTasks = {
		
		Desc: {
			getWater: ['Gets some water'],
			drink: ['Drinks the water', 'name, unusedArg']
		},
	
		getWater: function(){
			console.log("Got water!\n");
		},
	
		'drink < getWater': function(name, unusedArg){
			console.log("Drink " + name + "\n");
		},
		
		test: function(name){
			console.log("No description!")
		}
	
	};

These information will be displayed when the user runs `jask -l`. Public tasks with no description are also displayed, but they won't have any nice documentation:
	
	Available Tasks:
	
		myTasks:getWater		Gets some water
		myTask:drink [name]		Drinks the water
		myTask:test


The Console Object
------------------

Jask includes a built-in `console` object that can be used to log and time your tasks. All loggers output to `os.stdout`:

	console.log(1, 2, 3);
	/* 
	outputs:
	 	1 2 3
	*/
	
	console.dir({a: 1}, {b:'string'});
	/* 
	outputs:
	 	{"a": 1}
		{"b": "string"}
	*/
	
	console.time('id');
	console.timeEnd('id');
	/* 
	outputs:
	 	id: 0ms.
	*/

The Engine Object
-----------------

In order for Jask to run on both v8cgi and nodejs, Jask detects the engine using simple feature detection. There's a global `Engine` object accessible to all tasks files.
	
- `Engine.name` - could either be `nodejs` or `v8cgi`
- `Engine.nodejs` - true if the current engine is `nodejs`
- `Engine.v8cgi` - true if the current engine is `v8cgi`


Useful Links
------------
- [Keetology](http://keetology.com) - The author's site.
- [v8cgi](http://code.google.com/p/v8cgi/) - Official v8cgi repo.
- [node.js](http://nodejs.org/) - Official node.js site.


Copyright and License
---------------------

Copyright (c) 2009 Mark Obcena <http://keetology.com>

Permission is hereby granted, free of charge, to any person
obtaining a copy of this software and associated documentation
files (the "Software"), to deal in the Software without
restriction, including without limitation the rights to use,
copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the
Software is furnished to do so, subject to the following
conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.