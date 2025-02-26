# Futures

```js
const spot_price = view(Inputs.number({type: "number", label: "Current Spot Price"}));
const price_var = view(Inputs.range([0.1, 20], {label: "Price Variation Step Size", step: 0.1, value: 1}));
const price_var_steps = view(Inputs.range([0, 20], {label: "Price Variation Steps", step: 1, value: 1}));
const risk_rate = view(Inputs.number({type: "number", label: "Risk-free Rate"}));
const steps_m = view(Inputs.range([0.5, 72], {label: "Max delivery date in months", step: 0.5, value: 10}));
```

<!-- need to use a seperate js block as reactive udates only work between blocks -->
```js
const steps = steps_m / 12;
const step_size = 0.1;
const data_ls = Array.from({length: price_var_steps*2 + 1}, (_, j) => {
  const sp = spot_price + (j - price_var_steps) * price_var;
  return { sp: sp, arr: Array.from({ length: steps * Math.floor(1/step_size) + 1}, (_, i) => { 
    return {forward_price: sp * Math.exp(risk_rate * (i * step_size)), timestep: (i * step_size)} 
})}});

const x = "Time After Purchase/months";
const y = "Forward Price";
view(Plot.plot({
  marks: data_ls.map((row) => {
    const data = row.arr;
    return [Plot.line(data.map((d) => ({[x]: d.timestep*12, [y]: d.forward_price})), {x: x, y: y, stroke: "green"}),
    Plot.tip(data.map((d) => ({[x]: d.timestep*12, [y]: d.forward_price})), Plot.pointer({x: x, y: y, maxRadius: 10, title: (d) => `${x}: ${d[x]}\n${y}: ${d[y]}\nSpot Price: ${row.sp}`})),] }
  )
}));
```

