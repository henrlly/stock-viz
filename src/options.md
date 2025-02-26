# Options

```js
const spot_price = view(Inputs.number({type: "number", label: "Current Spot Price"}));
const price_var = view(Inputs.range([0.1, 20], {label: "Price Variation Step Size", step: 0.1, value: 1}));
const price_var_steps = view(Inputs.range([0, 20], {label: "Price Variation Steps", step: 1, value: 1}));
const r = view(Inputs.number({type: "number", label: "Risk-free Rate"}));
const k = view(Inputs.number({type: "number", label: "Strike Price"}));
const vol = view(Inputs.number({type: "number", label: "Volatility of Asset"}));
const steps = view(Inputs.range([10, 50], {label: "Max delivery date in years", step: 5, value: 10}));
```

```js
function ncdf(x) {
  const mean = 0
  const std = 1
  x = (x - mean) / std
  const t = 1 / (1 + .2315419 * Math.abs(x))
  const d =.3989423 * Math.exp( -x * x / 2)
  const prob = d * t * (.3193815 + t * ( -.3565638 + t * (1.781478 + t * (-1.821256 + t * 1.330274))))
  return x > 0 ? 1 - prob : prob
}

```

<!-- need to use a seperate js block as reactive udates only work between blocks -->
```js
const step_size = 0.1;
const data_ls = Array.from({length: price_var_steps*2 + 1}, (_, j) => {
  const sp = spot_price + (j - price_var_steps) * price_var;
  return { sp: sp, arr: Array.from({ length: steps * Math.floor(1/step_size) + 1}, (_, i) => { 
    const t = (i * step_size);
    const d1 = (Math.log(sp/k) + (r + ((vol*vol)/2))*t) / (vol * Math.sqrt(t));
    const d2 = d1 - vol * Math.sqrt(t);
    const c = ncdf(d1) * sp - ncdf(d2) * k * Math.exp(-r * t);
    return {call_option_price: c, timestep: t} 
})}});

const x = "Time After Purchase/year";
const y = "Call Option Price";
view(Plot.plot({
  marks: data_ls.map((row) => {
    const data = row.arr;
    return [Plot.line(data.map((d) => ({[x]: d.timestep, [y]: d.call_option_price})), {x: x, y: y, stroke: "green"}),
    Plot.tip(data.map((d) => ({[x]: d.timestep, [y]: d.call_option_price})), Plot.pointer({x: x, y: y, maxRadius: 10, title: (d) => `${x}: ${d[x]}\n${y}: ${d[y]}\nSpot Price: ${row.sp}`})),] }
  )
}));
```

