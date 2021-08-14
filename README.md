# js-understanding--This-
## Conceptual Overview of "this"
When a function is created, a keyword called this is created (behind the scenes), which links to the object in which the function operates.

The this keyword’s value has nothing to do with the function itself, how the function is called determines this's value

Default "this" context
// define a function
var myFunction = function () {
  console.log(this);
};

// call it
myFunction();
