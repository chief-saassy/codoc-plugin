# Chart examples

Tested patterns for common charts in business and technical documents. Each one draws in Codoc as written: copy the
closest, then change the data and the field names. A test draws every fence here and checks the picture shows what its
section says (`apps/core/test/chart-skill-examples.test.ts`).

Every example follows Codoc's rules: the data is written in the source, nothing is fetched, there is no `title` (give the
chart a short line of prose instead), and colours and fonts are left to Codoc's theme unless they carry meaning.

Contents, in order. Search for the heading to jump to one:

- Report charts
  - Bars with a fitted exponential trend
  - Bars with a fitted linear trend
  - Smoothed line with area and labels
  - Period averages
  - Percentage line on a narrow axis
  - Change by month, coloured by sign
  - Observed against adjusted
- Sorted horizontal bar
- Bar with value labels
- Grouped bar
- Stacked bar with totals
- 100% stacked bar
- Diverging bar
- Line over time
- Several series
- Line with points
- Step line
- Smoothed line
- Area
- Stacked area
- Linear trend line
- Loess trend
- Moving average
- Target or reference line
- Forecast with an interval
- Range band (percentiles)
- Pie with percentage labels
- Donut with labels inside
- Scatter with a trend line
- Bubble chart
- Histogram
- Box plot
- Box plot from a summary
- Heatmap
- Two measures on two axes
- Waterfall (bridge)
- Funnel
- Bullet chart
- Progress against target
- Gantt or timeline
- Small multiples
- Slope chart
- Dumbbell
- Pareto chart
- Running total against a target (burn-up)
- Labels at the ends of lines
- One series highlighted
- Annotated point and event line
- Number formats: currency, thousands and percentages
- Number formats: another currency or locale

## Report charts

Charts from a real monthly growth report, the range agents are most often asked for. They share one way of drawing
months, so copy it whole:

- **Months are `temporal`, one row per month dated the 1st** (`"2026-01-01"`), on a `utc` scale with
  `"format": "%b %y"` and `"tickCount": "month"`, so each label ("Jan 26") sits under its month.
- **Room at the ends:** `"padding": {"expr": "width / 18"}` on the scale keeps the first and last bars and labels
  inside the plot, at any chart width. `"nice": false` stops that padding from pulling in an extra month's tick at
  either end (a "Dec 25" before the data starts).
- **Why not ordinal months?** An ordinal axis can't carry a fitted curve, which falls between months, or a segment that
  ends partway across. On a temporal axis, bars, trend lines, averages and labels all share one x scale. A trend on a
  month index with its own independent scale repeats the month labels along the top.
- **Why not `timeUnit`?** Bars with a `timeUnit` fill each month's band, and a line on the same field then steps from
  band edge to band edge. Bars on a plain date, with `"size": {"expr": "width / 14"}`, stay centred on their month at
  any chart width.

### Bars with a fitted exponential trend

Monthly totals growing by a steady percentage, such as calls or sign-ups, with the trend named in the legend.

Traps avoided:
- The trend is a `regression` with `"method": "exp"` on the dates themselves, so it shares the bars' x axis.
- The legend entry is the trend line's own `strokeDash`, from a field holding its name. A `strokeDash` encoding
  overrides the mark's own dash, so the dash pattern goes in the encoding's `range`.
- The rate in the name is worked out from the fitted line, not typed in: the ratio of its last to its first value over
  the months between them. Change the data and the legend stays right.
- `~s` gives 5k, 10k. As an axis `format` it labels zero "0k", since an axis picks one prefix for all its ticks;
  `labelExpr` with `format(datum.value, '~s')` formats each tick on its own.
- The labels sit above the bars, drawn last so the trend line passes under them. `".3~s"` writes them as 13.8k,
  short enough that neighbouring labels don't run together over narrow bars.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2026-01-01", "calls": 1515}, {"month": "2026-02-01", "calls": 3622},
      {"month": "2026-03-01", "calls": 5625}, {"month": "2026-04-01", "calls": 5093},
      {"month": "2026-05-01", "calls": 6363}, {"month": "2026-06-01", "calls": 8642},
      {"month": "2026-07-01", "calls": 13831}, {"month": "2026-08-01", "calls": 13927},
      {"month": "2026-09-01", "calls": 21870}
    ]
  },
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null,
      "scale": {"type": "utc", "padding": {"expr": "width / 18"}, "nice": false},
      "axis": {"format": "%b %y", "tickCount": "month"}
    }
  },
  "layer": [
    {
      "mark": {"type": "bar", "size": {"expr": "width / 14"}},
      "encoding": {
        "y": {
          "field": "calls", "type": "quantitative", "title": null,
          "axis": {"labelExpr": "format(datum.value, '~s')"}
        }
      }
    },
    {
      "transform": [
        {"regression": "calls", "on": "month", "method": "exp"},
        {
          "joinaggregate": [
            {"op": "argmin", "field": "month", "as": "first"},
            {"op": "argmax", "field": "month", "as": "last"}
          ]
        },
        {"calculate": "(datum.last.month - datum.first.month) / (30.4375 * 864e5)", "as": "span"},
        {
          "calculate": "pow(datum.last.calls / datum.first.calls, 1 / datum.span) - 1",
          "as": "rate"
        },
        {"calculate": "'Fitted trend ' + format(datum.rate, '+.0%') + '/month'", "as": "trend"}
      ],
      "mark": {"type": "line", "color": "#5b6370"},
      "encoding": {
        "y": {"field": "calls", "type": "quantitative"},
        "strokeDash": {
          "field": "trend", "type": "nominal", "title": null, "scale": {"range": [[6, 4]]},
          "legend": {"labelLimit": 0}
        }
      }
    },
    {
      "mark": {"type": "text", "baseline": "bottom", "dy": -4},
      "encoding": {
        "y": {"field": "calls", "type": "quantitative"},
        "text": {"field": "calls", "type": "quantitative", "format": ".3~s"}
      }
    }
  ]
}
```

### Bars with a fitted linear trend

The same, when the total grows by a steady amount each month rather than a steady percentage.

Traps avoided:
- Leave out `method` (or write `"linear"`) for a straight line. The legend gives the slope per month: the change
  along the fitted line over the months it spans.
- Pick the method that fits: here the exponential trend follows the data, and the straight one misses both ends.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2026-01-01", "calls": 1515}, {"month": "2026-02-01", "calls": 3622},
      {"month": "2026-03-01", "calls": 5625}, {"month": "2026-04-01", "calls": 5093},
      {"month": "2026-05-01", "calls": 6363}, {"month": "2026-06-01", "calls": 8642},
      {"month": "2026-07-01", "calls": 13831}, {"month": "2026-08-01", "calls": 13927},
      {"month": "2026-09-01", "calls": 21870}
    ]
  },
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null,
      "scale": {"type": "utc", "padding": {"expr": "width / 18"}, "nice": false},
      "axis": {"format": "%b %y", "tickCount": "month"}
    }
  },
  "layer": [
    {
      "mark": {"type": "bar", "size": {"expr": "width / 14"}},
      "encoding": {
        "y": {
          "field": "calls", "type": "quantitative", "title": null,
          "axis": {"labelExpr": "format(datum.value, '~s')"}
        }
      }
    },
    {
      "transform": [
        {"regression": "calls", "on": "month"},
        {
          "joinaggregate": [
            {"op": "argmin", "field": "month", "as": "first"},
            {"op": "argmax", "field": "month", "as": "last"}
          ]
        },
        {"calculate": "(datum.last.month - datum.first.month) / (30.4375 * 864e5)", "as": "span"},
        {"calculate": "(datum.last.calls - datum.first.calls) / datum.span", "as": "rate"},
        {"calculate": "'Fitted trend ' + format(datum.rate, '+,.0f') + '/month'", "as": "trend"}
      ],
      "mark": {"type": "line", "color": "#5b6370"},
      "encoding": {
        "y": {"field": "calls", "type": "quantitative"},
        "strokeDash": {
          "field": "trend", "type": "nominal", "title": null, "scale": {"range": [[6, 4]]},
          "legend": {"labelLimit": 0}
        }
      }
    },
    {
      "mark": {"type": "text", "baseline": "bottom", "dy": -4},
      "encoding": {
        "y": {"field": "calls", "type": "quantitative"},
        "text": {"field": "calls", "type": "quantitative", "format": ".3~s"}
      }
    }
  ]
}
```

### Smoothed line with area and labels

One rate over the months, such as calls per 1,000 patients, with every value written on it.

Traps avoided:
- The area and the line both use `"interpolate": "monotone"`, so the fill follows the curve exactly. Monotone passes
  through every point without overshooting.
- The area takes the line's colour and Codoc's faint fill, and the points are Codoc's filled, white-ringed points:
  nothing is coloured by hand.
- The labels sit 8 pixels above each point, clear of the line.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2026-01-01", "rate": 41}, {"month": "2026-02-01", "rate": 98},
      {"month": "2026-03-01", "rate": 87}, {"month": "2026-04-01", "rate": 79},
      {"month": "2026-05-01", "rate": 75}, {"month": "2026-06-01", "rate": 82},
      {"month": "2026-07-01", "rate": 106}, {"month": "2026-08-01", "rate": 84},
      {"month": "2026-09-01", "rate": 123}
    ]
  },
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null,
      "scale": {"type": "utc", "padding": {"expr": "width / 18"}, "nice": false},
      "axis": {"format": "%b %y", "tickCount": "month"}
    },
    "y": {"field": "rate", "type": "quantitative", "title": null}
  },
  "layer": [
    {"mark": {"type": "area", "interpolate": "monotone"}},
    {"mark": {"type": "line", "interpolate": "monotone", "point": true}},
    {
      "mark": {"type": "text", "baseline": "bottom", "dy": -8},
      "encoding": {"text": {"field": "rate", "type": "quantitative"}}
    }
  ]
}
```

### Period averages

Averages over parts of the range, drawn as dashed segments, such as before and after a change.

Traps avoided:
- Each segment is a `rule` with `x`, `x2` and `y`: from the period's first month to its last, at its average.
- The averages are worked out from the data (`aggregate` with `mean`, `min` and `max`, grouped by period), so they
  can't disagree with the line.
- Each period is named in the legend by its own dash pattern. Both are grey, the colour for something derived from
  the data rather than measured.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2026-01-01", "rate": 41}, {"month": "2026-02-01", "rate": 98},
      {"month": "2026-03-01", "rate": 87}, {"month": "2026-04-01", "rate": 79},
      {"month": "2026-05-01", "rate": 75}, {"month": "2026-06-01", "rate": 82},
      {"month": "2026-07-01", "rate": 106}, {"month": "2026-08-01", "rate": 84},
      {"month": "2026-09-01", "rate": 123}
    ]
  },
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null,
      "scale": {"type": "utc", "padding": {"expr": "width / 18"}, "nice": false},
      "axis": {"format": "%b %y", "tickCount": "month"}
    }
  },
  "layer": [
    {
      "mark": {"type": "line", "point": true},
      "encoding": {"y": {"field": "rate", "type": "quantitative", "title": null}}
    },
    {
      "transform": [
        {"calculate": "utcmonth(datum.month)", "as": "m"},
        {"calculate": "datum.m < 1 ? '' : datum.m <= 4 ? 'Feb–May' : 'Jun–Sep'", "as": "period"},
        {"filter": "datum.period"},
        {
          "aggregate": [
            {"op": "mean", "field": "rate", "as": "avg"},
            {"op": "min", "field": "month", "as": "from"},
            {"op": "max", "field": "month", "as": "to"}
          ],
          "groupby": ["period"]
        },
        {"calculate": "datum.period + ' average ' + format(datum.avg, '.0f')", "as": "label"}
      ],
      "mark": {"type": "rule", "color": "#5b6370", "strokeWidth": 2},
      "encoding": {
        "x": {"field": "from", "type": "temporal"},
        "x2": {"field": "to"},
        "y": {"field": "avg", "type": "quantitative"},
        "strokeDash": {
          "field": "label", "type": "nominal", "title": null, "scale": {"range": [[6, 4], [2, 3]]},
          "legend": {"labelLimit": 0}
        }
      }
    }
  ]
}
```

