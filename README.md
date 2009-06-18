Jask: Javascript Tasks
======================

Jask is a simple task runner written in Javascript and runs on [v8cgi](http://code.google.com/p/v8cgi/).


Installing Jask
---------------

	git clone git://github.com/keeto/jask jask
	cd jask
	chmod +x jask
	cp jask /usr/bin


Using Jask
----------

When run, Jask looks for a `taskfile` in the current directory. If you want to set the taskfile to read, use `-t=filename` or `--taskfile=filename`:
	
	$ jask [-t=taskfile] namespace:task arguments
	
To get a list of all available task, use `-l` or `--list`.

	$ jask -l


Taskfiles
---------

Taskfiles are simple Javascript files that contain a single `export` declaration:

	export.myTasts = {
		
		task1: function(name){
			console.log("Hello, " + name + '!\n');
		},
		
		task2: function(name, address){
			this.tast1(name);
			console.log('You live in ' + address + '.\n');
		}
		
	};

The taskfile above will create a new namespace named `myTask` with two tasks, `task1` and `task2`. It is absolutely necessary to add the export declaration together with the namespace, or Jask will not run.

Each task is a simple javascript function. Because task namespaces are objects, you can use `this` to refer to the current namespace. You also have access to the global namespace (via `global`). And because taskfiles are plain v8cgi javascript files, you can include any modules you'll need on your taskfile using the `include` and `require` declarations (see the [v8cgi docs](http://code.google.com/p/v8cgi/wiki/API) for more details).

To run a task, simply pass the namespace and task name to Jask, together with your arguments, separated by spaces:

	$ jask myTasks:task2 Mark Manila
	  Hello, Mark!
	  You live in Manila.


Including Other Taskfiles
-------------------------

To simplify the process of including other taskfiles, you can include an `Include` property on your taskfile, which is an array of filenames for other taskfiles:

	export.myTasts = {
		
		Include: ['myOtherTasks', 'moretasks/tasks.js'],
	
		task1: function(){
			console.log("Yay!");
		}
	
	};

Take note that Jask will look for files relative to the current directory where it was ran.


Task Dependencies
-----------------

You can specify simple dependencies using your task names:

	export.myTasts = {
	
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

	export.myTasts = {
		
		Desc: {
			getWater: ['Gets some water'],
			drink: ['Drinks the water', 'name']
		},
	
		getWater: function(){
			console.log("Got water!\n");
		},
	
		'drink < getWater': function(name){
			console.log("Drink " + name + "\n");
		}
	
	};


Useful Links
------------
[Keetology](http://keetology.com) - The author's site.
[v8cgi](http://code.google.com/p/v8cgi/) - Official v8cgi repo.


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