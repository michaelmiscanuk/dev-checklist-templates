## Laying out a responsive page

When: Starting a new page or section that has to work from a small phone up to a wide desktop screen.

Before you start: Is anything on this page reused in slots of different widths? (any that apply) Yes, blocks are reused / None of these. Checks marked "only if" apply only to that answer.

### 1. Before you write layout CSS

You have the design or the content and are about to choose the page's structure.

Must:
- [ ] Add the viewport meta tag
- [ ] Write markup in reading order

Stop if:
- If the design shows only one desktop width: ask the designer for phone and tablet layouts first

Move on when every block has a place at narrow, medium and wide sizes; base styles are the phone layout.

### 2. While building each section (every time)

You are writing the grid, flex and sizing rules for one section of the page.

Must:
- [ ] Set min-width 0 on shrinking items
- [ ] Use max-width, not fixed widths

Stop if:
- If text only fits with a fixed height: stop, remove the height and let the box grow

Move on when the section resizes smoothly with the window and never causes a horizontal scrollbar.

### 3. When you add a breakpoint (every time)

The layout stops working at some width and you are about to add a media or container query.

Must:
- [ ] Set breakpoints where content breaks

Stop if:
- If a block needs different breakpoints in each place it sits: switch it to container queries

Move on when the new breakpoint fixes its width, and no width just above or below it breaks.

### 4. Before you call it done

The page works in your own window; now test it the way real readers will see it.

Must:
- [ ] Drag through every width
- [ ] Test reflow at high browser zoom
- [ ] Enlarge the default text size

Stop if:
- If content scrolls sideways at the narrowest width: go to the Layout breaking at some screen sizes checklist
- If fixing large-text failures changes the design: take it to the designer before shipping

Move on when the page passes the width drag, high zoom, large text and real phone tests.

---
Must checks only. Why each check matters, how to verify it and its source: [full checklist](https://github.com/michaelmiscanuk/dev-checklist-templates/blob/main/checklists/laying-out-a-responsive-page.md). Interactive version: [Laying out a responsive page on CheatGrid](https://www.cheatgrid.com/checklists/frontend-developer/laying-out-a-responsive-page?utm_source=github&utm_medium=listing&utm_campaign=laying-out-a-responsive-page).