### Percentage line on a narrow axis

A share that moves within a band, such as completion between 40% and 65%, where an axis from zero would flatten it.

Traps avoided:
- Store shares as fractions (0.6) and format them with `".0%"`, on the axis and on the labels.
- `"scale": {"domain": [0.3, 0.8]}` narrows the axis. That is honest for a line, whose position carries the value.
  Never do it for bars or areas, whose length does.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2026-01-01", "share": 0.6}, {"month": "2026-02-01", "share": 0.49},
      {"month": "2026-03-01", "share": 0.41}, {"month": "2026-04-01", "share": 0.51},
      {"month": "2026-05-01", "share": 0.57}, {"month": "2026-06-01", "share": 0.61},
      {"month": "2026-07-01", "share": 0.65}, {"month": "2026-08-01", "share": 0.64},
      {"month": "2026-09-01", "share": 0.59}
    ]
  },
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null,
      "scale": {"type": "utc", "padding": {"expr": "width / 18"}, "nice": false},
      "axis": {"format": "%b %y", "tickCount": "month"}
    },
    "y": {
      "field": "share", "type": "quantitative", "title": null, "scale": {"domain": [0.3, 0.8]},
      "axis": {"format": ".0%"}
    }
  },
  "layer": [
    {"mark": {"type": "line", "point": true}},
    {
      "mark": {"type": "text", "baseline": "bottom", "dy": -8},
      "encoding": {"text": {"field": "share", "type": "quantitative", "format": ".0%"}}
    }
  ]
}
```

### Change by month, coloured by sign

Month-on-month change that can be up or down.

Traps avoided:
- Bars below zero are red by a `condition`; the rest keep Codoc's first colour, so only the meaning is coloured.
- Labels go above bars that rise and below bars that fall (`baseline` and `dy` chosen by expression), so a small
  negative bar's label sits clear below it instead of on it.
- `"+.1%"` signs every label: +4.9%, −6.1%. A rule at zero marks the baseline, from one row of its own,
  `{"zero": 0}`. Sharing the chart's data, a rule at `"datum": 0` is drawn once per month; on an empty row `{}`
  it isn't drawn at all.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2026-01-01", "change": 0.016}, {"month": "2026-02-01", "change": -0.003},
      {"month": "2026-03-01", "change": 0.028}, {"month": "2026-04-01", "change": 0.049},
      {"month": "2026-05-01", "change": 0.029}, {"month": "2026-06-01", "change": 0.044},
      {"month": "2026-07-01", "change": -0.061}, {"month": "2026-08-01", "change": -0.042},
      {"month": "2026-09-01", "change": -0.002}
    ]
  },
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null,
      "scale": {"type": "utc", "padding": {"expr": "width / 18"}, "nice": false},
      "axis": {"format": "%b %y", "tickCount": "month"}
    },
    "y": {"field": "change", "type": "quantitative", "title": null, "axis": {"format": ".0%"}}
  },
  "layer": [
    {
      "mark": {"type": "bar", "size": {"expr": "width / 14"}},
      "encoding": {"color": {"condition": {"test": "datum.change < 0", "value": "#e04a4a"}}}
    },
    {
      "mark": {
        "type": "text",
        "baseline": {"expr": "datum.change < 0 ? 'top' : 'bottom'"},
        "dy": {"expr": "datum.change < 0 ? 4 : -4"}
      },
      "encoding": {"text": {"field": "change", "type": "quantitative", "format": "+.1%"}}
    },
    {
      "data": {"values": [{"zero": 0}]},
      "mark": "rule",
      "encoding": {"y": {"field": "zero", "type": "quantitative"}, "x": null}
    }
  ]
}
```

### Observed against adjusted

Two versions of one measure, such as observed and seasonally adjusted, in a single legend.

Traps avoided:
- The data is long: one row per month and series. `color` and `strokeDash` on the same field merge into one legend,
  observed solid and adjusted dashed.
- The area is a layer of its own, for the observed series only, under the lines.
- The legend's symbols are drawn as strokes at full opacity (`symbolType`, `symbolOpacity`). Otherwise they take the
  faint area's look and the dashes don't show.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2026-01-01", "series": "Observed", "rate": 41},
      {"month": "2026-01-01", "series": "Adjusted", "rate": 55},
      {"month": "2026-02-01", "series": "Observed", "rate": 98},
      {"month": "2026-02-01", "series": "Adjusted", "rate": 90},
      {"month": "2026-03-01", "series": "Observed", "rate": 87},
      {"month": "2026-03-01", "series": "Adjusted", "rate": 86},
      {"month": "2026-04-01", "series": "Observed", "rate": 79},
      {"month": "2026-04-01", "series": "Adjusted", "rate": 82},
      {"month": "2026-05-01", "series": "Observed", "rate": 75},
      {"month": "2026-05-01", "series": "Adjusted", "rate": 80},
      {"month": "2026-06-01", "series": "Observed", "rate": 82},
      {"month": "2026-06-01", "series": "Adjusted", "rate": 85},
      {"month": "2026-07-01", "series": "Observed", "rate": 106},
      {"month": "2026-07-01", "series": "Adjusted", "rate": 97},
      {"month": "2026-08-01", "series": "Observed", "rate": 84},
      {"month": "2026-08-01", "series": "Adjusted", "rate": 90},
      {"month": "2026-09-01", "series": "Observed", "rate": 123},
      {"month": "2026-09-01", "series": "Adjusted", "rate": 108}
    ]
  },
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null,
      "scale": {"type": "utc", "padding": {"expr": "width / 18"}, "nice": false},
      "axis": {"format": "%b %y", "tickCount": "month"}
    },
    "y": {"field": "rate", "type": "quantitative", "title": null},
    "color": {
      "field": "series", "type": "nominal", "title": null, "sort": ["Observed", "Adjusted"],
      "legend": {"symbolType": "stroke", "symbolOpacity": 1, "symbolStrokeWidth": 2}
    }
  },
  "layer": [
    {"transform": [{"filter": "datum.series === 'Observed'"}], "mark": "area", "encoding": {}},
    {
      "mark": "line",
      "encoding": {
        "strokeDash": {
          "field": "series", "type": "nominal", "title": null, "sort": ["Observed", "Adjusted"],
          "scale": {"domain": ["Observed", "Adjusted"], "range": [[1, 0], [6, 4]]}
        }
      }
    }
  ]
}
```

## Sorted horizontal bar

Ranking categories by one number, especially when their names are long. The default for "which is biggest".

Traps avoided:
- A nominal axis sorts alphabetically. `"sort": "-x"` on the category ranks the bars by their length, largest at the top.
- Names go on the y axis, so they read horizontally instead of tilting. Codoc cuts a name longer than about 180
  pixels short with an ellipsis; `"labelLimit": 240` gives this one room.
- `"title": null` drops an axis title that would only repeat the category names.

```vega-lite
{
  "data": {
    "values": [
      {"team": "Platform", "tickets": 142}, {"team": "Payments", "tickets": 87},
      {"team": "Mobile apps", "tickets": 121}, {"team": "Data engineering", "tickets": 64},
      {"team": "Customer support tools", "tickets": 38}
    ]
  },
  "mark": "bar",
  "encoding": {
    "y": {
      "field": "team", "type": "nominal", "sort": "-x", "title": null, "axis": {"labelLimit": 240}
    },
    "x": {"field": "tickets", "type": "quantitative", "title": "Open tickets"}
  }
}
```

## Bar with value labels

When the exact numbers matter as much as the comparison, so the reader needn't read them off the axis.

Traps avoided:
- The label is a second layer sharing the bar's `x` and `y`, so each number sits at the end of its own bar.
- `"align": "left"` with `"dx": 4` puts the number just past the end of the bar instead of over it.
- A label past the longest bar sits in the chart's padding, which grows to fit it, so the scale needn't be stretched.

```vega-lite
{
  "data": {
    "values": [
      {"region": "North America", "revenue": 4820}, {"region": "Europe", "revenue": 3910},
      {"region": "Asia Pacific", "revenue": 2470}, {"region": "Latin America", "revenue": 960}
    ]
  },
  "encoding": {
    "y": {"field": "region", "type": "nominal", "sort": "-x", "title": null},
    "x": {"field": "revenue", "type": "quantitative", "title": "Revenue ($k)"}
  },
  "layer": [
    {"mark": "bar"},
    {
      "mark": {"type": "text", "align": "left", "dx": 4},
      "encoding": {"text": {"field": "revenue", "type": "quantitative", "format": "$,.0f"}}
    }
  ]
}
```

## Grouped bar

Comparing a few series side by side within each category, such as this year against last year by quarter.

Traps avoided:
- `xOffset` on the series field puts the bars side by side. Without it they stack.
- The colour (and its legend) is on the same field as `xOffset`, so the reader can tell the bars apart.
- Quarters written `Q1`…`Q4` sort correctly as text. Month names don't: give them a `sort` list or use dates.

```vega-lite
{
  "data": {
    "values": [
      {"quarter": "Q1", "year": "2025", "sales": 310},
      {"quarter": "Q1", "year": "2026", "sales": 365},
      {"quarter": "Q2", "year": "2025", "sales": 340},
      {"quarter": "Q2", "year": "2026", "sales": 402},
      {"quarter": "Q3", "year": "2025", "sales": 295},
      {"quarter": "Q3", "year": "2026", "sales": 388},
      {"quarter": "Q4", "year": "2025", "sales": 420},
      {"quarter": "Q4", "year": "2026", "sales": 451}
    ]
  },
  "mark": "bar",
  "encoding": {
    "x": {"field": "quarter", "type": "nominal", "title": null},
    "xOffset": {"field": "year", "type": "nominal"},
    "y": {"field": "sales", "type": "quantitative", "title": "Sales ($k)"},
    "color": {"field": "year", "type": "nominal", "title": "Year"}
  }
}
```

## Stacked bar with totals

Showing a total and what it is made of, for a few parts. Totals compare easily; of the parts, only the bottom
one's sizes compare well.

Traps avoided:
- The total labels are a separate layer that adds the parts up (`"aggregate": "sum"`) and has no `color`. With `color`
  in the shared encoding, the text layer would make one label per part.
- The months are given a `sort` list, since text sorts alphabetically (Apr, Feb, Jan, Mar).

```vega-lite
{
  "data": {
    "values": [
      {"month": "Jan", "channel": "Direct", "orders": 120},
      {"month": "Jan", "channel": "Partner", "orders": 60},
      {"month": "Jan", "channel": "Online", "orders": 90},
      {"month": "Feb", "channel": "Direct", "orders": 135},
      {"month": "Feb", "channel": "Partner", "orders": 72},
      {"month": "Feb", "channel": "Online", "orders": 110},
      {"month": "Mar", "channel": "Direct", "orders": 128},
      {"month": "Mar", "channel": "Partner", "orders": 81},
      {"month": "Mar", "channel": "Online", "orders": 140},
      {"month": "Apr", "channel": "Direct", "orders": 150},
      {"month": "Apr", "channel": "Partner", "orders": 77},
      {"month": "Apr", "channel": "Online", "orders": 162}
    ]
  },
  "encoding": {
    "x": {"field": "month", "type": "nominal", "sort": ["Jan", "Feb", "Mar", "Apr"], "title": null}
  },
  "layer": [
    {
      "mark": "bar",
      "encoding": {
        "y": {"field": "orders", "type": "quantitative", "title": "Orders"},
        "color": {"field": "channel", "type": "nominal", "title": "Channel"}
      }
    },
    {
      "mark": {"type": "text", "baseline": "bottom", "dy": -3},
      "encoding": {
        "y": {"field": "orders", "type": "quantitative", "aggregate": "sum"},
        "text": {"field": "orders", "type": "quantitative", "aggregate": "sum"}
      }
    }
  ]
}
```

## 100% stacked bar

Comparing the mix (shares) across categories when their totals differ, such as survey answers by department.

Traps avoided:
- `"stack": "normalize"` turns each bar into shares of 100%. The axis then needs `"format": "%"`, or it reads 0 to 1.
- The answers are stacked in a meaningful order (`"order"` on a rank field) rather than alphabetically, so "Agree" sits
  next to "Neutral", and the legend follows the same order through `"sort"`.

```vega-lite
{
  "data": {
    "values": [
      {"dept": "Sales", "answer": "Agree", "rank": 1, "n": 42},
      {"dept": "Sales", "answer": "Neutral", "rank": 2, "n": 18},
      {"dept": "Sales", "answer": "Disagree", "rank": 3, "n": 10},
      {"dept": "Engineering", "answer": "Agree", "rank": 1, "n": 61},
      {"dept": "Engineering", "answer": "Neutral", "rank": 2, "n": 30},
      {"dept": "Engineering", "answer": "Disagree", "rank": 3, "n": 9},
      {"dept": "Support", "answer": "Agree", "rank": 1, "n": 20},
      {"dept": "Support", "answer": "Neutral", "rank": 2, "n": 12},
      {"dept": "Support", "answer": "Disagree", "rank": 3, "n": 14}
    ]
  },
  "mark": "bar",
  "encoding": {
    "y": {"field": "dept", "type": "nominal", "title": null},
    "x": {
      "field": "n", "type": "quantitative", "stack": "normalize", "title": "Share of answers",
      "axis": {"format": "%"}
    },
    "color": {
      "field": "answer", "type": "nominal", "title": "Answer",
      "sort": ["Agree", "Neutral", "Disagree"]
    },
    "order": {"field": "rank", "type": "quantitative"}
  }
}
```

## Diverging bar

Values that can be positive or negative, such as change against last year or variance against budget.

Traps avoided:
- Colour says the direction, so it is set on purpose, with a `condition`: red below zero, green above. Leaving it to
  the theme could colour a loss green.
- A colour *field* on one layer (say a calculated `direction`) quietly drops the shared axis `sort`, and the bars come
  out alphabetical. A `condition` has no field, so the sort holds.
- The labels sit outside the end of each bar on either side, with `align` chosen per bar by an expression.
- A rule at zero marks the baseline, from one row of its own (`{"zero": 0}`), so it is drawn once. The bars are
  sorted by a field (`"sort": {"field": "change", "order": "descending"}`): `"-x"` fails when a layer has nothing
  on `x`.

```vega-lite
{
  "data": {
    "values": [
      {"product": "Analytics", "change": 18}, {"product": "Storage", "change": 7},
      {"product": "Messaging", "change": -4}, {"product": "Search", "change": 12},
      {"product": "Legacy API", "change": -15}
    ]
  },
  "encoding": {
    "y": {
      "field": "product", "type": "nominal", "title": null,
      "sort": {"field": "change", "order": "descending"}
    }
  },
  "layer": [
    {
      "mark": "bar",
      "encoding": {
        "x": {
          "field": "change", "type": "quantitative", "title": "Change in revenue (%)",
          "scale": {"domain": [-20, 25]}
        },
        "color": {
          "condition": {"test": "datum.change < 0", "value": "#e04a4a"},
          "value": "#1fb07a"
        }
      }
    },
    {
      "mark": {
        "type": "text",
        "align": {"expr": "datum.change < 0 ? 'right' : 'left'"},
        "dx": {"expr": "datum.change < 0 ? -4 : 4"}
      },
      "encoding": {
        "x": {"field": "change", "type": "quantitative"},
        "text": {"field": "change", "type": "quantitative", "format": "+d"}
      }
    },
    {
      "data": {"values": [{"zero": 0}]},
      "mark": "rule",
      "encoding": {"x": {"field": "zero", "type": "quantitative"}, "y": null}
    }
  ]
}
```

## Line over time

One measure over real dates, such as monthly active users.

Traps avoided:
- Dates are `temporal`, so the gaps between them are true to time and the axis picks sensible ticks.
- A date written `2026-01-01` is read as midnight UTC, and Vega shows times in the server's local zone, so labels can
  slip to the previous day or month. `"scale": {"type": "utc"}` keeps them on the date written.
- `"format": "%b %Y"` labels months as "Jan 2026" rather than Vega's mixed default.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2025-07-01", "users": 18200}, {"month": "2025-08-01", "users": 19100},
      {"month": "2025-09-01", "users": 21400}, {"month": "2025-10-01", "users": 22900},
      {"month": "2025-11-01", "users": 22100}, {"month": "2025-12-01", "users": 24800},
      {"month": "2026-01-01", "users": 27300}, {"month": "2026-02-01", "users": 28900},
      {"month": "2026-03-01", "users": 31200}
    ]
  },
  "mark": "line",
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null, "scale": {"type": "utc"},
      "axis": {"format": "%b %Y", "tickCount": "month"}
    },
    "y": {"field": "users", "type": "quantitative", "title": "Monthly active users"}
  }
}
```

