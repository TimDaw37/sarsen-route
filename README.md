# sarsen-route

**Sarsen transport modeller — the dials.** An interactive map of 360 pre-solved least-cost routes for hauling sarsens from West Woods, near Marlborough, to Stonehenge. Move a dial to see which landscape factors have to matter for a given corridor to emerge.

Author: **Tim Daw** ([sarsen.org](https://www.sarsen.org)) · Contact: tim.daw@gmail.com

This is a sensitivity tool, not a claim about the route the builders took. The opening setting is an analyst judgement, not a model output.

## Live

https://timdaw37.github.io/sarsen-route/ (formerly https://sarsenroute.netlify.app/)

## Pages

| File | Role |
|------|------|
| [index.html](index.html) | The whole app: one self-contained page, all route data embedded. Opens from `file://` too |

## The model

- **Start and end:** a 1 km circle round West Woods to Stonehenge.
- **Terrain:** OS Terrain 50 resampled to a 20 m grid, about 1.77 million cells and 14.1 million connections.
- **Ground:** BGS geology at 1:50,000. Alluvium and peat are measured but not charged; they are largely later than the question.
- **Checks:** 1 m LiDAR, and the Andrews & Dury map of Wiltshire (1773).
- **Cross-slope** was tested and changes almost nothing, so it is held fixed.

## The dials

| Dial | Default | What it does |
|------|---------|--------------|
| Climb penalty | 14 | Cost of gaining and losing height. Turning it up drives the route into the valleys, trading wet ground for gentler grades |
| Soft ground penalty | 20 | Charges ground that is level and low, or on Gault clay or greensand, taking the greater of the two, never both |
| Watercourse crossing cost | 2000 | Charged per stream or river crossed. The strongest lever: from 13 crossings down to 1, and interfluve (dry ridge) use from 17% to 46% |
| Descent prior, Walkers Hill over Huish | moderate | New Town / Workway (N2) has a road down the scarp on Andrews & Dury 1773, a holloway with a 73 m drop in the LiDAR, and usable tracks today; Huish (D2) has only a narrow farm track. Discounts N2 and surcharges N1 across a 600 m catchment on the descent, nowhere else |

Each setting reports length, total ascent, steepest gradient, watercourses crossed, distance within 200 m of water, and share on interfluve.

## Write-ups

On [sarsen.org](https://www.sarsen.org).

## History

Moved from Netlify to GitHub Pages on 29 Sep 2026. The page is the live Netlify version with only Netlify's own injected toolbar removed.
