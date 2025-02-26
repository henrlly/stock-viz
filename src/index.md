# Stocks

```js
const price = view(Inputs.number({type: "number", label: "Current Stock Price"}));
const step = view(Inputs.range([0.1, 20], {label: "Price Variation Step Size", step: 0.1, value: 1}));
const steps = view(Inputs.range([1, 50], {label: "No. of Price Variation Steps", step: 1, value: 5}));

const exit_plan = view(Inputs.number({type: "number", label: "Exit Plan"}));
const stop_loss = view(Inputs.number({type: "number", label: "Stop Loss"}));
```

<!-- need to use a seperate js block as reactive udates only work between blocks -->
```js
const small_step = 0.1;
const small_steps = Math.floor(step * steps / small_step);
const data = Array.from({ length: 2 * small_steps + 1}, (_, i) => {
  const target_price = price + (i-small_steps) * small_step;
  return {
  current_price: price, target_price: target_price, ep_price: exit_plan - target_price, sl_price: stop_loss - target_price
}});
const table_data = Array.from({ length: 2 * steps + 1}, (_, i) => {
  const target_price = price + (i-steps) * step;
  return {
  "Current Price": price, "Target Price": target_price, "Exit Plan": exit_plan - target_price, "Stop Loss": stop_loss - target_price
}});

const x = "Target Price";
const y = "P/L";
view(Plot.plot({
  marks: [
    Plot.line(data.map((d) => ({[x]: d.target_price, [y]: d.ep_price})),{x: x, y: y, stroke: "green", tip: "x"}),
    Plot.tip(data.map((d) => ({[x]: d.target_price, [y]: d.ep_price})),{x: price, y: exit_plan-price}),
    Plot.line(data.map((d) => ({[x]: d.target_price, [y]: d.sl_price})),{x: x, y: y, stroke: "red", tip: "x"}),
    Plot.tip(data.map((d) => ({[x]: d.target_price, [y]: d.sl_price})),{x: price, y: stop_loss-price}),
    Plot.ruleX([price]),
  ]
}));

view(Inputs.table(table_data));
```