## Several series

Comparing the same measure for a few groups over time. Up to about five lines stay readable.

Traps avoided:
- The data is long: one row per date and series, with the series in its own field for `color`. A wide table (one column
  per series) needs a `fold` transform first, as in the dumbbell example.
- `"scale": {"type": "utc"}` again, so the dates stay put.
- The y axis starts at zero only if that is meaningful. Here it is, so the default is kept.

```vega-lite
{
  "data": {
    "values": [
      {"week": "2026-01-05", "env": "Production", "deploys": 14},
      {"week": "2026-01-05", "env": "Staging", "deploys": 31},
      {"week": "2026-01-05", "env": "Preview", "deploys": 52},
      {"week": "2026-01-12", "env": "Production", "deploys": 17},
      {"week": "2026-01-12", "env": "Staging", "deploys": 28},
      {"week": "2026-01-12", "env": "Preview", "deploys": 60},
      {"week": "2026-01-19", "env": "Production", "deploys": 12},
      {"week": "2026-01-19", "env": "Staging", "deploys": 35},
      {"week": "2026-01-19", "env": "Preview", "deploys": 47},
      {"week": "2026-01-26", "env": "Production", "deploys": 19},
      {"week": "2026-01-26", "env": "Staging", "deploys": 39},
      {"week": "2026-01-26", "env": "Preview", "deploys": 66}
    ]
  },
  "mark": "line",
  "encoding": {
    "x": {
      "field": "week", "type": "temporal", "title": "Week starting", "scale": {"type": "utc"},
      "axis": {"format": "%d %b", "tickCount": "week"}
    },
    "y": {"field": "deploys", "type": "quantitative", "title": "Deploys"},
    "color": {"field": "env", "type": "nominal", "title": "Environment"}
  }
}
```

## Line with points

Few data points, or uneven gaps between them, where the reader should see where each measurement is.

Traps avoided:
- `"point": true` on the line mark draws the points in the line's colour. A separate point layer would need its own
  encodings kept in step.
- Quarterly dates use `"format": "Q%q %Y"` for "Q1 2026".

```vega-lite
{
  "data": {
    "values": [
      {"quarter": "2025-01-01", "nps": 31}, {"quarter": "2025-04-01", "nps": 34},
      {"quarter": "2025-07-01", "nps": 29}, {"quarter": "2025-10-01", "nps": 38},
      {"quarter": "2026-01-01", "nps": 42}
    ]
  },
  "mark": {"type": "line", "point": true},
  "encoding": {
    "x": {
      "field": "quarter", "type": "temporal", "title": null, "scale": {"type": "utc"},
      "axis": {"format": "Q%q %Y", "tickCount": "quarter"}
    },
    "y": {"field": "nps", "type": "quantitative", "title": "Net Promoter Score"}
  }
}
```

## Step line

A value that holds until it changes, such as a price, a plan limit or headcount. The honest choice for such data.

Traps avoided:
- `"interpolate": "step-after"` holds each value until the next date. Plain `"step"` puts the jump halfway between
  dates, which shows a change before it happened.
- `monotone` or `basis` would show a gradual slope where the value jumped.

```vega-lite
{
  "data": {
    "values": [
      {"date": "2025-01-01", "price": 49}, {"date": "2025-05-01", "price": 59},
      {"date": "2025-09-01", "price": 59}, {"date": "2026-02-01", "price": 69},
      {"date": "2026-06-01", "price": 69}
    ]
  },
  "mark": {"type": "line", "interpolate": "step-after", "point": true},
  "encoding": {
    "x": {
      "field": "date", "type": "temporal", "title": null, "scale": {"type": "utc"},
      "axis": {"format": "%b %Y"}
    },
    "y": {
      "field": "price", "type": "quantitative", "title": "Price per seat ($)",
      "scale": {"zero": false}
    }
  }
}
```

## Smoothed line

A measure that really changes gradually, sampled now and then, such as a temperature or a weight. Use with care.

Traps avoided:
- `"interpolate": "monotone"` curves through every point without overshooting, so it never shows a peak or dip that
  isn't in the data. `"basis"` misses the points and `"cardinal"` can overshoot: avoid both for measured data.
- The points stay visible, so the reader can see which values were measured and which are the curve.
- Smoothing suggests values between measurements. For counts per period (sales per month), use a straight line.

```vega-lite
{
  "data": {
    "values": [
      {"hour": 0, "load": 22}, {"hour": 3, "load": 18}, {"hour": 6, "load": 31},
      {"hour": 9, "load": 68}, {"hour": 12, "load": 74}, {"hour": 15, "load": 71},
      {"hour": 18, "load": 55}, {"hour": 21, "load": 37}, {"hour": 24, "load": 24}
    ]
  },
  "mark": {"type": "line", "interpolate": "monotone", "point": true},
  "encoding": {
    "x": {
      "field": "hour", "type": "quantitative", "title": "Hour of day", "scale": {"domain": [0, 24]},
      "axis": {"values": [0, 6, 12, 18, 24]}
    },
    "y": {"field": "load", "type": "quantitative", "title": "CPU load (%)"}
  }
}
```

## Area

One measure over time where the amount (the area under it) matters, such as storage used.

