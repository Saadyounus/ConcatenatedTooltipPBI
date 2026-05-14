# Concatenated Tooltips Demo — Power BI

A Power BI report demonstrating how to surface multiple row-level values from a single map location using a `CONCATENATEX` DAX measure. Hovering over any mall on the Azure Maps visual shows all tenant shops in a single, readable tooltip field.

![Demo](tooltip_demo_short.mp4)

---

## The Problem

Power BI map visuals (and most other visuals) can only display one row per location in a tooltip. If your table has multiple rows sharing the same category or coordinates — for example, one row per shop per mall — the tooltip only ever shows the first row. There is no built-in way to aggregate text values across rows in a tooltip.

---

## The Solution

A DAX measure that uses `CONCATENATEX` to collapse all related rows into a single comma-separated string. The measure is then added to the **Tooltips** field well of the Azure Maps visual, making all values visible on hover.

---

## Data Model

A single flat table: `gulf_mall_demo`

| Column | Description |
|---|---|
| `Mall` | Mall name (the grouping key) |
| `Shop Name` | Individual tenant shop name |
| `Latitude` | Mall latitude coordinate |
| `Longitude` | Mall longitude coordinate |

Each mall has multiple rows, one per tenant shop. All rows for the same mall share the same `Mall`, `Latitude`, and `Longitude` values.

---

## DAX Measure

```dax
Shops = 
CONCATENATEX(
    DISTINCT(
        SELECTCOLUMNS(
            FILTER(
                gulf_mall_demo,
                gulf_mall_demo[Mall] = SELECTEDVALUE(gulf_mall_demo[Mall])
            ),
            "Shops", gulf_mall_demo[Shop Name]
        )
    ),
    [Shops],
    ", "
)
```

### How it works

| Step | Function | Purpose |
|---|---|---|
| 1 | `SELECTEDVALUE(gulf_mall_demo[Mall])` | Gets the mall currently in context (the hovered map point) |
| 2 | `FILTER(...)` | Narrows the table to only rows matching that mall |
| 3 | `SELECTCOLUMNS(...)` | Projects only the `Shop Name` column into a virtual table, renaming it `Shops` |
| 4 | `DISTINCT(...)` | Removes duplicate shop names |
| 5 | `CONCATENATEX(..., ", ")` | Joins all shop names into a single comma-separated string |

---

## Visual Configuration

1. Add an **Azure Maps** visual to your report page
2. Set the following field wells:

| Field Well | Column / Measure |
|---|---|
| Location | `Latitude`, `Longitude` |
| Legend | `Mall` |
| Tooltips | `Shops` (the measure above) |

3. Enable **Tooltips** in the visual's Format pane if not already on

---

## Adapting This Pattern

This pattern works for any scenario where you have multiple rows per category and want to surface all values in a tooltip, not just the first. Replace `gulf_mall_demo`, `Mall`, and `Shop Name` with your own table and column names.

**Other use cases:**

- Products per supplier location on a map
- Tags or labels per project in a matrix
- Incidents per site over a time period
- Staff names per department in a table visual

The delimiter `", "` can be changed to any separator, including `UNICHAR(10)` for a line break (works in some visuals).

---

## Requirements

- Power BI Desktop (any recent version)
- Azure Maps visual (built-in, no marketplace install needed)
- No custom visuals or external dependencies required

---

## Files

| File | Description |
|---|---|
| `Concatenated Tooltips Demo.pbix` | Main Power BI report file |
| `tooltip_demo_short.mp4` | Demo video for LinkedIn / social sharing |
| `README.md` | This file |
