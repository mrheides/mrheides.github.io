# Browserify

### Intro

This is about the tool that helped us to organize our js knockout view models (and ultimately all js files) into require-able js modules bundled together as one file.

### Why

Looking at the number of js files, number of code lines and desire to have reusability and dependencies managed easily (somewhat...)

### How

First, we need to specify the js module in separate js file. Lets say we would like to tweak our CommentsComponents.

The simplest Comment view model:
```
var CommentViewModel = function (params) {
    this.commentText = params.text;
    this.avatarPath = params.avatarPath;
}

module.exports = CommentViewModel;
```

And Comments view module:
```
var CommentViewModel = require("./CommentViewModel.js");

var CommentsViewModel = function(params) {
    var self = this;
    this.comments = ko.observableArray(loadAllTheCommentsUsingParamenter(params.loadingParameter) || []);
    this.addNewElement = function(textValue) {
        self.comments.push(new CommentViewModel({ text: textValue, avatarPath: "path/to/new.png" }));
    }

    function loadAllTheCommentsUsingParamenter(parameter) {
        //TODO: Add some fancy creation logic
        return [
            new CommentViewModel({text: "first comment", avatarPath: "path/to/avatar1.png"}),
            new CommentViewModel({text: "second comment", avatarPath: "path/to/avatar2.png"}),
            new CommentViewModel({text: "third comment", avatarPath: "path/to/avatar3.png"}),
        ];
    }
}

module.exports = CommentsViewModel;
```

View models are one part of ko.components, another one are templates. Lets move our html templates into separate files but remove "Script" tag in it.
So in this moment we ended up with list of Component pairs like: "CoomentsViewModel.js" and "CommentsTemplate.html"

One last thing before we bundle everything up is knocokut components registration. We will try to implement custom component registration which will do everything automagically for us later but now we need to register them manually.
```
(function() {
    ko.components.register('comment-input', { viewModel: require("./CommentInputViewModel.js"), template: require("./CommentInputTemplate.html") });
    ko.components.register('comment', { viewModel: require("./CommentViewModel.js"), template: require("./CommentTemplate.html") });
    ko.components.register('comments-list', { viewModel: require("./CommentsViewModel.js"), template: require("./CommentsTemplate.html") });
})();
```

Now, having such files will show us nothing more than lots of red lines thrown in browser's console. Lets create special Gulp task which will transform our nice to read/code modules to browser friendly js content.

First things first, we need to get Gulp (nuget - `install package gulp`, npm - `npm install gulp`, or whatever...)

Then get the other dependencies...

`npm install --save-dev browserify stringify vinyl-source-stream`

```
var gulp = require('gulp');
var browserify = require('browserify');
var stringify = require('stringify');
var streamHelper = require('vinyl-source-stream');

gulp.task('bundle', function () {
    return browserify({
        entries: ['./Components/ComponentRegistration.js'],
        debug: true,
        transform: stringify({
            extensions: ['.html']
        })
    })
     .bundle()
     .pipe(streamHelper('./Bundle.js'))
     .pipe(gulp.dest('./Content'));
})
```

Ok, a few words of explanation here:

+ "entries" - array of "entry point" files, files that have modules required but nothing requires them (they can by script src'ed from html view directly)
+ "debug" - information whether source maps should be included into bundle
+ "transform" - list of transformations that will be applied when bundling (stringify says that 'whenever you find html file, just include its content into js as regular string')
+ "pipe(streamHelper('./Bundle.js'))" - streams output into file "Bundle.js"
+ "pipe(gulp.dest('./Scripts'))" - points the output folder where that "Bundle.js" goes

Lets run that task - "gulp bundle". This will produce the "Bundle.js" file which next needs to be included in html view. We can use simple 

`<script src="~/Content/Bundle.js"></script> `

And that's it! We have pretty small, isolated, reusable modules with straight forward dependencies management.