Traps avoided:
- `"line": true` draws a crisp edge on top of Codoc's faint fill, so the shape reads well in both schemes.
- An area must start at zero to be honest; Vega-Lite does that by default, so `zero` is left alone.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2025-10-01", "tb": 41}, {"month": "2025-11-01", "tb": 46},
      {"month": "2025-12-01", "tb": 52}, {"month": "2026-01-01", "tb": 55},
      {"month": "2026-02-01", "tb": 63}, {"month": "2026-03-01", "tb": 70}
    ]
  },
  "mark": {"type": "area", "line": true},
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null, "scale": {"type": "utc"},
      "axis": {"format": "%b %Y", "tickCount": "month"}
    },
    "y": {"field": "tb", "type": "quantitative", "title": "Storage used (TB)"}
  }
}
```

## Stacked area

A total over time and how its parts contribute. Only the bottom layer and the total read precisely.

Traps avoided:
- Codoc draws areas faint, for one area under a line. Stacked areas need `"opacity": 1`, or the layers wash
  into each other.
- A series missing a date leaves a hole or a spike in a stacked area. `"impute": {"value": 0}` fills the gap (here
  Enterprise has no January row) so every layer has every date.
- A `sort` on `color` orders the legend but not the stack. The stack follows `order`, here a rank worked out with
  `indexof`, so the largest, steadiest series (Free) sits at the bottom, and the legend lists them in the same order.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2026-01-01", "plan": "Free", "users": 900},
      {"month": "2026-01-01", "plan": "Team", "users": 300},
      {"month": "2026-02-01", "plan": "Free", "users": 950},
      {"month": "2026-02-01", "plan": "Team", "users": 340},
      {"month": "2026-02-01", "plan": "Enterprise", "users": 120},
      {"month": "2026-03-01", "plan": "Free", "users": 1010},
      {"month": "2026-03-01", "plan": "Team", "users": 410},
      {"month": "2026-03-01", "plan": "Enterprise", "users": 160},
      {"month": "2026-04-01", "plan": "Free", "users": 1080},
      {"month": "2026-04-01", "plan": "Team", "users": 460},
      {"month": "2026-04-01", "plan": "Enterprise", "users": 230}
    ]
  },
  "mark": {"type": "area", "opacity": 1},
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null, "scale": {"type": "utc"},
      "axis": {"format": "%b %Y", "tickCount": "month"}
    },
    "y": {"field": "users", "type": "quantitative", "title": "Users", "impute": {"value": 0}},
    "color": {
      "field": "plan", "type": "nominal", "title": "Plan", "sort": ["Free", "Team", "Enterprise"]
    },
    "order": {"field": "rank", "type": "quantitative"}
  },
  "transform": [{"calculate": "indexof(['Free', 'Team', 'Enterprise'], datum.plan)", "as": "rank"}]
}
```

## Linear trend line

Showing the overall direction of a series over time, beside the actual values.

Traps avoided:
- The `regression` transform fits the line from the same data, in its own layer. `"on"` is the x field and
  `"regression"` the y field.
- A regression on dates fits against time in milliseconds, which is fine for drawing; the trend layer needs the same
  `utc` scale as the data.
- The trend is grey and dashed and the data solid, so it doesn't read as a measurement.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2025-07-01", "mrr": 182}, {"month": "2025-08-01", "mrr": 190},
      {"month": "2025-09-01", "mrr": 186}, {"month": "2025-10-01", "mrr": 201},
      {"month": "2025-11-01", "mrr": 214}, {"month": "2025-12-01", "mrr": 209},
      {"month": "2026-01-01", "mrr": 226}, {"month": "2026-02-01", "mrr": 231}
    ]
  },
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null, "scale": {"type": "utc"},
      "axis": {"format": "%b %Y", "tickCount": "month"}
    },
    "y": {"field": "mrr", "type": "quantitative", "title": "MRR ($k)", "scale": {"zero": false}}
  },
  "layer": [
    {"mark": {"type": "line", "point": true}},
    {
      "transform": [{"regression": "mrr", "on": "month"}],
      "mark": {"type": "line", "color": "#5b6370", "strokeDash": [6, 4]}
    }
  ]
}
```

## Loess trend

Noisy data where the trend bends, such as daily sign-ups, and a straight line would hide the bend.

Traps avoided:
- `"loess"` follows the bend locally. `"bandwidth"` (0 to 1) sets how smooth: higher is smoother. 0.3 to 0.5 suits
  most business series.
- The raw points are drawn faintly so the reader can judge how noisy the data is.

```vega-lite
{
  "data": {
    "values": [
      {"day": 1, "signups": 42}, {"day": 2, "signups": 51}, {"day": 3, "signups": 38},
      {"day": 4, "signups": 47}, {"day": 5, "signups": 55}, {"day": 6, "signups": 61},
      {"day": 7, "signups": 58}, {"day": 8, "signups": 70}, {"day": 9, "signups": 66},
      {"day": 10, "signups": 79}, {"day": 11, "signups": 74}, {"day": 12, "signups": 81},
      {"day": 13, "signups": 77}, {"day": 14, "signups": 72}, {"day": 15, "signups": 68},
      {"day": 16, "signups": 63}, {"day": 17, "signups": 66}, {"day": 18, "signups": 59},
      {"day": 19, "signups": 62}, {"day": 20, "signups": 57}
    ]
  },
  "encoding": {
    "x": {"field": "day", "type": "quantitative", "title": "Day of campaign"},
    "y": {"field": "signups", "type": "quantitative", "title": "Sign-ups"}
  },
  "layer": [
    {"mark": {"type": "point", "opacity": 0.5}},
    {
      "transform": [{"loess": "signups", "on": "day", "bandwidth": 0.4}],
      "mark": {"type": "line", "strokeWidth": 2.5}
    }
  ]
}
```

## Moving average

A noisy daily series where the reader needs the underlying level, such as daily orders.

Traps avoided:
- The `window` transform needs `"sort"` by date, or it averages rows in whatever order they were written.
- `"frame": [-6, 0]` averages each day with the six before it (a trailing 7-day average). The first six days average
  fewer days, so they are left out with a `filter` on the row number.
- The raw series stays, faint, behind the average.

```vega-lite
{
  "data": {
    "values": [
      {"date": "2026-03-01", "orders": 120}, {"date": "2026-03-02", "orders": 135},
      {"date": "2026-03-03", "orders": 98}, {"date": "2026-03-04", "orders": 142},
      {"date": "2026-03-05", "orders": 160}, {"date": "2026-03-06", "orders": 88},
      {"date": "2026-03-07", "orders": 79}, {"date": "2026-03-08", "orders": 131},
      {"date": "2026-03-09", "orders": 148}, {"date": "2026-03-10", "orders": 110},
      {"date": "2026-03-11", "orders": 156}, {"date": "2026-03-12", "orders": 171},
      {"date": "2026-03-13", "orders": 95}, {"date": "2026-03-14", "orders": 90},
      {"date": "2026-03-15", "orders": 140}, {"date": "2026-03-16", "orders": 162},
      {"date": "2026-03-17", "orders": 125}, {"date": "2026-03-18", "orders": 170},
      {"date": "2026-03-19", "orders": 183}, {"date": "2026-03-20", "orders": 104},
      {"date": "2026-03-21", "orders": 99}
    ]
  },
  "encoding": {
    "x": {
      "field": "date", "type": "temporal", "title": null, "scale": {"type": "utc"},
      "axis": {"format": "%d %b"}
    }
  },
  "layer": [
    {
      "mark": {"type": "line", "opacity": 0.35},
      "encoding": {"y": {"field": "orders", "type": "quantitative", "title": "Orders per day"}}
    },
    {
      "transform": [
        {
          "window": [
            {"op": "mean", "field": "orders", "as": "avg"}, {"op": "row_number", "as": "n"}
          ],
          "sort": [{"field": "date"}],
          "frame": [-6, 0]
        },
        {"filter": "datum.n >= 7"}
      ],
      "mark": {"type": "line", "strokeWidth": 2.5},
      "encoding": {"y": {"field": "avg", "type": "quantitative"}}
    }
  ]
}
```

## Target or reference line

A goal, limit, budget or average drawn across the chart, so the reader sees which values are over it.

Traps avoided:
- The limit and its label have their own one-row `data`. A text layer that shares the chart's data draws its label
  once per row, stacked on top of itself.
- The rule is a `y` on the same scale as the bars, so it can't drift from them, and the scale grows to include it.
- The label sits just past the right end of the line, with `"x": {"value": {"expr": "width"}}` (the plot's width,
  whatever it is) and `"align": "left"`, so no bar can cover it. The chart grows to make room.
- Bars start at zero. Narrowing a bar chart's scale (`"domain": [80, 100]`) makes small gaps look huge, and the bars
  then draw far below the axis. To compare values close together, use points or a dot plot instead.

```vega-lite
{
  "data": {
    "values": [
      {"service": "API", "p95": 180}, {"service": "Web", "p95": 240},
      {"service": "Workers", "p95": 410}, {"service": "Search", "p95": 290},
      {"service": "Billing", "p95": 350}
    ]
  },
  "layer": [
    {
      "mark": "bar",
      "encoding": {
        "x": {"field": "service", "type": "nominal", "title": null},
        "y": {"field": "p95", "type": "quantitative", "title": "p95 latency (ms)"}
      }
    },
    {
      "data": {"values": [{"limit": 300, "label": "Limit 300 ms"}]},
      "encoding": {"y": {"field": "limit", "type": "quantitative"}},
      "layer": [
        {"mark": {"type": "rule", "strokeDash": [6, 4]}},
        {
          "mark": {"type": "text", "align": "left", "dx": 6},
          "encoding": {
            "x": {"value": {"expr": "width"}},
            "text": {"field": "label", "type": "nominal"}
          }
        }
      ]
    }
  ]
}
```

## Forecast with an interval

Actual values followed by a forecast and its likely range, for a plan or budget.

Traps avoided:
- The range is an `area` with `y` (low) and `y2` (high), drawn first so the lines sit on top of it.
- The forecast line is dashed and starts at the last actual value, so the two lines join.
- Each row says whether it is actual or forecast; `filter` picks the rows for each layer.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2026-01-01", "kind": "Actual", "value": 120},
      {"month": "2026-02-01", "kind": "Actual", "value": 128},
      {"month": "2026-03-01", "kind": "Actual", "value": 135},
      {"month": "2026-04-01", "kind": "Actual", "value": 141},
      {"month": "2026-04-01", "kind": "Forecast", "value": 141, "low": 141, "high": 141},
      {"month": "2026-05-01", "kind": "Forecast", "value": 148, "low": 140, "high": 156},
      {"month": "2026-06-01", "kind": "Forecast", "value": 155, "low": 142, "high": 169},
      {"month": "2026-07-01", "kind": "Forecast", "value": 162, "low": 143, "high": 183}
    ]
  },
  "encoding": {
    "x": {
      "field": "month", "type": "temporal", "title": null, "scale": {"type": "utc"},
      "axis": {"format": "%b %Y", "tickCount": "month"}
    }
  },
  "layer": [
    {
      "transform": [{"filter": "datum.kind === 'Forecast'"}],
      "mark": "area",
      "encoding": {
        "y": {"field": "low", "type": "quantitative", "title": "Bookings ($k)"},
        "y2": {"field": "high"}
      }
    },
    {
      "transform": [{"filter": "datum.kind === 'Actual'"}],
      "mark": {"type": "line", "point": true},
      "encoding": {"y": {"field": "value", "type": "quantitative"}}
    },
    {
      "transform": [{"filter": "datum.kind === 'Forecast'"}],
      "mark": {"type": "line", "strokeDash": [6, 4]},
      "encoding": {"y": {"field": "value", "type": "quantitative"}}
    }
  ]
}
```

## Range band (percentiles)

A typical value and its spread over time, such as median latency with the 10th to 90th percentile around it.

Traps avoided:
- With the range already worked out, `errorband` takes it as `y` and `y2`.
- Don't let `errorband` or `errorbar` work out a confidence interval (`"extent": "ci"`) from raw rows: it resamples at
  random, so the picture changes every time it is drawn. Work the range out first, or use `"stderr"` or `"stdev"`.

