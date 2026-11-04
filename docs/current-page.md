# Current page: voxility.com/network/latency-check

Notes taken from the live page on 2026-11-02, as the baseline for the 3D globe redesign.

## How it works today

- The visitor types an **IP or hostname** (`#ip_to_check`) and presses **Ping** (Enter also works).
- **Every PoP pings that target from Voxility's side.** The test runs on Voxility's servers, not in the visitor's browser.
  - `GET https://api.voxility.com/latency-check/ping.wjson?ip_to_check=<target>&network_monitor=<pop-id>`
  - One request per PoP, all sent at the same time. Each result uses `data.data[0].min` / `min_formatted` (ms).
- When a ping finishes, that PoP's box is coloured by its latency, and the page title and H1 become
  `Latency Check: <ms> ms for <target> from <fastest PoP>`.
- The target goes into the URL as `?ip=<target>` (`?mode=ip` is also read), so a test can be shared or bookmarked.
- After the pings finish, an **MTR** button appears.
  - `GET https://api.voxility.com/latency-check/mtr.wjson` (same parameters) runs for every PoP.
  - Once a PoP has MTR data, clicking it shows a table with the columns Host, Avg, **Best**, Last, Loss% *, Snt, Stdev, Wrst, plus a Refresh button for that PoP.
  - Hint: "Select a location on the map to view MTR results."
  - Busy state: "At least one location is busy processing another request. Please try again in a few seconds."
- Errors 412/422 show as a validation message on the input field.
- The current map is a flat PNG (`cdn.voxility.com/.../world-map.png`) with absolutely positioned label boxes connected to dots by lines.

## PoPs (13)

| id               | Label           | Full name              | location_id  |
|------------------|-----------------|------------------------|--------------|
| los-angeles      | Los Angeles     | Los Angeles, USA       | 66110619649  |
| dallas           | Dallas          | Dallas, USA            | 68559044609  |
| miami            | Miami           | Miami, USA             | 54049898497  |
| ashburn-reston   | Ashburn/Reston  | Washington, USA        | 37874565121  |
| london           | London          | London, England        | 6134169601   |
| amsterdam        | Amsterdam       | Amsterdam, Netherlands | 36271292417  |
| frankfurt        | Frankfurt       | Frankfurt, Germany     | 81585504257  |
| vienna           | Vienna          | Vienna, Austria        | 69031952385  |
| bucharest        | Bucharest       | Bucharest, Romania     | 93314875393  |
| prague           | Prague          | Prague, Czech Republic | 76377227265  |
| madrid           | Madrid          | Madrid, Spain          | 96022298625  |
| singapore        | Singapore       | Singapore              | 44717572097  |
| hong-kong        | Hong Kong       | Hong Kong              | 33455865857  |

## Latency tiers (from the current code)

| Tier        | Rule          | Light bg              | Selected bg           |
|-------------|---------------|-----------------------|-----------------------|
| top-speed   | ≤ 5 ms        | hsl(115 130% 90%), flashing animation, border `#41c200` | hsl(142 94% 37%) |
| good-speed  | 5–15 ms       | hsl(85 130% 90%)      | hsl(85 70% 45%)       |
| medium-speed| 15–60 ms      | hsl(60 130% 90%)      | hsl(55 95% 40%)       |
| other-speed | > 60 ms       | hsl(30 130% 90%)      | hsl(35 85% 50%)       |

## Brand palette (from the site CSS)

| Token        | Value                          | Where it's used                  |
|--------------|--------------------------------|----------------------------------|
| yellow       | `#ffff00`                      | main brand accent (nav, highlights) |
| olive/gold   | `#8c7c00` / rgba(140,124,0,…)  | links and accents in the light theme |
| dark         | `#1a1a1a`, `#262626`           | header and dark surfaces          |
| grey         | `#6c6c6c`, `#888`, `#dddddd`   | secondary text, borders           |
| light        | `#f5f5f5`, `#f7f7f7`, `#ffffff`| backgrounds                       |
| orange       | `#fd8e01` (+ `#fff8f0` tint)   | latency-check page accent         |
| green        | `#41c200`                      | best-latency border               |

Font: `"Segoe UI", "Apple SD Gothic Neo", Arial, Helvetica`.

## What this means for the globe

- The arcs should run **from each PoP to the target** (or to the visitor, before anything has been tested). The target's position could come from a GeoIP lookup on the IP being tested.
- Keep the 4 latency tiers and the ms thresholds so the results read the same way as today.
- Keep the "fastest PoP" headline, `?ip=` URL sharing, MTR per PoP, and the busy and validation states.
- A preview hosted anywhere other than voxility.com probably can't call `api.voxility.com` directly, because the browser will block the cross-origin request unless the API allows it. That isn't verified yet. Development should use a mock API that returns the same response shapes.
