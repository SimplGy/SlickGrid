# SlickGrid (Coatue)

**Note:** This is not the original SlickGrid.  This is a fork that has been heavily customized for our use.
Go [here](https://github.com/mleibman/SlickGrid) if you want the original.

Find documentation and examples in [the wiki](https://github.com/mleibman/SlickGrid/wiki).

## SlickGrid is an advanced JavaScript grid/spreadsheet component

Some highlights:

* Adaptive virtual scrolling (handle hundreds of thousands of rows with extreme responsiveness)
* Extremely fast rendering speed
* Supports jQuery UI Themes
* Background post-rendering for richer cells
* Configurable & customizable
* Full keyboard navigation
* Column resize/reorder/show/hide
* Column autosizing & force-fit
* Column pinning
* Pluggable cell formatters & editors
* Support for editing and creating new rows.
* Grouping, filtering, custom aggregators, and more!
* Advanced detached & multi-field editors with undo/redo support.
* “GlobalEditorLock” to manage concurrent edits in cases where multiple Views on a page can edit the same data.
* Support for [millions of rows](http://stackoverflow.com/a/2569488/1269037)

## Contributing

We're currently modularizing the codebase with [rollup.js](http://rollupjs.org).  If you would like to contribute a change to `slick.grid.js`, edit the files in `src/` and then execute:

```bash
npm run build
```

which will build `slick.grid.js`.  Please don't work directly on the file itself.

## Column pinning

[Pinned column examples](http://git.simple.gy/SlickGrid/) live here.

## Renaming

To support pinned columns, we slice up the grid regions, and try to be very clear and consistent about the naming.
This is because having a left and right region for every content area makes a flat list of naming conventions multiply
quickly.
I also was uncomfortable with the proliferation of names like `header`, `headerScroller`, and `headerRow`.

Two big changes:
1. UI components are labeled top to bottom by what control they are.
2. `headerRow` renamed to `subHeader`. The old name was confusing. TODO: This means the name of related options has
   changed, too.

Canvases always represent content size, viewports always represent scrollable regions.

Every element has side `[0]` and side `[1]`, for left and right.

```
Visual Grid Components
                        [0]       [1]
                      ....................
topViewport           .     .            .     // The scrolling region
  topCanvas           .     .            .     // The full size of content (both off and on screen)
    header            .     .            .     // The column headers
    subHeader         .     .            .     // Optional row of cells below the column headers
                      ....................
contentViewportWrap   .     .            .
contentViewport       .     .            .     // The scrolling region for the grid rows
  contentCanvas       .     .            .     // Full size of row content, both width and height
                      .     .            .
                      .     .            .
                      .     .            .
                      .     .            .
                      .     .            .
                      ....................
```

## Other Changes:

**Adds some methods** that make it more performant to do auto column resizing and exposes some methods that make it
easier to work with multiple grid instances and pinned columns.

* `grid.updateColumnWidths(columnDefinitions)`
  * Using this method improves the performance of changing the width of one or more grid columns by a lot. The existing
    API only allows for a whole grid redraw, which can be very slow. Pull request with notes
    [here](https://github.com/mleibman/SlickGrid/pull/897). Use cases for fast column size adjustment may be:
    auto-sizing columns to fit content, responsive sizing cells to fill the screen, and similar.
* `grid.updateColumnHeader`
  * when called, also ensures that column.headerCssClass gets applied to the header DOM element
  * motivation: <1ms per column, versus 100-200ms when calling grid.setColumns()
* `grid.updateColumnHeaders`
  * calls `grid.updateColumnHeader` for all columns
* `grid.getId()` lets you get the uid of the grid instance
* `grid.isGroupNode(row, cell)` lets you check if a node is part of a group row
* Triggers existing event `onColumnsResized` when you change the column widths
* Triggers a new event `onColumnsChanged` when you set the columns
* Exposes the existing method `grid.setupColumnResize`, which allows you to re-enable column resizing if you're
  manually screwing around with the headers.
* Some new options on `setColumns` and `resizeCanvas` let you prevent some of the expensive calculations, useful if
  you're doing them yourself externally.

**Adds [antiscroll](https://github.com/learnboost/antiscroll) compatibility** to enable a uniform, OSX-style scrolling
experience across browsers. Enable antiscroll by including the antiscroll library on your page, and passing the
`useAntiscroll: true` option to your SlickGrid instance. By default we don't show scrollbars until the user begins
scrolling (to mimic the way OSX does it); to change that behavior, you can set the `showScrollbarsOnHover` option.

**Adds `skipPaging` option** to prevent slickgrid from paging when user keypress take the user off the current page.
Instead, up & down keypresses reveal one new row at a time.

**Exposes** `grid` as the final parameter of the asyncPostRender callback. Signature is now `asyncPostRender(node, row,
dataRow, column, grid)`

**Adds `appendSubheadersToContainer` option** to allow injecting the subheaders straight into the root of the container,
which is useful for absolutely positioning them relative to the container (eg. for a fixed footer). *Note:* this
feature does not support pinned columns.

**Adds `scrollbarSize` and `maxSupportedCssHeight` options**. Pass these in if you know their values beforehand to
avoid layout thrashing on load.

**Adds the `withTransaction(fn)` method** on Slick.Dataview as a shorthand for `beginUpdate()` and `endUpdate()`.