```vega-lite
{
  "data": {
    "values": [
      {"day": "2026-05-01", "p10": 82, "p50": 120, "p90": 210},
      {"day": "2026-05-02", "p10": 80, "p50": 118, "p90": 240},
      {"day": "2026-05-03", "p10": 85, "p50": 131, "p90": 265},
      {"day": "2026-05-04", "p10": 90, "p50": 140, "p90": 310},
      {"day": "2026-05-05", "p10": 84, "p50": 125, "p90": 230},
      {"day": "2026-05-06", "p10": 79, "p50": 115, "p90": 205},
      {"day": "2026-05-07", "p10": 81, "p50": 117, "p90": 198}
    ]
  },
  "encoding": {
    "x": {
      "field": "day", "type": "temporal", "title": null, "scale": {"type": "utc"},
      "axis": {"format": "%a %d", "tickCount": "day"}
    }
  },
  "layer": [
    {
      "mark": "errorband",
      "encoding": {
        "y": {"field": "p10", "type": "quantitative", "title": "Latency (ms), p10 to p90"},
        "y2": {"field": "p90"}
      }
    },
    {
      "mark": {"type": "line", "point": true},
      "encoding": {"y": {"field": "p50", "type": "quantitative"}}
    }
  ]
}
```

## Pie with percentage labels

Parts of a whole, with at most five or six parts. Prefer a sorted bar when the parts are close in size.

Traps avoided:
- The labels are worked out as percentages with `joinaggregate` (the total) and `calculate`, then formatted `.0%`.
- The slices and the labels both have `"order"` on the same field, largest first. Without the same `order`, the text
  stacks in a different order from the arcs and labels land on the wrong slices.
- `"stack": true` on the text's `theta` puts each label at the middle of its slice's angle.
- The labels sit outside the pie (`radius` beyond `outerRadius`), so they keep the theme's ink colour. The `color`
  encoding is on the arc layer only: if it were shared, it would colour the text too.

```vega-lite
{
  "width": 300, "height": 300,
  "data": {
    "values": [
      {"source": "Organic search", "visits": 4200}, {"source": "Direct", "visits": 2600},
      {"source": "Referral", "visits": 1300}, {"source": "Social", "visits": 900},
      {"source": "Email", "visits": 500}
    ]
  },
  "transform": [
    {"joinaggregate": [{"op": "sum", "field": "visits", "as": "total"}]},
    {"calculate": "datum.visits / datum.total", "as": "share"}
  ],
  "encoding": {
    "theta": {"field": "visits", "type": "quantitative", "stack": true},
    "order": {"field": "visits", "type": "quantitative", "sort": "descending"}
  },
  "layer": [
    {
      "mark": {"type": "arc", "outerRadius": 110},
      "encoding": {
        "color": {
          "field": "source", "type": "nominal", "title": "Source",
          "sort": {"field": "visits", "order": "descending"}
        }
      }
    },
    {
      "mark": {"type": "text", "radius": 130},
      "encoding": {"text": {"field": "share", "type": "quantitative", "format": ".0%"}}
    }
  ]
}
```

## Donut with labels inside

Parts of a whole with the total in the middle, a common summary for a budget or a portfolio.

Traps avoided:
- Slices and labels share one `order` encoding (largest first), so each label lands on its own slice.
- The labels sit on the slices, so they are white and the slice colours are set dark enough to read white text on.
  A shared `color` field encoding would override a text mark's colour, so the text layer sets
  `"color": {"value": "white"}` in its own encoding.
- The total in the middle is its own layer, with its own data added up by `aggregate`, and no `theta`.

```vega-lite
{
  "width": 300, "height": 300,
  "data": {
    "values": [
      {"item": "Salaries", "cost": 620}, {"item": "Cloud", "cost": 180},
      {"item": "Marketing", "cost": 120}, {"item": "Office", "cost": 80}
    ]
  },
  "transform": [
    {"joinaggregate": [{"op": "sum", "field": "cost", "as": "total"}]},
    {"calculate": "datum.cost / datum.total", "as": "share"}
  ],
  "layer": [
    {
      "mark": {"type": "arc", "innerRadius": 70, "outerRadius": 130},
      "encoding": {
        "theta": {"field": "cost", "type": "quantitative", "stack": true},
        "order": {"field": "cost", "type": "quantitative", "sort": "descending"},
        "color": {
          "field": "item", "type": "nominal", "title": "Cost",
          "sort": {"field": "cost", "order": "descending"},
          "scale": {"range": ["#2b4c7e", "#8a3b12", "#2e6b3a", "#6b3d7a"]}
        }
      }
    },
    {
      "mark": {"type": "text", "radius": 100, "fontWeight": "bold"},
      "encoding": {
        "theta": {"field": "cost", "type": "quantitative", "stack": true},
        "order": {"field": "cost", "type": "quantitative", "sort": "descending"},
        "text": {"field": "share", "type": "quantitative", "format": ".0%"},
        "color": {"value": "white"}
      }
    },
    {
      "transform": [{"aggregate": [{"op": "sum", "field": "cost", "as": "sum"}]}],
      "mark": {"type": "text", "fontSize": 20, "fontWeight": "bold"},
      "encoding": {"text": {"field": "sum", "type": "quantitative", "format": "$,.0f"}}
    }
  ]
}
```

## Scatter with a trend line

Whether two measures move together, such as ad spend against sign-ups, one point per item.

Traps avoided:
- The trend is a `regression` layer over the same data, and a text layer reports its fit: `"params": true` gives one
  row with `rSquared`, which `calculate` turns into a label.
- `"scale": {"zero": false}` keeps the points from bunching up when no value is near zero.

```vega-lite
{
  "data": {
    "values": [
      {"spend": 12, "signups": 310}, {"spend": 18, "signups": 402}, {"spend": 9, "signups": 260},
      {"spend": 25, "signups": 530}, {"spend": 30, "signups": 590}, {"spend": 15, "signups": 330},
      {"spend": 22, "signups": 488}, {"spend": 27, "signups": 512}, {"spend": 34, "signups": 660},
      {"spend": 20, "signups": 455}
    ]
  },
  "encoding": {
    "x": {
      "field": "spend", "type": "quantitative", "title": "Ad spend ($k)", "scale": {"zero": false}
    },
    "y": {
      "field": "signups", "type": "quantitative", "title": "Sign-ups", "scale": {"zero": false}
    }
  },
  "layer": [
    {"mark": {"type": "point", "filled": true}},
    {
      "transform": [{"regression": "signups", "on": "spend"}],
      "mark": {"type": "line", "color": "#5b6370", "strokeDash": [6, 4]}
    },
    {
      "transform": [
        {"regression": "signups", "on": "spend", "params": true},
        {"calculate": "'R² = ' + format(datum.rSquared, '.2f')", "as": "fit"}
      ],
      "mark": {"type": "text", "align": "left", "baseline": "top"},
      "encoding": {
        "x": {"value": 8},
        "y": {"value": 8},
        "text": {"field": "fit", "type": "nominal"}
      }
    }
  ]
}
```

## Bubble chart

Three measures per item: two positions and a size, such as growth, margin and revenue per product.

Traps avoided:
- `size` maps to the bubble's area, which is what readers compare. `"range"` sets the smallest and largest areas, so
  small items stay visible.
- Bubbles overlap, so they are half transparent with an outline.
- Each bubble is named by a text layer beside it, rather than a colour legend the reader must match up.

```vega-lite
{
  "data": {
    "values": [
      {"product": "Core", "growth": 4, "margin": 62, "revenue": 5200},
      {"product": "Analytics", "growth": 22, "margin": 48, "revenue": 1800},
      {"product": "Mobile", "growth": 15, "margin": 35, "revenue": 950},
      {"product": "Services", "growth": -3, "margin": 18, "revenue": 1400},
      {"product": "Marketplace", "growth": 31, "margin": 27, "revenue": 400}
    ]
  },
  "encoding": {
    "x": {"field": "growth", "type": "quantitative", "title": "Growth (%)"},
    "y": {"field": "margin", "type": "quantitative", "title": "Gross margin (%)"}
  },
  "layer": [
    {
      "mark": {"type": "circle", "opacity": 0.5, "stroke": "#5b6370", "strokeWidth": 1},
      "encoding": {
        "size": {
          "field": "revenue", "type": "quantitative", "title": "Revenue ($k)",
          "scale": {"range": [80, 2400]}
        }
      }
    },
    {
      "mark": {"type": "text", "align": "left", "dx": 12, "dy": -12},
      "encoding": {"text": {"field": "product", "type": "nominal"}}
    }
  ]
}
```

## Histogram

How values are spread, from raw measurements such as response times or deal sizes.

Traps avoided:
- `"bin": {"step": 100}` gives round bin edges a reader can name. `maxbins` alone picks edges for you.
- The bars are counted with `"aggregate": "count"`. Data already counted into bins is drawn with `"bin": "binned"`
  and `x2` instead.
- The x axis labels the bin edges, not the middles.

```vega-lite
{
  "data": {
    "values": [
      {"ms": 120}, {"ms": 135}, {"ms": 150}, {"ms": 180}, {"ms": 190}, {"ms": 210}, {"ms": 215},
      {"ms": 220}, {"ms": 230}, {"ms": 240}, {"ms": 245}, {"ms": 250}, {"ms": 260}, {"ms": 265},
      {"ms": 270}, {"ms": 280}, {"ms": 290}, {"ms": 300}, {"ms": 310}, {"ms": 320}, {"ms": 335},
      {"ms": 350}, {"ms": 370}, {"ms": 390}, {"ms": 420}, {"ms": 460}, {"ms": 510}, {"ms": 640}
    ]
  },
  "mark": "bar",
  "encoding": {
    "x": {
      "field": "ms", "type": "quantitative", "bin": {"step": 100},
      "title": "Response time (ms)"
    },
    "y": {"aggregate": "count", "type": "quantitative", "title": "Requests"}
  }
}
```

## Box plot

Comparing how values are spread across a few groups, from raw measurements.

Traps avoided:
- `boxplot` works the quartiles out itself, so it needs the raw values, not a summary. With quartiles already
  worked out, use the next example.
- `"extent": 1.5` draws whiskers to 1.5 times the box's height and outliers as points beyond them. `"min-max"` hides
  outliers inside the whiskers.
- Groups sit on the y axis, so their names read across.

```vega-lite
{
  "data": {
    "values": [
      {"svc": "Auth", "ms": 40}, {"svc": "Auth", "ms": 45}, {"svc": "Auth", "ms": 52},
      {"svc": "Auth", "ms": 48}, {"svc": "Auth", "ms": 61}, {"svc": "Auth", "ms": 55},
      {"svc": "Auth", "ms": 150}, {"svc": "Search", "ms": 90}, {"svc": "Search", "ms": 120},
      {"svc": "Search", "ms": 135}, {"svc": "Search", "ms": 110}, {"svc": "Search", "ms": 160},
      {"svc": "Search", "ms": 142}, {"svc": "Search", "ms": 128}, {"svc": "Upload", "ms": 210},
      {"svc": "Upload", "ms": 260}, {"svc": "Upload", "ms": 300}, {"svc": "Upload", "ms": 240},
      {"svc": "Upload", "ms": 330}, {"svc": "Upload", "ms": 280}, {"svc": "Upload", "ms": 275}
    ]
  },
  "mark": {"type": "boxplot", "extent": 1.5},
  "encoding": {
    "y": {"field": "svc", "type": "nominal", "title": null},
    "x": {"field": "ms", "type": "quantitative", "title": "Latency (ms)"}
  }
}
```

## Box plot from a summary

The spread of each group when you have its quartiles but not the raw values.

Traps avoided:
- `boxplot` can't take a summary, so the box is drawn from its parts: a `rule` from min to max, a `bar` from the
  first to the third quartile, and a `tick` at the median.
- The box is drawn faint (`"opacity": 0.35`) so the median tick, in the full colour, stands out in both light and dark
  schemes without setting any colour.

