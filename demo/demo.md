## Configuration

We need a configurable function, since most charts require customization of their appearance or behavior. For example, you may need to specify the width and height, or the color palette. A simple method of configuration is passing arguments:

    function chart(width, height) {
    // generate chart here, using `width` and `height`
    }

Yet this is cumbersome for the caller: they must store the chart’s various arguments, and pass them whenever an update is needed. A simple function is insufficient for highly-configurable charts. You could try a configuration object instead, as is done by many charting libraries:

    function chart(config) {
    // generate chart here, using `config.width` and `config.height`
    }

However, the caller must then manage both the chart function (assuming you have multiple types of charts to pick from) and the configuration object. To bind the chart configuration to the chart function, we need a closure:
A conventional object-oriented approach as Chart.​proto­type.​render would also work, but then you must manage the this context when calling the function.

    function chart(config) {
    return function() {
        // generate chart here, using `config.width` and `config.height`
    };
    }

Now, the caller need merely say:

    var myChart = chart({width: 720, height: 80});

And subsequently, myChart() to update. Simple!

## Reconfiguration

But what if you want to change the configuration after construction? Or if you want to inspect the configuration of an existing chart? The configuration object is trapped by the closure and inaccessible to the outside world. Fortunately, JavaScript functions are objects, so we can store configuration properties on the function itself!

    var myChart = chart();
    myChart.width = 720;
    myChart.height = 80;

The chart implementation changes slightly so that it can reference its configuration:
The inner function (my) can be named whatever you like; the name is only visible internally. You can even use the name chart, which would mask the outer function!

    function chart() {
    return function my() {
        // generate chart here, using `my.width` and `my.height`
    };
    }

With a little bit of syntactic sugar, we can replace raw properties with getter-setter methods that allow method chaining. This gives the caller a more elegant way of constructing charts, and also allows the chart to manage side-effects when a configuration parameter changes. The chart may also provide default configuration values. Here we create a new chart and set two properties:

    var myChart = chart().width(720).height(80);

Modifying an existing chart is similarly easy:

    myChart.height(500);

As is inspecting it:

    myChart.height(); // 500

Internally, the chart implementation becomes slightly more complex to support getter-setter methods, but convenience for the user merits additional developer effort! (And besides, this pattern becomes natural after you’ve used it for a while.)

    function chart() {
    var width = 720, // default width
        height = 80; // default height

    function my() {
        // generate chart here, using `width` and `height`
    }

    my.width = function(value) {
        if (!arguments.length) return width;
        width = value;
        return my;
    };

    my.height = function(value) {
        if (!arguments.length) return height;
        height = value;
        return my;
    };

    return my;
    }

To sum up: implement charts as closures with getter-setter methods. Conveniently, this is the same pattern used by D3’s other reusable objects, including scales, layouts, shapes, axes, etc.

## Implementation

The chart can now be configured, but two essential ingredients are still missing: the DOM element into which to render the chart (such as a particular div or document.body), and the data to display. These could be considered configuration, but D3 provides a more natural representation for data and elements: the selection.

By taking a selection as input, charts have greater flexibility. For example, you can render a chart into multiple elements simultaneously, or easily move a chart between elements without explicitly unbinding and rebinding. You can control exactly when and how the chart gets updated when data or configuration changes (for example, using a transition rather than an instantaneous update). In effect, the chart becomes a rubber stamp for rendering data visually.

The simplest way of invoking our chart function on a selection, then, is to pass the selection as an argument:

    myChart(selection);

From the API reference: "[call] invokes the specified function once, passing in the current selection… The call operator is identical to invoking a function by hand; but it makes it easier to use method chaining."

Or equivalently, using selection.call:

    selection.call(myChart);

Internally, a call-based chart implementation looks something like this:
You could also design your chart function to work directly with selection.each, but selection.call is more general and has precedent with the brush and axis components.

    function my(selection) {
    selection.each(function(d, i) {
        // generate chart here; `d` is the data and `this` is the element
    });
    }


## Examples

To make this proposal concrete, consider a simple yet ubiquitous use case: time-series visualization. A time series is a variable that changes over time.

## Further Considerations

