# Javascript CRUD
1. Vanilla JavaScript

![Roger's Retro Cars: a form for brand, model, year and color above a heading reading "The
collection", followed by a list of four cars — each row showing brand, model, year and color
with Edit and Delete buttons on the right.](../Docs/screenshot.png)

Full CRUD against the Cars API, in a single self-contained `index.html`: no build step, no
dependencies, no framework. Open the file and it runs.

This is the first of three levels. Each one builds the same app again, so you can feel what
each tool actually buys you — see [What comes next](#what-comes-next).

## Running it

The frontend needs the API up first. From the repo root, in two terminals:

```bash
# Terminal 1 — the API on http://localhost:5014
cd api
dotnet run
```

```bash
# Terminal 2 — any static server will do
cd web-vanilla
npx serve
```

Then open the address the server prints. Opening `index.html` straight off disk (`file://`)
works too, since CORS is wide open on the API.

Swagger UI is at <http://localhost:5014/swagger> — useful when you want to know whether a bug
is in your fetch or in your data.

The database (`api/cars.db`) seeds itself with four cars on first run. Delete the file and
restart the API to get them back.

## What it does

| Operation | Request                  | Response         |
| --------- | ------------------------ | ---------------- |
| Read      | `GET /api/cars`          | `200` + array    |
| Create    | `POST /api/cars`         | `201` + the car  |
| Update    | `PUT /api/cars/{id}`     | `204`, no body   |
| Delete    | `DELETE /api/cars/{id}`  | `200` + message  |

`PUT` returns `204 No Content`. There is nothing to parse — calling `.json()` on that response
throws.

## How it's put together

Everything lives in `index.html`: markup, styling hook, and one `<script>` at the bottom.

- **`fetchCars()`** is the single source of truth for what's on screen. Every mutation ends by
  calling it again and re-rendering from the server, rather than patching the DOM by hand.
- **`renderCars()`** rebuilds the whole list from the array it's handed.
- **`currentEditingId`** is a module-scope variable holding the id of the car being edited, or
  `null` when adding. It's what the submit handler branches on to choose `POST` or `PUT`, and
  it's the reason one form serves both jobs.

That last one is worth sitting with. `currentEditingId` lives outside every function because
two separate user actions — clicking *Edit*, then clicking *Update* — have to agree about
which car is in play, and nothing else connects them. Forget to reset it after a successful
save and the next car you add silently overwrites the one you just edited. This is the exact
problem React was built to remove, and Level 2 removes it.

Styling is [Pico CSS](https://picocss.com) from a CDN — classless, so the HTML stays semantic
and there's nothing to configure. It's a placeholder until Level 3.

## Known rough edges

Left in deliberately — each one is something a later level removes properly:

- **`onclick` with a serialized object.** `prepareEdit(...)` pushes an entire object through
  an HTML attribute. JSON is full of double quotes, which would close the attribute early and
  leave the browser compiling a fragment like `prepareEdit({` — hence the
  `.replace(/"/g, '&quot;')`. It works, and it's still a data structure squeezed through a
  string. Level 2 hands the object to the handler directly, and the problem stops existing.
- **`innerHTML` with server data.** Fine for trusted seed data, not fine as a habit.
- **`alert()` for errors.** Blocking, unstyled, unhelpful.
- **No loading state.** The list is empty while the first fetch is in flight.

## What comes next

**Level 2 — React.** The same app, same API, same `fetch` calls. `useState` replaces the
module-scope globals and `useEffect` replaces the manual `fetchCars()` call, so the DOM stops
being something you update and becomes something you describe. `currentEditingId` becomes
lifted state, and the "forgot to reset it" bug becomes structurally hard to write.

**Level 3 — Tailwind, then TypeScript.** Pico gives way to real design control, and the `Car`
shape stops being something you have to remember — `car.Brand` vs `car.brand` becomes a red
squiggle instead of `undefined undefined` on screen. Optional, and worth doing.
