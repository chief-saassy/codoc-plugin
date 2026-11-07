---
name: charts
description: Use when adding, changing, checking or showing a chart in a Codoc document (a vega-lite block), including its data, its theme, copying it to another document, or picturing it in chat.
---

# Charts in Codoc

A ```` ```vega-lite ```` fence is a chart. Codoc's server draws it into a picture, so readers see the picture and can open the source. There is no hover or other interaction. Codoc checks every chart that an edit, a suggestion or `create_document` adds or changes, and refuses one it can't draw (`invalid_diagram`, with the reason).

**Start from an example.** [examples.md](examples.md) has a tested chart for each common case: bars, lines, trends, parts of a whole, distributions, schedules, annotations. Copy the closest one and change its data and fields. Writing Vega-Lite from scratch is where the traps below catch you.

**Then look at it.** After adding or changing a chart, call `draw_chart(doc, block)` and look at the picture before telling the user it's done. A chart can draw without error and still be wrong: labels on the wrong slices, text the colour of what it sits on, an axis that starts in the wrong place.

## Writing one

- **The source is one JSON object,** a Vega-Lite spec.
- **Put the title in the text, not the spec.** Write a short line just above or below the fence, as for a diagram.
- **Write the data in it:** `"data": {"values": [...]}`, or `"data": {"name": "sales"}` with `"datasets": {"sales": [...]}`. Include only what the chart draws, since readers can see it all in the source. For larger data, see *A chart's data* below.
- **Add it up first.** A chart can have up to 2,000 points: every row in the source and every mark drawn, since a transform can make more. Send 12 monthly totals, not the 50,000 rows behind them.
- **Nothing can be fetched:** a `url` in any data or lookup, an `image` mark, `href`, and `url(...)` in a colour or font are all refused.
- **Expressions** can't use `sequence`, `pad`, `test`, `regexp` or `replace`, and data is JSON, not CSV.
- **Leave colours and fonts out** unless they carry meaning. The theme is applied in light and dark, and exports always use light.
- **Limits:** a `sequence` can make up to 2,000 values, the source can be up to 200,000 characters, width and height up to 5,000, `tickCount` and `maxbins` up to 1,000, density `steps` up to 10,000, and the picture up to 5 MiB.
- **If a chart is refused,** the error says why. Reduce the data rather than trying to get round a limit.
- Keep every line of the source to 100 characters or fewer.

## Traps

- **Layers share encodings.** A text layer over arcs or bars inherits the `color` field. That colour overrides the mark's own `color`, so the labels come out the same colour as what they sit on. Set `"color": {"value": "white"}` in the text layer's `encoding`.
- **Stacked layers need the same order.** A text layer without the `color` field stacks in a different order from the slices, so each label lands on the wrong one. Give the whole chart one `order` encoding, such as by value, descending.
- **Dates are `temporal`,** written as ISO dates (`"2026-07-01"`), with `"scale": {"type": "utc"}` and a time format on the axis (`"%b %y"`). Without `utc` a month's tick can go missing, and as `ordinal` dates lose their spacing and can't carry a trend line.
- **A label or rule drawn once needs its own data.** A text or rule layer with a fixed value that shares the chart's data is drawn once per row, piled on itself. Give it one row of its own, such as `"data": {"values": [{"zero": 0}]}`.
- **Sort by a named field.** A colour field on one layer can drop a shared axis `sort`, and `"sort": "-x"` fails when a layer has nothing on x. On a stacked chart, only `order` sets the stack; a colour `sort` orders the legend.
- **A bar's scale starts at zero.** A bar chart whose axis starts higher draws its bars off the plot.
- **Formats:** `~s` writes billions as `G`, so write `B` with a `labelExpr` dividing by `1e9`. `"extent": "ci"` on an error band resamples at random, so the picture changes every time it's drawn: use another extent.
- **Narrow marks hide their labels.** Put a label past the end of a short bar, not inside it.
- **Say what zero means.** Bars must start at zero. A line can leave it out with `"scale": {"zero": false}` when the change is the point, and the text should say so.

## A chart's data

Small data goes in the chart's source. For anything larger, send a JSON array of up to 2,000 row objects to a link with `request_upload` and `{for: "data", doc}`, or in the tool call with `upload_data(doc, data)`. Either answers a `reference`. Put it on the fence line, and name the data in the source:

````
```vega-lite data.sales="codoc-blob:sha256-…"
{"data": {"name": "sales"}, "mark": "bar", "encoding": {…}}
```
````

- **This isn't a data store.** Send only the rows the chart needs, added up first. Readers never get the file, but anyone who can read the document can copy the chart and draw its other columns.
- **A reference works only in the document it was issued for,** and every save checks it (`invalid_reference`). Copy it exactly, and leave it alone when editing the source.
- **To use new data,** send it and replace the reference in one edit, along with any source changes it needs. This can't be a suggestion yet, so in suggest mode, say so in a comment.
- **To copy a chart to another document,** use `copy_chart(from_doc, block, to_doc)` and put its `markdown` in the other document with an edit. A new document can't start with a reference: create it first.

## Themes

Every chart is drawn in the organisation's default theme. Chart themes are assets (`list_assets(type: "chart_theme")`), but a chart can't be given one yet, so don't write a theme's colours into a chart's source to imitate it.

## Showing a chart in chat

`draw_chart(doc, block, scheme?)` gets a picture of a chart. It comes as a PNG image beside the answer, so show the user that image. It's in the light scheme unless they ask for dark, and viewer access is enough. If the answer has `format: "svg"`, a PNG can't be made here: tell the user what `message` says.