```vega-lite
{
  "data": {
    "values": [
      {"team": "Platform", "min": 1, "q1": 3, "median": 5, "q3": 8, "max": 14},
      {"team": "Payments", "min": 2, "q1": 4, "median": 6, "q3": 9, "max": 12},
      {"team": "Mobile", "min": 1, "q1": 2, "median": 3, "q3": 5, "max": 9}
    ]
  },
  "encoding": {"y": {"field": "team", "type": "nominal", "title": null}},
  "layer": [
    {
      "mark": "rule",
      "encoding": {
        "x": {"field": "min", "type": "quantitative", "title": "Days from start to merge"},
        "x2": {"field": "max"}
      }
    },
    {
      "mark": {"type": "bar", "size": 16, "opacity": 0.35},
      "encoding": {"x": {"field": "q1", "type": "quantitative"}, "x2": {"field": "q3"}}
    },
    {
      "mark": {"type": "tick", "size": 16, "thickness": 3},
      "encoding": {"x": {"field": "median", "type": "quantitative"}}
    }
  ]
}
```

## Heatmap

A measure across two categories, such as incidents by weekday and hour, or a cohort's retention.

Traps avoided:
- `rect` with two ordinal axes and a quantitative `color` gives a cell per pair. `"paddingInner": 0` on both scales
  closes the gaps Codoc leaves between bars, and `"grid": false` keeps gridlines from striking through the cells.
- Each cell is labelled, and the text colour flips to white on dark cells with a `condition`, so every number stays
  readable. The threshold sits near the middle of the colour scale.
- Weekdays are given a `sort` list; alphabetical order would put Fri first.

```vega-lite
{
  "data": {
    "values": [
      {"day": "Mon", "slot": "00-06", "n": 2}, {"day": "Mon", "slot": "06-12", "n": 14},
      {"day": "Mon", "slot": "12-18", "n": 21}, {"day": "Mon", "slot": "18-24", "n": 6},
      {"day": "Tue", "slot": "00-06", "n": 1}, {"day": "Tue", "slot": "06-12", "n": 11},
      {"day": "Tue", "slot": "12-18", "n": 25}, {"day": "Tue", "slot": "18-24", "n": 8},
      {"day": "Wed", "slot": "00-06", "n": 3}, {"day": "Wed", "slot": "06-12", "n": 18},
      {"day": "Wed", "slot": "12-18", "n": 30}, {"day": "Wed", "slot": "18-24", "n": 9},
      {"day": "Thu", "slot": "00-06", "n": 2}, {"day": "Thu", "slot": "06-12", "n": 16},
      {"day": "Thu", "slot": "12-18", "n": 27}, {"day": "Thu", "slot": "18-24", "n": 7},
      {"day": "Fri", "slot": "00-06", "n": 4}, {"day": "Fri", "slot": "06-12", "n": 12},
      {"day": "Fri", "slot": "12-18", "n": 19}, {"day": "Fri", "slot": "18-24", "n": 5}
    ]
  },
  "encoding": {
    "x": {"field": "slot", "type": "ordinal", "title": "Hour (UTC)", "scale": {"paddingInner": 0}},
    "y": {
      "field": "day", "type": "ordinal", "title": null, "sort": ["Mon", "Tue", "Wed", "Thu", "Fri"],
      "scale": {"paddingInner": 0},
      "axis": {"grid": false}
    }
  },
  "layer": [
    {
      "mark": "rect",
      "encoding": {"color": {"field": "n", "type": "quantitative", "title": "Incidents"}}
    },
    {
      "mark": "text",
      "encoding": {
        "text": {"field": "n", "type": "quantitative"},
        "color": {"condition": {"test": "datum.n >= 15", "value": "white"}, "value": "#1f2328"}
      }
    }
  ]
}
```

## Two measures on two axes

Two measures in different units over the same time, such as revenue and headcount. Use sparingly.

Traps avoided:
- `"resolve": {"scale": {"y": "independent"}}` gives each layer its own y axis, left and right.
- Each layer gets a `color` from a `datum` naming its measure, so the two take the theme's first two colours and a
  legend says which is which. No colour is written in.
- When not to: two axes invite readers to compare heights that mean nothing, and the scales can be tuned to show any
  story. If the measures share a unit, use one axis. If the point is a ratio (revenue per head), chart the ratio.
  Otherwise consider two charts one above the other.

```vega-lite
{
  "data": {
    "values": [
      {"year": "2022", "revenue": 3.1, "staff": 42}, {"year": "2023", "revenue": 4.4, "staff": 51},
      {"year": "2024", "revenue": 6.0, "staff": 58}, {"year": "2025", "revenue": 7.9, "staff": 64},
      {"year": "2026", "revenue": 9.2, "staff": 66}
    ]
  },
  "encoding": {"x": {"field": "year", "type": "ordinal", "title": null}, "color": {"title": null}},
  "layer": [
    {
      "mark": "bar",
      "encoding": {
        "y": {"field": "revenue", "type": "quantitative", "title": "Revenue ($M)"},
        "color": {"datum": "Revenue ($M)"}
      }
    },
    {
      "mark": {"type": "line", "point": true},
      "encoding": {
        "y": {
          "field": "staff", "type": "quantitative", "title": "Staff", "axis": {"orient": "right"}
        },
        "color": {"datum": "Staff"}
      }
    }
  ],
  "resolve": {"scale": {"y": "independent"}}
}
```

## Waterfall (bridge)

How a total moved from one period to the next through increases and decreases, such as a revenue bridge.

Traps avoided:
- Each step's start and end come from a running total (`window` with `sum`), which follows the order written, so
  write the rows in the order of the bridge.
- `"sort": null` keeps the steps in that order on the axis instead of sorting them alphabetically.
- Colour carries meaning (total, increase, decrease), so it is set with a fixed `domain` and `range`.
- Long step names are split on spaces with `split` in `labelExpr`, so they wrap onto two lines instead of overlapping.
- The labels show the change (`+1,300`, `−700`) for steps and the amount for totals. An increase is labelled above its
  bar and a decrease below, so the labels of a rise followed by a fall, which top out at the same height, don't
  collide.

```vega-lite
{
  "data": {
    "values": [
      {"step": "FY25 revenue", "amount": 4200, "kind": "Total"},
      {"step": "New customers", "amount": 1300, "kind": "Increase"},
      {"step": "Expansion", "amount": 600, "kind": "Increase"},
      {"step": "Churn", "amount": -700, "kind": "Decrease"},
      {"step": "Price change", "amount": -400, "kind": "Decrease"},
      {"step": "FY26 revenue", "amount": 0, "kind": "Total"}
    ]
  },
  "transform": [
    {"window": [{"op": "sum", "field": "amount", "as": "end"}]},
    {"calculate": "datum.kind === 'Total' ? 0 : datum.end - datum.amount", "as": "start"},
    {
      "calculate": "datum.amount < 0 ? min(datum.start, datum.end) : max(datum.start, datum.end)",
      "as": "labelAt"
    },
    {
      "calculate": "datum.kind === 'Total' ? format(datum.end, ',') : format(datum.amount, '+,')",
      "as": "label"
    }
  ],
  "encoding": {
    "x": {
      "field": "step", "type": "nominal", "sort": null, "title": null,
      "axis": {"labelExpr": "split(datum.label, ' ')"}
    }
  },
  "layer": [
    {
      "mark": "bar",
      "encoding": {
        "y": {"field": "start", "type": "quantitative", "title": "Revenue ($k)"},
        "y2": {"field": "end"},
        "color": {
          "field": "kind", "type": "nominal", "title": null,
          "scale": {
            "domain": ["Total", "Increase", "Decrease"],
            "range": ["#7a8290", "#1fb07a", "#e04a4a"]
          }
        }
      }
    },
    {
      "mark": {
        "type": "text",
        "baseline": {"expr": "datum.amount < 0 ? 'top' : 'bottom'"},
        "dy": {"expr": "datum.amount < 0 ? 3 : -3"}
      },
      "encoding": {
        "y": {"field": "labelAt", "type": "quantitative"},
        "text": {"field": "label", "type": "nominal"}
      }
    }
  ]
}
```

## Funnel

How many make it through each stage of a process, such as visit to sign-up to purchase.

Traps avoided:
- Each bar is centred by drawing it from minus half its value to plus half (`x` and `x2`), with the axis hidden since
  its numbers would be meaningless.
- `"sort": null` keeps the stages in process order.
- The labels sit just past the right end of each bar, not on it: the last stages are too narrow to hold text. Off the
  bar, they keep the theme's ink colour. The share of the first stage is worked out by `joinaggregate`.

```vega-lite
{
  "data": {
    "values": [
      {"stage": "Visited", "n": 12000}, {"stage": "Signed up", "n": 4100},
      {"stage": "Activated", "n": 2300}, {"stage": "Subscribed", "n": 640}
    ]
  },
  "transform": [
    {"joinaggregate": [{"op": "max", "field": "n", "as": "first"}]},
    {"calculate": "-datum.n / 2", "as": "left"},
    {"calculate": "datum.n / 2", "as": "right"},
    {
      "calculate": "format(datum.n, ',') + ' (' + format(datum.n / datum.first, '.0%') + ')'",
      "as": "label"
    }
  ],
  "encoding": {"y": {"field": "stage", "type": "nominal", "sort": null, "title": null}},
  "layer": [
    {
      "mark": "bar",
      "encoding": {
        "x": {"field": "left", "type": "quantitative", "axis": null},
        "x2": {"field": "right"}
      }
    },
    {
      "mark": {"type": "text", "align": "left", "dx": 6},
      "encoding": {
        "x": {"field": "right", "type": "quantitative"},
        "text": {"field": "label", "type": "nominal"}
      }
    }
  ]
}
```

## Bullet chart

Progress of a few measures against their targets, each with bands for poor, fair and good.

Traps avoided:
- Measures in different units each get their own row and scale: a `facet` by row with
  `"resolve": {"scale": {"x": "independent"}}`.
- The bands are three bars drawn widest first, so the narrower ones sit on top; their greys carry the meaning, so
  they are set.
- The actual value is a thin bar in the theme's colour, and the target a tick, set dark to stand out from the bands.
- Row labels are turned upright with `"labelAngle": 0`, and left-aligned.
- The rows are given a `sort` list; a facet ignores `"sort": null` and goes alphabetical.

```vega-lite
{
  "data": {
    "values": [
      {
        "measure": "Revenue ($k)", "poor": 150, "fair": 225, "good": 300, "actual": 270,
        "target": 250
      },
      {"measure": "Profit (%)", "poor": 20, "fair": 25, "good": 30, "actual": 23, "target": 27},
      {
        "measure": "New customers", "poor": 1400, "fair": 2000, "good": 2500, "actual": 1650,
        "target": 2100
      }
    ]
  },
  "facet": {
    "row": {
      "field": "measure", "type": "nominal", "title": null,
      "sort": ["Revenue ($k)", "Profit (%)", "New customers"],
      "header": {"labelAngle": 0, "labelAlign": "left", "labelOrient": "left"}
    }
  },
  "spacing": 10,
  "spec": {
    "width": 360, "height": 26,
    "layer": [
      {
        "mark": {"type": "bar", "color": "#e8eaed"},
        "encoding": {"x": {"field": "good", "type": "quantitative", "title": null}}
      },
      {
        "mark": {"type": "bar", "color": "#d0d4da"},
        "encoding": {"x": {"field": "fair", "type": "quantitative"}}
      },
      {
        "mark": {"type": "bar", "color": "#b4bac3"},
        "encoding": {"x": {"field": "poor", "type": "quantitative"}}
      },
      {
        "mark": {"type": "bar", "size": 8},
        "encoding": {"x": {"field": "actual", "type": "quantitative"}}
      },
      {
        "mark": {"type": "tick", "size": 20, "thickness": 2.5, "color": "#1f2328"},
        "encoding": {"x": {"field": "target", "type": "quantitative"}}
      }
    ]
  },
  "resolve": {"scale": {"x": "independent"}}
}
```

