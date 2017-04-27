#Knockout Components

###1. Intro
In this chapter I'll try to show the concept of Knockout Components. The thing that brings reusable modules responsible for its' own state and look. Main idea reminds me all the modern react'ich style of components and besides some details thy are pretty similar.

###2. Why
We've used the Knockout as the forontend framework of choice because of its' "unobtrusive" nature. As well as for React it's just a JS library but it in comparison, it can be used without redesigning the whole application's view structure, building process and deployment. 

###3. How
It starts with the bindings. Lets say that we have a really basic list of the comments to show.

Some html:
`<div class="comments" data-bind="foreach: comments">
	<div class="single-comment">
		<img data-bind="attr: {src: avatarPath}" />
		<span data-bind="text: commentText"></span>
	</div>
</div>

<script>
	(function() {
		ko.applyBindings({
			comments: [
				{commentText: "first comment",
				 avatarPath: "path/to/avatar1.png" },
				{commentText: "second comment",
				 avatarPath: "path/to/avatar2.png" },
				{commentText: "third comment",
				 avatarPath: "path/to/avatar3.png" }]
			});
	}();
</script>`

As you can see, comments has list of similar objects which are of the same "type". Lets create the "class" for them (ES5).

`var CommentViewModel = function(text, avatarPath) {
	this.commentText = text;
	this.avatarPath = avatarPath;
}`

And for the comments container

`var CommentsViewModel = function(loadingParameter) {
	this.comments = loadAllTheCommentsUsingParamenter(loadingParameter);
	
	function loadAllTheCommentsUsingParamenter(parameter) {	
		//TODO: Add some fancy creation logic
		
		return [
			new CommentViewModel("first comment", "path/to/avatar1.png"),
			new CommentViewModel("second comment", "path/to/avatar2.png"),
			new CommentViewModel("third comment", "path/to/avatar3.png")
		];
	}
}`

So now it can be used as

`var loadingParameter = "parameter1";
var comments = new CommentsViewModel(loadingParameter);
ko.applyBindings(comments);`

Now lets say that we need to display that list in many different places within the system. This is what the templates should address nicely.

So let's move the comments and comments list into templates

`<script type="text/html" id="comment-template">
   	<div class="single-comment">
		<img data-bind="attr: {src: avatarPath}" />
		<span data-bind="text: commentText"></span>
	</div>
</script>

<script type="text/html" id="comments-template">
    <div class="comments" data-bind="template: {name: 'comment-template', foreach: comments} "></div>
</script>`

Which can be use now as:
`<div data-bind="template: {name: 'comments-template'} "></div>`

Having the pairs of View Models and Templates gives us the ability to pack them into Components. Well... of course we could go to the components right away with the inline html template and view model but that's look for me like a more real life process. 

Creating one looks like this:

`ko.components.register('comments-list', { viewModel: CommentsViewModel, template: { element: 'comments-template' } });`

After registration it may be used like this

`<div data-bind="component: 'comments-list'"></div>`

Or even like this

`<comments-list params="loadingParameter: 'test1'"></comments-list>`
