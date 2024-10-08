# ode-plotter
Simple plotting utility for nonlinear dynamics attractors and other ODEs using the Plotly graphing library.

Try it out [here](https://ed-puckett.github.io/ode-plotter/ode-plotter.html).

[Repository on github](https://github.com/ed-puckett/ode-plotter)

# License
This software is licensed under the MIT License.  See [here](LICENSE.txt).

# Usage
Load the file ode-plotter.html in a modern Web browser such as Firefox or Chromium.

Define and return an "equation definition" in the input area at the top of the page, and then click "Go".

You can rotate the graph while running by dragging it, but for full control you must Pause or Stop the process.

# Saving your work {#saving-your-work}
If you save the page on your computer as "Web page, complete", then the changes you make in the input area will be saved to that local copy.  Additionally, the Plotly library will be cached locally, and that permits offline use and makes startup faster.

Note: this has been tested on Chromium and Firefox on Ubuntu 20.04.  I hope that it will work equally well on other platforms and/or other "modern" browsers.

# Defining Equations

Equations are defined in the input area, by returning an equation definition object.

## Equation Definition Object Properties

| Name        | Required? | Type                    | Description |
| :---------- | :-------: | :---------------------- | :---------- |
| title       |           | String                  | title for the plot |
| f           | x         | Function                | function returning ẋ; (x: number[], params: any) => number[] |
| iter        |           | Iterable&#124;Generator | iterable, array or generator function returning Config objects for each iteration |
| iter_delay  |           | number[]                | delay between iterations (milliseconds) |

... or any field from a Config object

## Config Object Properties

| Name        | Required? | Type                 | Description |
| :---------- | :-------: | :------------------- | :---------- |
| integrator  |           | Function             | (see below for an example); (dt: number, f: Function, x: number[], params: any) => number[] |
| dt          |           | number[]             | time step |
| x0          |           | number[]             | initial value |
| params      |           | any                  | parameters that will be passed to f |
| skip        |           | non-negative integer | number of time steps to skip before beginning plot (default: 0) |
| steps       |           | non-negative integer | number of time steps to plot after skipped time steps (default: Infinity) |
| point_cloud |           | PointCloud           | definition of point cloud to be plotted after main equation is plotted |

## PointCloud Object Properties

| Name        | Required? | Type                 | Description |
| :---------- | :-------: | :------------------- | :---------- |
| n           | x         | positive integer     | number of point cloud points |
| c           | x         | number[]             | center of point cloud |
| r           | x         | number               | maximum radius around center for points |
| dt          |           | number               | time step (default: time step from current config) |
| steps       |           | non-negative integer | number of time steps to run point cloud (default: Infinity) |

## Operation

Overall, one or more plots is displayed.  Optionally, each plot is followed by a point cloud which is a set of points that start out from nearly the same point and evolve according to the function, giving a sense of the dynamics.

Each of the displayed plots is controlled by a Config.  If there is no _iter_ property defined in the Equation Definition, then the Config is the Equation Definition itself and only one plot is displayed.  If there is an _iter_ property defined, then each Config returned defines one plot, and the Equation Definition and Config are combined to form the merged Config used for that plot.  In either case, the _integrator_, _dt_ and _x0_ properties must be defined in the (merged) Config.

## Examples

The following examples reference an integrator.  An example is:

```javascript
function runge_kutta_integrator(dt, f, x, params) {
    // This is an implementation of the fourth-order Runge-Kutta method
    // as presented in Nonlinear Dynamics and Chaos, Second Edition, by
    // Steven H. Strogatz, page 34.
    // This implementation works for x with arbitrary dimension.
    const k1 = f( x,                                params ).map( xidot => xidot*dt );
    const k2 = f( x.map( (xi, i) => xi + k1[i]/2 ), params ).map( xidot => xidot*dt );
    const k3 = f( x.map( (xi, i) => xi + k2[i]/2 ), params ).map( xidot => xidot*dt );
    const k4 = f( x.map( (xi, i) => xi + k3[i]),    params ).map( xidot => xidot*dt );
    return x.map( (xi, i) => xi + (k1[i] + 2*k2[i] + 2*k3[i] + k4[i])/6 );
}
const integrator = runge_kutta_integrator;
```

The following subsections give some example definitions.

### Lorenz attractor

```javascript
return {
    title: 'Lorenz Attractor',
    integrator,
    dt: 0.005,
    x0: [3, 3, 1],
    params: { sigma: 10, r: 28, b: 8/3 },
    f:  ([x, y, z], {sigma, r, b}) => [
            sigma*(y - x),
            r*x - y - x*z,
            x*y - b*z,
        ]
};
```

### Rössler attractor iterating its _a_ parameter over several plots

```javascript
return {
    integrator,
    dt: 0.02,
    x0: [10, 10, 1],
    f:  ([x, y, z], {a, b, c}) => [
            -y - z,
            x + a*y,
            b + z*(x - c),
        ],
    params: { a: 0.2, b: 0.2, c: 5.7 },
    skip:  1000,
    steps: 5000,
    iter: function* (def) {
        for (let a = 0.1; a < 0.2; a += 0.001) {
            yield { params: { ...def.params, a } }
        }
    },
    iter_delay: 1000,
};
```

### Lorenz attractor with point cloud

```javascript
return {
    title: 'Lorenz Attractor With Point Cloud',
    integrator,
    dt: 0.005,
    x0: [3, 3, 1],
    params: { sigma: 10, r: 28, b: 8/3 },
    f:  ([x, y, z], {sigma, r, b}) => [
             sigma*(y - x),
             r*x - y - x*z,
             x*y - b*z,
        ],
    skip:  1000,
    steps: 10000,
    point_cloud: {
        n: 6000,
        c: [1, 1, 1],
        r: 0.05,
        dt: 0.05,
    }
};
```

# Optimizing startup
If you experience startup delays, you can download the Plotly code instead of loading it from the CDN each time.  Download the code from https://cdn.plot.ly/plotly-2.8.3.min.js and then change the &lt;script&gt; element src attribute to point to your donwloaded copy.

An alternative is to save the page locally.  See [Saving your work](#saving-your-work) above.