## Progress against target

How far along several items are, as a share of their own target, such as projects or quotas.

Traps avoided:
- A pale full-length bar behind each item shows what 100% looks like, so a short bar reads as "a long way to go".
- The share is worked out with `calculate` and labelled at the bar's end; the axis is `%` and capped at 100%.
- An item past its target is clamped to 100% for the bar but labelled with its true share.
- The rows are sorted by a field (`"sort": {"field": "share", ...}`). `"-x"` would be ambiguous here, since each layer
  has a different field on `x`, and the rows come out alphabetical.

```vega-lite
{
  "data": {
    "values": [
      {"team": "EMEA", "done": 840, "target": 1000},
      {"team": "Americas", "done": 1210, "target": 1100},
      {"team": "APAC", "done": 390, "target": 800},
      {"team": "Partners", "done": 270, "target": 500}
    ]
  },
  "transform": [
    {"calculate": "datum.done / datum.target", "as": "share"},
    {"calculate": "min(datum.share, 1)", "as": "bar"}, {"calculate": "1", "as": "full"}
  ],
  "encoding": {
    "y": {
      "field": "team", "type": "nominal", "sort": {"field": "share", "order": "descending"},
      "title": null
    }
  },
  "layer": [
    {
      "mark": {"type": "bar", "color": "#e2e4e8"},
      "encoding": {"x": {"field": "full", "type": "quantitative"}}
    },
    {
      "mark": "bar",
      "encoding": {
        "x": {
          "field": "bar", "type": "quantitative", "title": "Share of quota",
          "axis": {"format": "%"},
          "scale": {"domain": [0, 1]}
        }
      }
    },
    {
      "mark": {"type": "text", "align": "left", "dx": 4},
      "encoding": {
        "x": {"field": "bar", "type": "quantitative"},
        "text": {"field": "share", "type": "quantitative", "format": ".0%"}
      }
    }
  ]
}
```

## Gantt or timeline

A schedule: tasks with start and end dates, optionally a "today" line.

Traps avoided:
- Each task is a `bar` with `x` (start) and `x2` (end) as dates, on a `utc` scale so the bars start on the dates
  written.
- The tasks are sorted by start date (`"sort": {"field": "start", "op": "min"}`), not alphabetically.
- An end date means the bar stops at the start of that day. Write the day after the last working day, or the bar
  looks a day short.
- The "today" line and its label have their own one-row `data`, so the label is drawn once, at the top.

```vega-lite
{
  "data": {
    "values": [
      {"task": "Discovery", "start": "2026-01-05", "end": "2026-01-24", "phase": "Plan"},
      {"task": "Design", "start": "2026-01-19", "end": "2026-02-14", "phase": "Plan"},
      {"task": "Build API", "start": "2026-02-09", "end": "2026-03-21", "phase": "Build"},
      {"task": "Build app", "start": "2026-02-23", "end": "2026-04-04", "phase": "Build"},
      {"task": "Beta", "start": "2026-04-06", "end": "2026-04-25", "phase": "Launch"},
      {"task": "Launch", "start": "2026-04-27", "end": "2026-05-02", "phase": "Launch"}
    ]
  },
  "layer": [
    {
      "mark": {"type": "bar", "cornerRadius": 3},
      "encoding": {
        "y": {
          "field": "task", "type": "nominal", "title": null,
          "sort": {"field": "start", "op": "min"}
        },
        "x": {
          "field": "start", "type": "temporal", "title": null, "scale": {"type": "utc"},
          "axis": {"format": "%d %b", "tickCount": "week"}
        },
        "x2": {"field": "end"},
        "color": {"field": "phase", "type": "nominal", "title": "Phase", "sort": null}
      }
    },
    {
      "data": {"values": [{"date": "2026-03-02", "label": "Today"}]},
      "encoding": {"x": {"field": "date", "type": "temporal"}},
      "layer": [
        {"mark": {"type": "rule", "strokeDash": [4, 3]}},
        {
          "mark": {"type": "text", "align": "left", "baseline": "top", "dx": 4},
          "encoding": {"y": {"value": 0}, "text": {"field": "label", "type": "nominal"}}
        }
      ]
    }
  ]
}
```

## Small multiples

The same chart repeated for each group, such as a line per region, when one chart with many lines would tangle.

Traps avoided:
- `facet` in the encoding with `"columns": 2` wraps the panels into a grid. `width` and `height` are each panel's.
- The panels share their axes by default, so heights compare across panels. Make the y scale `independent` only to
  show shapes, and say so in the text.
- The panel titles come from the facet's `header`; the field's name is dropped with `"title": null`.
- Panels are in alphabetical order unless the facet has a `sort` list. `"sort": null` doesn't keep the data's order
  for a facet as it does for an axis.

```vega-lite
{
  "width": 200, "height": 110,
  "data": {
    "values": [
      {"region": "North", "month": "2026-01-01", "sales": 42},
      {"region": "North", "month": "2026-02-01", "sales": 48},
      {"region": "North", "month": "2026-03-01", "sales": 51},
      {"region": "North", "month": "2026-04-01", "sales": 57},
      {"region": "South", "month": "2026-01-01", "sales": 30},
      {"region": "South", "month": "2026-02-01", "sales": 29},
      {"region": "South", "month": "2026-03-01", "sales": 35},
      {"region": "South", "month": "2026-04-01", "sales": 33},
      {"region": "East", "month": "2026-01-01", "sales": 25},
      {"region": "East", "month": "2026-02-01", "sales": 31},
      {"region": "East", "month": "2026-03-01", "sales": 38},
      {"region": "East", "month": "2026-04-01", "sales": 46},
      {"region": "West", "month": "2026-01-01", "sales": 50},
      {"region": "West", "month": "2026-02-01", "sales": 47},
      {"region": "West", "month": "2026-03-01", "sales": 41},
      {"region": "West", "month": "2026-04-01", "sales": 39}
    ]
  },
  "mark": {"type": "line", "point": true},
  "encoding": {
    "facet": {
      "field": "region", "type": "nominal", "columns": 2, "title": null,
      "sort": ["North", "South", "East", "West"]
    },
    "x": {
      "field": "month", "type": "temporal", "title": null, "scale": {"type": "utc"},
      "axis": {"format": "%b", "tickCount": "month"}
    },
    "y": {"field": "sales", "type": "quantitative", "title": "Sales ($k)"}
  }
}
```

## Slope chart

Before and after for a few items, such as each team's score in two surveys, where the reader looks for who rose and
who fell.

Traps avoided:
- One line per item (`detail`) between two ordinal x positions. `"padding": 0.6` on x leaves room for the labels,
  and `"padding": 16` on y keeps the top point clear of the axis labels above it.
- The labels are two text layers: names and values on the left end aligned right, values on the right end aligned
  left, picked with `filter`.
- Check the values don't put two labels at nearly the same height; if they would, use a dumbbell chart instead.

```vega-lite
{
  "width": 360,
  "data": {
    "values": [
      {"team": "Support", "survey": "2025", "score": 62},
      {"team": "Support", "survey": "2026", "score": 78},
      {"team": "Sales", "survey": "2025", "score": 71},
      {"team": "Sales", "survey": "2026", "score": 66},
      {"team": "Engineering", "survey": "2025", "score": 80},
      {"team": "Engineering", "survey": "2026", "score": 84},
      {"team": "Finance", "survey": "2025", "score": 54},
      {"team": "Finance", "survey": "2026", "score": 57}
    ]
  },
  "encoding": {
    "x": {
      "field": "survey", "type": "ordinal", "title": null, "scale": {"padding": 0.6},
      "axis": {"orient": "top", "domain": false, "ticks": false}
    },
    "y": {
      "field": "score", "type": "quantitative", "axis": null,
      "scale": {"zero": false, "padding": 16}
    }
  },
  "layer": [
    {
      "mark": {"type": "line", "point": true},
      "encoding": {"detail": {"field": "team", "type": "nominal"}}
    },
    {
      "transform": [
        {"filter": "datum.survey === '2025'"},
        {"calculate": "datum.team + '  ' + datum.score", "as": "label"}
      ],
      "mark": {"type": "text", "align": "right", "dx": -8},
      "encoding": {"text": {"field": "label", "type": "nominal"}}
    },
    {
      "transform": [{"filter": "datum.survey === '2026'"}],
      "mark": {"type": "text", "align": "left", "dx": 8},
      "encoding": {"text": {"field": "score", "type": "quantitative"}}
    }
  ]
}
```

## Dumbbell

Before and after (or two groups) for many items, sorted, where a slope chart's labels would collide.

Traps avoided:
- The data is wide (one row per item, a column per period), so a `rule` joins the two columns, and `fold` turns them
  into rows for the coloured points.
- The items are sorted by the later value with `"sort": {"field": "after", "order": "descending"}`. Sorting by an
  encoding such as `-x` would not work, since the folded layer has different rows.

```vega-lite
{
  "data": {
    "values": [
      {"country": "Germany", "before": 31, "after": 44},
      {"country": "France", "before": 28, "after": 35},
      {"country": "Spain", "before": 22, "after": 39},
      {"country": "Italy", "before": 25, "after": 27},
      {"country": "Poland", "before": 12, "after": 30},
      {"country": "Netherlands", "before": 40, "after": 46}
    ]
  },
  "encoding": {
    "y": {
      "field": "country", "type": "nominal", "title": null,
      "sort": {"field": "after", "order": "descending"}
    }
  },
  "layer": [
    {
      "mark": {"type": "rule", "strokeWidth": 2},
      "encoding": {
        "x": {"field": "before", "type": "quantitative", "title": "Adoption (%)"},
        "x2": {"field": "after"}
      }
    },
    {
      "transform": [{"fold": ["before", "after"], "as": ["period", "value"]}],
      "mark": {"type": "circle", "size": 120, "opacity": 1},
      "encoding": {
        "x": {"field": "value", "type": "quantitative"},
        "color": {
          "field": "period", "type": "nominal", "title": null, "sort": ["before", "after"],
          "scale": {"domain": ["before", "after"], "range": ["#a9afb8", "#2a78d6"]}
        }
      }
    }
  ]
}
```

## Pareto chart

Which few causes account for most of the problems: bars sorted largest first, with the cumulative share as a line.

Traps avoided:
- The cumulative share needs the rows in bar order: `window` sorted by count descending, then divided by a
  `joinaggregate` total.
- The bars and the line share an x axis sorted the same way (`"sort": "-y"` is wrong for the line layer, so both sort by
  the count field explicitly), and the line has its own y axis in `%` on the right. Each layer's `color` is a
  `datum`, so the legend names them.

```vega-lite
{
  "data": {
    "values": [
      {"cause": "Config error", "n": 46}, {"cause": "Dependency outage", "n": 31},
      {"cause": "Bad deploy", "n": 18}, {"cause": "Capacity", "n": 9},
      {"cause": "Hardware", "n": 4}, {"cause": "Other", "n": 2}
    ]
  },
  "transform": [
    {
      "window": [{"op": "sum", "field": "n", "as": "running"}],
      "sort": [{"field": "n", "order": "descending"}]
    },
    {"joinaggregate": [{"op": "sum", "field": "n", "as": "total"}]},
    {"calculate": "datum.running / datum.total", "as": "cumulative"}
  ],
  "encoding": {
    "x": {
      "field": "cause", "type": "nominal", "title": null,
      "sort": {"field": "n", "order": "descending"},
      "axis": {"labelExpr": "split(datum.label, ' ')"}
    },
    "color": {"title": null}
  },
  "layer": [
    {
      "mark": "bar",
      "encoding": {
        "y": {"field": "n", "type": "quantitative", "title": "Incidents"},
        "color": {"datum": "Incidents"}
      }
    },
    {
      "mark": {"type": "line", "point": true},
      "encoding": {
        "y": {
          "field": "cumulative", "type": "quantitative", "title": "Cumulative share",
          "scale": {"domain": [0, 1]},
          "axis": {"format": "%", "orient": "right"}
        },
        "color": {"datum": "Cumulative share"}
      }
    }
  ],
  "resolve": {"scale": {"y": "independent"}}
}
```

