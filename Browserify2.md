# Browserify (part 2 - integration)

### Intro
It's really nice to have everything composed just as you've imagined but as usual in life, you have to communicate...

### Why
Sometimes (always?) during project's lifetime, there's no possibility to just switch over to something new. 
You've probably been there and have seen similar ways to follow as we did:
+ Lets do it! there's only like... ok, it looks like there's a gazillion of files that needs to be adjusted, but maybe if we could... 
+ Ok, it's not gonna happen in this project, maybe in the next one?
+ Maybe we could try with some of the new functionalities that are somehow separated from others and then try to refactor them one by one?

We have made the decision to take the third option.

### How
Lets say that before we've started our knockout components and browserify adventure we had the "module" somewhere in the system that looks pretty similar to what we've created in the previous part( [Browserify (part 1 - introduction)](./Browserify.md) ).

```
<div class="jumbotron" id="test2">
    <h1>Out of bundle usage</h1>
    <div class="comment-input">
        <label for="commentInputElement">add new comment:</label>
        <input id="commentInputElement" type="text" data-bind="value: commentInputValue"/>
        <button data-bind="click: onClick">add</button>
    </div>
    <!-- ko foreach: comments -->
    <div class="single-comment">
        <img data-bind="attr: {src: $data.avatarPath}" />
        <span data-bind="text: $data.commentText"></span>
    </div>
    <!-- /ko -->
</div>

```

Well... they are actually the same but this one, doesn't have any knockout components defined.

In a separate js, we had function that applies binding like this

``` 
var SomeOutOfBundle = (function() {
    var _comments = ko.observableArray([
        {
            avatarPath: "test2",
            commentText: "comment2"
        }
    ]);

    var _commentInputValue = ko.observable();

    function init() {
        ko.applyBindings({
            commentInputValue: _commentInputValue,
            onClick: function() {
                _comments.push({ avatarPath: "newAvatar", commentText: _commentInputValue() });
                _commentInputValue("");
            },
            comments: _comments

        }, document.getElementById("test2"));
    }

    return {
        init: init
    }
})()
```

Ok, even if there's a lot of things that might be improved, we just cannot stand the duplication and we want to use our previously defined knockout view model "clases" here. The problem is, they are bundled into internal scope and are not accessible from the outside of that bundle. At least, browserify is giving us ability to share our modules by using the `standalone` option within gulp task. 

We just have to provide some name for the "namespace" that will be used to access our modules:
`standalone: "ViewModels"`

So now the browserify will append all the `module.exports` from the `entries` that we've defined in gulp task. Unfortunately we are not exporting anything from our "entry point" yet (ComponentRegistration.js). Lets say that we want to use only one from our view models: 

```
module.exports.CommentViewModel = require("./CommentViewModel.js");
```

After that change, we should be able to use it from our "external" js e.g. like this:

```
 var _comments = ko.observableArray([
        new ViewModels.CommentViewModel({
            avatarPath: "test2",
            text: "comment2"
        })
    ]);
```

The only thing that we need to be aware of is the right order when including js files in view. The bundle.js file must be included before SomeOutOfBundle.js file naturally.