## Running total against a target (burn-up)

Progress accumulating toward a goal, such as story points done in a release or spend against an annual budget.

Traps avoided:
- `window` with `sum`, sorted by date, turns weekly amounts into a running total.
- The goal is a rule with its label at the right end, on its own one-row `data`, as in the reference line example.

```vega-lite
{
  "data": {
    "values": [
      {"week": "2026-03-02", "points": 18}, {"week": "2026-03-09", "points": 22},
      {"week": "2026-03-16", "points": 15}, {"week": "2026-03-23", "points": 27},
      {"week": "2026-03-30", "points": 20}, {"week": "2026-04-06", "points": 24}
    ]
  },
  "transform": [
    {"window": [{"op": "sum", "field": "points", "as": "done"}], "sort": [{"field": "week"}]}
  ],
  "layer": [
    {
      "mark": {"type": "line", "point": true},
      "encoding": {
        "x": {
          "field": "week", "type": "temporal", "title": "Week starting", "scale": {"type": "utc"},
          "axis": {"format": "%d %b", "tickCount": "week"}
        },
        "y": {
          "field": "done", "type": "quantitative", "title": "Story points done",
          "scale": {"domain": [0, 160]}
        }
      }
    },
    {
      "data": {"values": [{"scope": 150, "label": "Release scope 150"}]},
      "encoding": {"y": {"field": "scope", "type": "quantitative"}},
      "layer": [
        {"mark": {"type": "rule", "strokeDash": [6, 4]}},
        {
          "mark": {"type": "text", "align": "left", "dx": 6},
          "encoding": {
            "x": {"value": {"expr": "width"}},
            "text": {"field": "label", "type": "nominal"}
          }
        }
      ]
    }
  ]
}
```

## Labels at the ends of lines

Several series labelled where they end instead of in a legend, so the reader needn't match colours.

Traps avoided:
- The label layer picks each series' last point with `"aggregate": "max"` on the date and `{"argmax": "month"}` on
  the value, which works whatever order the rows are in.
- The legend is turned off (`"legend": null`) on the line layer, since the labels replace it.
- The text uses the same `color` field so each name matches its line. Here that is what is wanted.

```vega-lite
{
  "data": {
    "values": [
      {"month": "2026-01-01", "browser": "Chrome", "share": 0.61},
      {"month": "2026-01-01", "browser": "Safari", "share": 0.24},
      {"month": "2026-01-01", "browser": "Firefox", "share": 0.08},
      {"month": "2026-02-01", "browser": "Chrome", "share": 0.6},
      {"month": "2026-02-01", "browser": "Safari", "share": 0.26},
      {"month": "2026-02-01", "browser": "Firefox", "share": 0.07},
      {"month": "2026-03-01", "browser": "Chrome", "share": 0.58},
      {"month": "2026-03-01", "browser": "Safari", "share": 0.28},
      {"month": "2026-03-01", "browser": "Firefox", "share": 0.07}
    ]
  },
  "encoding": {"color": {"field": "browser", "type": "nominal", "legend": null}},
  "layer": [
    {
      "mark": {"type": "line", "point": true},
      "encoding": {
        "x": {
          "field": "month", "type": "temporal", "title": null, "scale": {"type": "utc"},
          "axis": {"format": "%b %Y", "tickCount": "month"}
        },
        "y": {
          "field": "share", "type": "quantitative", "title": "Share of visits",
          "axis": {"format": "%"}
        }
      }
    },
    {
      "mark": {"type": "text", "align": "left", "dx": 6},
      "encoding": {
        "x": {"field": "month", "type": "temporal", "aggregate": "max"},
        "y": {"field": "share", "type": "quantitative", "aggregate": {"argmax": "month"}},
        "text": {"field": "browser", "type": "nominal"}
      }
    }
  ]
}
```

## One series highlighted

Many series for context, with one (yours) picked out, so the chart has a point.

Traps avoided:
- Every line is drawn in the theme's colour, and a `condition` on `opacity` and `strokeWidth` fades all but the one
  named. No colours are set, so the chart follows the theme.
- `detail` splits the lines without a colour legend.
- The highlighted line is labelled at its end, the others aren't.

```vega-lite
{
  "data": {
    "values": [
      {"q": 1, "co": "Us", "v": 10}, {"q": 2, "co": "Us", "v": 14}, {"q": 3, "co": "Us", "v": 21},
      {"q": 4, "co": "Us", "v": 29}, {"q": 1, "co": "Acme", "v": 18},
      {"q": 2, "co": "Acme", "v": 19}, {"q": 3, "co": "Acme", "v": 21},
      {"q": 4, "co": "Acme", "v": 20}, {"q": 1, "co": "Globex", "v": 12},
      {"q": 2, "co": "Globex", "v": 11}, {"q": 3, "co": "Globex", "v": 15},
      {"q": 4, "co": "Globex", "v": 16}, {"q": 1, "co": "Initech", "v": 8},
      {"q": 2, "co": "Initech", "v": 10}, {"q": 3, "co": "Initech", "v": 9},
      {"q": 4, "co": "Initech", "v": 12}
    ]
  },
  "encoding": {
    "x": {"field": "q", "type": "ordinal", "title": "Quarter"},
    "y": {"field": "v", "type": "quantitative", "title": "Market share (%)"}
  },
  "layer": [
    {
      "mark": "line",
      "encoding": {
        "detail": {"field": "co", "type": "nominal"},
        "opacity": {"condition": {"test": "datum.co === 'Us'", "value": 1}, "value": 0.25},
        "strokeWidth": {"condition": {"test": "datum.co === 'Us'", "value": 3}, "value": 1.5}
      }
    },
    {
      "transform": [{"filter": "datum.co === 'Us' && datum.q === 4"}],
      "mark": {"type": "text", "align": "left", "dx": 6, "fontWeight": "bold"},
      "encoding": {"text": {"value": "Us"}}
    }
  ]
}
```

## Annotated point and event line

Pointing out what happened: a labelled peak, and a vertical line at a launch or incident date.

Traps avoided:
- The note is a field in the data, and a text layer draws only the rows that have one (`"filter": "datum.note"`),
  so the label sits exactly on its point.
- The event line is a rule at a fixed date `datum`, written as a date object with `"utc": true` to match the scale.
- `"baseline": "bottom"` and `dy` lift the label clear of the point and line.

```vega-lite
{
  "data": {
    "values": [
      {"week": "2026-02-02", "tickets": 120}, {"week": "2026-02-09", "tickets": 131},
      {"week": "2026-02-16", "tickets": 128},
      {"week": "2026-02-23", "tickets": 214, "note": "Outage"},
      {"week": "2026-03-02", "tickets": 150}, {"week": "2026-03-09", "tickets": 118},
      {"week": "2026-03-16", "tickets": 96}, {"week": "2026-03-23", "tickets": 90}
    ]
  },
  "encoding": {
    "x": {
      "field": "week", "type": "temporal", "title": null, "scale": {"type": "utc"},
      "axis": {"format": "%d %b", "tickCount": "week"}
    },
    "y": {
      "field": "tickets", "type": "quantitative", "title": "Support tickets",
      "scale": {"domain": [0, 250]}
    }
  },
  "layer": [
    {"mark": {"type": "line", "point": true}},
    {
      "transform": [{"filter": "datum.note"}],
      "mark": {"type": "text", "baseline": "bottom", "dy": -8, "fontWeight": "bold"},
      "encoding": {"text": {"field": "note", "type": "nominal"}}
    },
    {
      "data": {"values": [{"date": "2026-03-09", "label": "Self-service help launched"}]},
      "layer": [
        {
          "mark": {"type": "rule", "strokeDash": [4, 3]},
          "encoding": {"x": {"field": "date", "type": "temporal"}, "y": null}
        },
        {
          "mark": {"type": "text", "align": "left", "baseline": "top", "dx": 4},
          "encoding": {
            "x": {"field": "date", "type": "temporal"},
            "y": {"value": 4},
            "text": {"field": "label", "type": "nominal"}
          }
        }
      ]
    }
  ]
}
```

## Number formats: currency, thousands and percentages

Formatting numbers so a reader takes them in at a glance: `$1.2M` on an axis, `$1,234` and `12.5%` on labels.

Traps avoided:
- Formats are d3 format strings: `",.0f"` adds thousand separators, `"$,.0f"` a dollar sign, `".1%"` multiplies a
  fraction by 100 and adds `%`. Store shares as fractions (0.125), not percents (12.5), or `%` shows 1250%.
- `"$.2s"` gives `$1.2M`, but SI prefixes say `G` for billions, not `B`. For billions divide in `labelExpr`, as on
  the y axis here.
- The axis and the labels format separately: `axis.format` for the axis, `format` on the text encoding for labels.

```vega-lite
{
  "data": {
    "values": [
      {"segment": "Enterprise", "arr": 2350000000, "margin": 0.412},
      {"segment": "Mid-market", "arr": 1180000000, "margin": 0.338},
      {"segment": "SMB", "arr": 640000000, "margin": 0.215}
    ]
  },
  "transform": [
    {
      "calculate": "format(datum.arr / 1e9, '$.2f') + 'B · ' + format(datum.margin, '.1%')",
      "as": "label"
    }
  ],
  "encoding": {"x": {"field": "segment", "type": "nominal", "title": null}},
  "layer": [
    {
      "mark": "bar",
      "encoding": {
        "y": {
          "field": "arr", "type": "quantitative", "title": "Annual recurring revenue",
          "axis": {"labelExpr": "format(datum.value / 1e9, '$.1f') + 'B'"}
        }
      }
    },
    {
      "mark": {"type": "text", "baseline": "bottom", "dy": -3},
      "encoding": {
        "y": {"field": "arr", "type": "quantitative"},
        "text": {"field": "label", "type": "nominal"}
      }
    }
  ]
}
```

## Number formats: another currency or locale

Euros, pounds or a European number style (`1.234,5 €`).

Traps avoided:
- `"$"` in a format is the locale's currency, US dollars by default. `config.locale.number` sets the currency symbol,
  the decimal mark and the thousands separator for the whole chart, so `"$,.0f"` then reads `1.234 €`.
- Setting `locale` in the chart's `config` changes only number formatting; the colours and fonts still come from the
  theme.

```vega-lite
{
  "config": {
    "locale": {
      "number": {"decimal": ",", "thousands": ".", "grouping": [3], "currency": ["", " €"]}
    }
  },
  "data": {
    "values": [
      {"country": "Germany", "sales": 182400}, {"country": "France", "sales": 131250},
      {"country": "Italy", "sales": 76900}, {"country": "Spain", "sales": 64300}
    ]
  },
  "encoding": {
    "y": {"field": "country", "type": "nominal", "sort": "-x", "title": null},
    "x": {"field": "sales", "type": "quantitative", "title": "Sales", "axis": {"format": "$,.0f"}}
  },
  "layer": [
    {"mark": "bar"},
    {
      "mark": {"type": "text", "align": "left", "dx": 4},
      "encoding": {"text": {"field": "sales", "type": "quantitative", "format": "$,.0f"}}
    }
  ]
}
```
